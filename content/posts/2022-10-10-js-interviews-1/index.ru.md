+++
date = "2022-10-10T08:52:39+03:00"
title = "Мозги на прокачку: безопасники, XSS и при чем тут формочки"
description = "Попытка пролить свет на не часто освещаемую тему безопасности в Web-Приложениях"
tags = ["study", "JavaScript", "security", "web"]
keywords = ["xss", "forms"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = true
+++

## Для собеса

> `XSS` - это термин, описывающий вредоносный скриптинг на сайте
> путем `внедрения JavaScript` кода в тех частях пользовательского интерфейса,
> которые предварительно `не проверяются разработчиком`

### Как?

Это `возможно` из-за особенностей работы JS движков внутри браузеров, если точнее -
`из-за того, что движок ради определения корректного порядка выполнения трансформирует код в вид, понятный ему`

### Например?

#### React

```jsx
const unsecuredInputId = "unsecured-input_id";
/** @type {[string, (data: string) => void]} */
const [state, setState] = useState();
const [isFetching, setIsFetching] = useState(false);

const onSendClick = () => {
  setIsFetching(true);

  axios
    .put(process.env.REACT_APP_API_URL, {
      body: state
    })
    .then({ data } => {
      if (data) console.log(state);

      setIsFetching(false);
    });
};

return (
  <div className="someClassName">
    <label htmlFor={unsecuredInputId}>
      <input id={unsecuredInputId} />
    </label>
    <button type="button" className="someButtonClassName" onClick={onSendClick}>
      Отправить данные
    </button>
  </div>
);
```

В текущем виде, если пользователь введёт сюда `alert('Hello, Hacker World!')`,
то `при нажатии на кнопку` (на моменте с `console.log`)
покажется Alert с текстом "Hello, Hacker World!".

Это сработает, потому что здесь `нет валидации` введенных пользователем данных

Данная уязвимость `распространяется на`:

- `input`
- `textarea`
- `URL`, если приложение `слушает` какие-то `параметры` из адресной строки

### Решение?

`**УНИВЕРСАЛЬНО**` - ВАЛИДИРОВАТЬ

#### А поточнее?

1. `Валидировать` вводимые пользователем данные перед фактическим выполнением логики
   (стандартный пример: проверка на валидность `eMail'а` или `пароля`)
2. `Не запускать код` до тех пор, пока `не прошла валидация`.

## Для общего развития
