+++
date = "2022-10-12T14:45:00+03:00"
title = "Въетнамские флэшбэки: Девичья память у React'а (и немножко DotNotation)"
description = "Немного рассуждений насчет особенностей разработки на React'е (самоирония прилагается)"
tags = ["study", "JavaScript", "react", "async", "web",]
keywords = ["react", "reactjs", "dot-notation", "memo", "async"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = true 
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

Про `оптимизацию` и `профилирование компонентов`
{{< del >}}и немножко о react'овском геморрое{{< /del >}}

### Про оптимизацию

`Одна из` главных `"фишек" React'а` по сравнению с _Angular_ & _Vue_,
`это возможность` достаточно `тонко управлять процессом рендеринга`
UI в Web-Приложении
(клеит на React ярлычок "Fedora", Vue - "MacOSX Mountain Lion",
а на Angular - "Windows 10 for Business")

Если точнее, то я сейчас говорю про то, `как React понимает`,
`что` ему `пора работать и как` мы можем `этим` процессом `управлять`.

{{< del >}}А не балду гонять, ваяя постики в блоге{{< /del >}}

#### И-и-и???

На самом деле, React проповедует достаточно простую, но эффективную идеологию
`однонаправленного потока данных`
{{< del >}} о как звучит! Да-да, совсем накрыло{{< /del >}}

Обычно о ней говорят в контексте работы с реактивностью, но свою "лепту" в
отрисовку компонентов это тоже привносит
(И, забегая вперед, не всегда очевидную, но прям существенную).

#### Спектакль в 2 действиях

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

_Бизнес уходит, просыпается dev-team_

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

#### Код в студию!

А теперь давайте немного о суровом кодерском.

Допустим, условный "Андрюха" проспался, и прямо перед дейликом накидал
что-то подобное:

```ts
// types/table.d.ts
import { HTMLAttributes } from "react";

export interface IDataTable {
  columng: string;
  rows: string[];
}

export interface ITableComponentProps
  extends Partial<HTMLAttributes<HTMLDivElement>> {
  header: string;
  data: IDataTable[];
}
```

```jsx
// TableComponent.jsx

/**
 * @returns {JSX.Element} Cool table Component
 *
 * @param {import("types/table").ITableComponentProps} props
 */
export function TableComponent({ header, data = [], ...divAttrs }) {
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

```jsx
// root.jsx

/** @type {import("types/table").IDataTable[]} */
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

#### Какого хрена здесь происходит?!

Тихо-тихо, сейчас всё расскажу.
