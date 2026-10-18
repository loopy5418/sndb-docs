# Table

Returned by [`db:table`](database.md#table). Tables also have every [Query](query.md) method except those
listed below under the same name, which take a primary key instead.

Fields: `table.name` (the table's name).

All returned rows are copies.

## Rows

### `insert`

```luau
table:insert(row: { [string]: any }) -> Row
```

Validates and inserts a row, filling in defaults and generated values. Returns the stored row.

### `insertMany`

```luau
table:insertMany(rows: { { [string]: any } }) -> { Row }
```

Inserts all rows, or none if any of them fails.

### `get`

```luau
table:get(pk: any) -> Row?
```

The row with primary key `pk`, or `nil`.

### `has`

```luau
table:has(pk: any) -> boolean
```

### `update`

```luau
table:update(pk: any, changes: { [string]: any }) -> Row
```

Changes the given columns of the row with primary key `pk` and returns it. Use `sndb.null` to clear a value.
Raises an error if there's no such row.

### `upsert`

```luau
table:upsert(row: { [string]: any }, on: string?) -> Row
```

Updates the row whose column `on` (default: the primary key) equals `row[on]` with the values in `row`, or
inserts `row` if there's none.

### `delete`

```luau
table:delete(pk: any) -> boolean
```

Deletes the row with primary key `pk`. Returns whether a row was deleted.

### `clear`

```luau
table:clear() -> number
```

Deletes every row. Returns how many were deleted.

### `query`

```luau
table:query() -> Query
```

A query matching every row.

## Schema

### `addColumn`

```luau
table:addColumn(name: string, def: ColumnDef)
```

Adds a column, filling existing rows from `auto` or `default`. Can't add a second primary key.

### `dropColumn`

```luau
table:dropColumn(name: string)
```

Can't drop the primary key.

### `renameColumn`

```luau
table:renameColumn(from: string, to: string)
```

### `alterColumn`

```luau
table:alterColumn(name: string, def: ColumnDef, convert: ((value: any, row: Row) -> any)?)
```

Replaces a column's definition, passing each value through `convert` first if given, and revalidates every
row. Can't change which column is the primary key.

### `createIndex`

```luau
table:createIndex(column: string, unique: boolean?)
```

### `dropIndex`

```luau
table:dropIndex(column: string)
```

Removes the index and `unique` constraint. Can't be used on the primary key.

### `columnList`

```luau
table:columnList() -> { ColumnInfo }
```

Each column's stored definition, primary key first: `name`, `type` (a name, or `{ name, ...args }`), `kind`
(the storage kind), and `optional`, `primary`, `unique`, `index`, `auto`, `min`, `max`, `pattern` and
`default` where set.

## Column definitions

A `ColumnDef` is a type name (`"int"`, `"string?"`), a type object, `{ typeName, ...args }`, or a table of
options:

| Option | Type |
| ------ | ---- |
| `type` | type name, type object, or `{ name, ...args }` |
| `optional` | `boolean` |
| `default` | any value, or `() -> any` |
| `primary` | `boolean` |
| `unique` | `boolean` |
| `index` | `boolean` |
| `auto` | `boolean` |
| `min`, `max` | `number` |
| `pattern` | Lua pattern `string` |
| `check` | `(value) -> (boolean, string?)` |

See [Tables & columns](../guide/tables.md#column-definitions).
