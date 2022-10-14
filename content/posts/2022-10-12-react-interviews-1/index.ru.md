+++
date = "2022-10-12T14:45:00+03:00"
title = "Въетнамские флэшбэки: Девичья память у React'а (и немножко DotNotation)"
description = "Немного рассуждений насчет особенностей разработки на React'е (шутейки прилагаются)"
tags = ["study", "JavaScript", "react", "async", "web",]
keywords = ["react", "reactjs", "dot-notation", "memo", "async"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = false 
+++

---

## Оффтоп

Крутые-модные-молодежные _Svelte'ы_, _Solid'ы_, _Qwik'и_ и прочее
сегодня не рассматриваем, ибо каждый из них топит за свою идеологию и способ
работы.

Мы тут сегодня про "старпёров"
(_слышит голос вдалеке_: "Ма, неси таблетки, деда снова накрыло")

---

## Про что поговорим?

Про `ЁПТИМИЗАЦИЮ`
(автор хотел, правда хотел написать правильно, но не сдержался)

## Шта? 'ЁПТИМИЗАЦИЯ'???? 0_o

Ага, именно так, и именно через "ё", ибо хоть
`одна из` главных `"фишек" React'а` по сравнению с _Angular_ & _Vue_,
`это возможность` достаточно `тонко управлять процессом рендеринга`
UI в Web-Приложении, но по опыту выходит примерно вот как:

Покуда не "съел на этом собаку", можно не раз и ДАЛЕКО не два выстрелить себе
в колено
{{< del >}}
хотя не, о чём это я.
Тут жеж все "тру-JS-developer'ы", а не "framework'щики" какие-то, как я (плачет)
{{< /del >}}

---

(Подумав, клеит на React ярлычок "Fedora", на Vue - "MacOSX Mountain Lion",
а на Angular - "Windows 10 for Business")

## Спектакль в 2 действиях

Давайте представим, что бизнес врывается с ноги в dev-team чат, и такой:

- (Б - Бизнес) НАМ нужен ДАШБОРД!
- (Р - Какой-то разраб) Ну ок, нужен и нужен. (косит под дурака) Мы-то тут причём??
- (Л - ТехЛид) А накиньте контекста: что хотите показывать, для кого делаем, что по эстимейтам, ...?
- (Б) Большой ДАШБОРД! Чтобы там как только мы в админке что-то поменяли, сразу
  рисовалось тут! И чтобы графики моментально были! Столбчатые! Нет, даже Ленточные!
  Даже оба сразу, и чтобы не лагало! Вообще! Сделаете???
- (Л) Ну, в принципе, понятно.
  Давайте соберемся на созвон, обсудим возможные риски и приоритеты
  в соответствии с уже имеющимися задачами.
- (Б) ОК! Завтра, в течение дня!
- (Л) Ок.

_Бизнес уходит, просыпается "мафиозный" dev-team_

- (Л) Народ, вангую, что нас припашут ваять аналог AgGrid, но чисто свой,
  и чтобы с плюшками.
- (Р) А в чём проблема взять AgGrid?
- (Л) Скорее всего, в лицензии, ибо тут бизнес, возможно, захочет собирать
  стату по какому-то внутреннему проекту, у которого бюджета - что кот наплакал.
  Ну или просто показать CEO с СТО, за что мы свой хлеб едим. Андрюха?
- (А - Андрюха) (отвлекается от просмотра боя в Dear Or Alive: Paradise) А??
- (Л) Короче, скорее всего в тебя прилетит таска: сваять таблицу, пока просто -
  без графиков. Так, чтобы там бодро рисовалось, но чтобы не выжирало батарейку
  у ноута в ноль за полминуты. Прям супер производительность пока не нужна, но лучше на начальном
  этапе потратить чутка времени на оптимизацию. Потянешь?
- (Р) ОптЕмизацию
- (Л) Ага, спс. Кстати, как там тесты? Написал?
- (Р) (исчез из-за "внезапно" появившихся проблем с сетью)
- (Л) Так, Андрюх - ты пока поизучай доку по рендерингу реакта, и сделай прототип.
  Ну, допустим, с 2000 строк. Чисто "Hello, world!" в каждой.
  Если что - пингуй, я подскажу
- (А) (кивает, но вообще ни в зуб ногой, чё от него хотят, ибо всю ночь гонял в "танки")
  Ага, принял. Сделаю
- (Л) Ну и ладушки. Поехали

## Код в студию!

А теперь давайте немного о суровом, кодерском.

Допустим, условный "Андрюха" проспался, и прямо перед дейликом накидал
что-то подобное:

```ts
// types/table.d.ts

declare namespace NDataTable {
  interface IDataTable {
    columng: string;
    rows: string[];
  }
}

export = NDataTable;

export as namespace AppTypes;
```

```ts
// TableComponent/index.d.ts
import { HTMLAttributes } from "react";

export interface IProps extends Partial<HTMLAttributes<HTMLDivElement>> {
  header: string;
  data: AppTypes.IDataTable[];
}
```

```tsx
// TableComponent/index.tsx
import { memo, async } from "react";
import { IProps } from ".";

export function TableComponent({
  header,
  data = [],
  ...divAttrs
}: Readonly<IProps>) {
  const computedTableColumnRows = useCallback(
    rows => {
      return rows.map((row, idx) => (
        <div className="table__body__column__row" key={`${idx}-row`}>
          {row}
        </div>
      ));
    },
    [data]
  );

  const computedTableBody = useMemo(() => {
    if (data.length === 0)
      return <div className="table__body__no-data">No Data</div>;

    return (
      <>
        {data.map(item => (
          <>
            <div className="table__body__column__header">{item.column}</div>
            {computedTableColumnRows(item.rows)}
          </>
        ))}
      </>
    );
  }, [data]);

  return (
    <div {...divAttrs}>
      <div className="table__header">{header || "Default table name"}</div>
      <div className="table__body">{computedTableBody}</div>
    </div>
  );
}

TableComponent.Variants = {
  Memoized: memo(TableComponent),
  Async: async(() => import(".")),
  AsyncMemoized: memo(async(() => import(".")))
};

export default TableComponent;
```

```tsx
// HomePage/index.tsx
import { useState, useEffect, Dispatch, SetStateAction } from "react";

const rows = new Array(500).fill("Hello, Data Table World!");
const _mockDataTable: AppTypes.IDataTable[] = new Array(5).fill({
  column: "Some column header",
  rows
});

export function HomePage(): JSX.Element {
  const [data, setData]: [
    Record<String, unknown>,
    Dispatch<SetStateAction<Record<string, unknown>>>
  ] = useState();
  // TODO Better move to separate hook
  useEffect(() => {
    const { abort, signal } = new AbortContoller();

    fetch(`${process.env.REACT_APP_API_URL}/some-endpoint`, { signal })
      .then(data => {
        setState(data);
      })
      .catch(error => {
        console.error(error);
      });

    return () => {
      abort();
    };
  }, []);

  return (
    <SomeParentComponent data={data}>
      <TableComponent data={_mockDataTable} />
    </SomeParentComponent>
  );
}
```

### Нифига не понятно, но уже чутка интересно ^\_^

Ладно-ладно, сек. Ща всё расскажу и покажу. Начнем с "фишечек", а точнее - с хуков.

`useMemo`\|`useCallback`
внутри компонента "TableComponent.tsx"
`"наблюдают" за необходимыми им значениями, и запускают перерисовку`
только в 2 случаях:

- ЕСЛИ изменились значения у наблюдаемых в каждом конкретном хуке полей
  {{< del >}}надеюсь вы знаете, что объекты сюда класть почти бессмысленно?{{< /del >}}
- ЕСЛИ будет будет перерисован РОДИТЕЛЬСКИЙ (SomeParentComponent) компонент

Собсна, резко и дерзко проникаем в вашу Любимую
{{< del >}}IDE, ребзи, IDE!. Надо работу работать, а не вот это вот всё{{< /del >}},
и видим следующее:

#### Оффтоп

---

Что это?
`Это` просто `один из способов добавления новых свойств компоненту, в простонародье - DotNotation`.
Не каждому зайдет, но я, как только распробовал, влюбился
(ибо в больших проектах, простите, за\*\*\*шься бегать по 20+ файлам,
дабы проверить - это Асинхронный, Мемоизированный, или еще какой-то вариант Компонента.
Тут, на мой вкус, нагляднее: `<TableComponent.Memoized />` \| `<TableComponent.Async />`
или вообще "чистый" `<TableComponent />`. У - Удобно).

---

```tsx
TableComponent.Variants = {
  Memoized: memo(TableComponent),
  Async: async(() => import(".")),
  AsyncMemoized: memo(async(() => import(".")))
};
```

Итак, что тут интересного? `Поля "Memoized"` и, может, `"AsyncMemoized"`.

Давайте чисто "по верхам":
"memo" заставляет React проводить глубокое сравнение свойств компонента,
предотвращая излишние перерисовки. (Закрывает наше 2 ИЛИ)

Пока забудем про
[кучу нюансов](https://attardi.org/why-we-memo-all-the-things/),
ибо на текущем этапе это позволит нам сделать главное:

Вообще "на изи" пролететь код-ревью (А вы о чём подумали? 0_o)

### И всё-таки, про мемоизацию...

Так, как только прошли код-ревью, давайте вернемся "к нашим баранам", а точнее:
к перерисовке при изменении родителя ("\<SomeParentComponent \/\>")

```tsx
// HomePage/index.tsx

return (
  <SomeParentComponent data={data}>
    <TableComponent.Memoized data={_mockDataTable} />
  </SomeParentComponent>
);
```

Таким нехитрым приёмом, вы с какой-то степенью уверенности сможете сказать:
"Я сделяль!", и вам даже не будет стыдно
{{< del >}}
но это не точно
{{< /del >}}

Причина такой самоуверенности в том, что, как бы, мы выполнили задание лида:

- Таблица есть? Есть
- \+\\\- 2000 строк в ней есть? Ну, если считать с колонками, то да, есть!
- Тут есть какая-то базовая оптимизация? Есть
- Батарейку при текущем решении мы поберегли? Ну, не сказать, что прям сильно, но
  да, вполне

## Вместо заключения

Текущий пост не преследовал цели быть "супер-мега-всеобъемлющим" руководством
по оптимизации компонентов.

Скорее, это был около рабочий пример из давнего-давнего проекта, на котором
хотелось показать возможные решения не всегда "подсвечиваемых" проблем,
которые возникают при разработке на React'е
(Да-да, Vue - я на тебя смотрю. Готовься, ты следующий)
