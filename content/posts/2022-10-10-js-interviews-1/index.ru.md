+++
date = "2022-10-10T08:52:39+03:00"
title = "Мозги на прокачку: безопасники, XSS и при чем тут формочки"
description = "Попытка пролить свет на нечасто освещаемую тему безопасности в Web-Приложениях"
tags = ["study", "JavaScript", "security", "web"]
keywords = ["xss", "forms"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = false 
+++

## Что?

> `XSS` - это термин, описывающий вредоносный скриптинг на сайте
> путем `внедрения JavaScript` кода в тех частях пользовательского интерфейса,
> которые предварительно `не проверяются разработчиком`

## Как?

Это `возможно из-за особенностей работы JS движков внутри браузера`.

Если точнее, то `движок`, по сути, `парсит скрипты из обычного текста`, а значит -
ему до фонаря, откуда этот скрипт появился.

## А поточнее?

Идея заключается в том, чтобы обходными путями добавить в приложение код,
который изначально не был предусмотрен разработчиками.

### Допустим, через _ссылки_

```jsx
const unsecuredInputId = "unsecured-input_id";
/** @type {[string, (data: string) => void]} */
const [state, setState] = useState();

const onInputChange = el => {
  setState(el.target.value);
};

return (
  <div className="someClassName">
    <label htmlFor={unsecuredInputId}>
      <input id={unsecuredInputId} />
    </label>

    <a href={state} id="socialMediaLink">
      Social Media Link
    </a>
  </div>
);
```

#### И шо-ж в этом плохого??

Допустим, `кулХацкер` введёт в радостно отрисованный ему input вот такую конструкцию
("Cовершенно случайно")

```javascript
javascript: alert("Hello, Hacker World!");
```

В таком случае, после ввода данных может оказаться неприятная ситуация:

1. `кулХацкер` создал профиль с вот такой ссылкой (например, якобы на аккаунт ВК)
2. `Невинный пользователь`, который хочет в личном письме высказать кулХацкеру, где он не прав,
   ничего не подозревая тыкает, докустим, на иконку, дабы перейти по ссылке
3. Вместо заветной странички обидчика, недоумевающий пользователь `видит alert`
   с сообщением "Hello, Hacker World".

Готово, вы великолепны (и, собсна, уволены)

### Допустим, через _dangerouslySetInnerHTML_

#### Оффтоп

Видел в паре проектов выставление иконок таким способом, особеннно -
если они получались откуда-то с сервера, и потом парсились в текст.

Если `Вы` с таким `не сталкивались` - `круто`, вам повезло.

```jsx
/** @type {[string, (data: string) => void]} */
const [innerHtml, setInnerHtml] = useState("");

const onGetSvgFromApi = () => {
  axios
    .get(process.env.REACT_APP_API_URL + "/some-endpoint")
    .then(({ data }) => {
      setInnerHtml(data);
    })
    .catch(error => {
      console.error(error);
    });
};

return (
  <div className="someClassName">
    <button type="button" onClick={onGetSvgFromApi}>
      Получить SVG-иконку с сервера
    </button>

    <div
      id="some-social-media-container"
      dangerouslySetInnerHTML={{ __html: innerHtml }}
    />
  </div>
);
```

#### И шо-ж в этом плохого??

Допустим, `кулХацкер №2` (поскольку у вас `нет SSL сертификата`) как `man-in-the-middle`
перехватил данные, которые летят с вашего сервака, и теперь `шлёт` в ответе
`не SVG'шку`, а

```html
<script>
  alert("Hello, Hacker World!");
</script>
```

При таком раскладе, поскольку вы `используете dangerouslySetInnerHTML`,
то браузер превратит этот ответ в элемент DOM-дерева, а значит - Пам-Пам.

Да-да, все верно. `JavaScript движок`, увидя не обработанный скрипт,
`со спокойной браузерной душой выполнит все`, что отправил ему `кулХацкер №2`

Итог? Вы очешуенны (и снова уволены)

## Вредные советы

1. Никогда не `ВАЛИДИРУЙТЕ` то, `ЧТО ВВОДИТ ПОЛЬЗОВАТЕЛЬ`. Серьезно,
   нафига тратить время на такую глупую штуку, когда можно по\*\*\*\*ить
   на себя в зеркале?

2. Никогда не `УСТАНАВЛИВАЙТЕ SSL-СЕРТИФИКАТ`, ибо нафига - у вас же User
   никогда ничего в Web-Приложухе не делает. Так зачем заморачиваться??

3. Никогда не `СЛЕДИТЕ ЗА dangerouslySetInnerHTML`, ибо - нафига?
   Все работает-же, значит можно заняться более важными делами (например из `ПУНКТА №1`)
