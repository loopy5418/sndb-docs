# Database

Returned by [`sndb.open`](sndb.md#open) and [`sndb.memory`](sndb.md#memory).

Tables can be accessed as fields: `db.users` is the same as `db:table("users")`, and is `nil` when there's
no such table.

## Tables

### `table`

```luau
db:table(name: string, columns: { [string]: ColumnDef }?) -> Table
```

Returns the table called `name`. If it doesn't exist, it's created from `columns` (an error if `columns`
isn't given). If it does exist and `columns` is given, columns that are missing are added and
[runtime-only options](../guide/tables.md#runtime-only-options) are re-attached.

### `createTable`

```luau
db:createTable(name: string, columns: { [string]: ColumnDef }?) -> Table
```

Creates a table, raising an error if one with that name already exists.

### `hasTable`

```luau
db:hasTable(name: string) -> boolean
```

### `tables`

```luau
db:tables() -> { string }
```

The names of all tables, sorted.

### `renameTable`

```luau
db:renameTable(from: string, to: string)
```

### `dropTable`

```luau
db:dropTable(name: string)
```

Deletes a table and all its rows.

## Transactions

### `transaction`

```luau
db:transaction(fn: (db: Database) -> ...any) -> ...any
```

Runs `fn` so that all of its changes apply or none do, and returns what it returns. Nested calls act as
savepoints. See [Transactions](../guide/transactions.md).

## Persistence

### `save`

```luau
db:save()
```

Writes every changed table and clears the journal. Does nothing for in-memory databases. Raises an error
inside a transaction.

### `close`

```luau
db:close()
```

Saves if anything changed since the last save. The database can still be used afterwards.

### `backup`

```luau
db:backup(path: string, options: StoreOptions?)
```

Writes a full copy of the database to `path`. `options` are the [storage options](../guide/storage.md#options)
`format`, `compression`, `level`, `layout`, `password` and `pretty`.

### `stats`

```luau
db:stats() -> Stats
```

| Field | Description |
| ----- | ----------- |
| `tables` | `{ [name]: { rows: number, columns: number } }` |
| `sync` | The sync mode. |
| `journal` | Bytes appended to the journal since the last save. |
| `path`, `layout`, `format`, `compression` | Storage settings (absent for in-memory databases). |
