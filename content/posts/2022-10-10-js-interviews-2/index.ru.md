+++
date = "2022-10-10T21:12:57+03:00"
title = "Мозги на прокачку: асинхронная синхронность или 'работа в темпе вальса'"
description = "Немного 'брюзжания' на тему не всегда очевидного поведения асинхронности в JS"
tags = ["study", "JavaScript", "async", "web", "event-loop"]
keywords = ["async-await", "race-condition", "engines"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = true
+++

## Что?

> `Асинхронность` в JavaScript - это
> `способность исполнять команды, не блокируя взаимодействие` с приложением

## Как?

Это возможно благодаря тому, что JavaScript, хоть он и "как-бы-однопоточный"
([Web-Worker'ы](https://developer.mozilla.org/ru/docs/Web/API/Web_Workers_API/Using_web_workers) - привет!),
имеет `под капотом не только себя` любимого, `но и`, например, `Web API`.
`Или V8`.

## А поточнее?

Идея в том, что чтобы `улучшить UserExperience, не нужно забирать` у пользователя
`возможность взаимодействия` с Web-Приложением.

Например, давайте представим, что у нас есть Word.

Вы хотите добавить в ваш Word'овский документ таблицу, допустим, с 2000 строк.

Поскольку законы физики никто не отменял, а закон Мура в последнее время
начинает "подбуксовывать", то давайте представим вот такой пример:

```ts
// table.d.ts

export interface IDataTable {
  columng: string;
  rows: string[];
}

export interface IProps {
  header: string;
  data: IDataTable[];
}
```

```jsx
// TableComponent.jsx

/**
 * @returns {JSX.Element} Cool table Component
 *
 * @param {import("table").IProps & import("react").HTMLAttributes<HTMLDivElement>} props
 */
export function TableComponent({ header, data, ...divProps }) {
  const computedTableColumnRows = useCallback(
    rows => {
      return rows.map(row => (
        <div className="table__body__column__row">{row}</div>
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
    <div {...divProps}>
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

```jsx
// root.jsx

/** @type {import("table").IDataTable[]} */
const _mockDataTable = new Array(5).fill({
  column: "Some column header",
  rows: new Array(500).fill("Hello, Data Table World!")
});
/**
 * @returns {JSX.Element} Root element
 */
export function App() {
  return <TableComponent data={_mockDataTable} />;
}
```

### Допустим, через _макротаски_
