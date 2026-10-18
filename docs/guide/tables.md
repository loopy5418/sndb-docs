# Tables & columns

A database holds named **tables**; each table has a fixed set of typed **columns** and holds **rows**,
which are plain Luau tables mapping column names to values.

## Creating tables

```luau
local users = db:table("users", {
	name = "string",
	email = { type = "email", unique = true },
	age = { type = "int", min = 0, index = true },
	bio = "string?",
})
```

[`db:table(name, columns)`](../reference/database.md#table) creates the table if it doesn't exist. If it
does, it returns it, adds any columns that are in `columns` but not yet in the table, and re-attaches
[runtime-only options](#runtime-only-options). Use [`db:createTable`](../reference/database.md#createtable)
instead if you want an error when the table already exists.

Without `columns`, `db:table(name)` returns an existing table or raises an error. Tables are also available
as fields of the database: `db.users`.

Table names start with a letter and contain letters, digits and `_`.

## Column definitions

A column is defined by either a **type** or a table of **options**.

```luau
{
	name = "string",                     -- a type name
	bio = "string?",                     -- `?` makes it optional
	role = sndb.enum("user", "admin"),   -- a type object
	price = { "money", "EUR" },          -- a parameterised custom type
	email = { type = "email", unique = true },
}
```

| Option     | Description |
| ---------- | ----------- |
| `type`     | The column's [type](types.md): a name (`"int"`, `"string?"`), a type object (`sndb.enum(...)`), or `{ name, ...args }` for a parameterised type. |
| `optional` | Allow `nil`. Same as a trailing `?` on the type name. |
| `default`  | The value used when an inserted row leaves the column out. A function is called for each row. Table values are deep-copied. |
| `primary`  | Make this the primary key. Implies `unique`; a primary key can't be optional. |
| `unique`   | No two rows may share a value. Implies `index`. `nil` values don't count. |
| `index`    | Keep a hash index so [equality and `in` filters](queries.md#indexes) are fast. |
| `auto`     | Fill the value automatically: an increasing counter for `int` columns, or a generated value for types that have a generator (`uuid`, `datetime`). |
| `min`, `max` | Bounds, inclusive. For numbers the value is checked; for strings and arrays, the length. |
| `pattern`  | A Lua pattern that string values must contain (anchor it with `^...$` to match the whole value). |
| `check`    | A function `(value) -> (ok, err?)` for any other rule. |

Column names start with a letter or `_` and contain letters, digits and `_`.

### Primary keys

Each table has exactly one primary column. If none is marked `primary`, sndb adds:

```luau
id = { type = "int", primary = true, auto = true }
```

If you define a column called `id` yourself, it must be the primary key.

Any type can be the primary key; a `uuid` key is a common alternative to integers:

```luau
db:table("sessions", {
	id = { type = "uuid", primary = true, auto = true },
	user = "int",
})
```

A table can have only one auto-incrementing `int` column. The counter never reuses a value, even after
rows are deleted.

### Required, optional and defaults

When you insert a row, each column the row leaves out is filled from `auto`, then `default`. If there is
still no value and the column isn't optional, the insert fails:

```luau
users:insert({ email = "kim@example.com", age = 3 }) -- error: sndb: users.name is required
```

To store `nil` explicitly in an optional column that has a default, pass [`sndb.null`](../reference/sndb.md#null).

### Validation

Every value is checked when it's written, by `insert`, `update`, `upsert`, and schema changes alike, in this
order: the type's coercion (e.g. parsing a datetime string), the type's check, `min`/`max`, `pattern`, then
`check`. A failed check raises an error and the operation changes nothing.

```luau
local users = db:table("users", {
	username = {
		type = "string",
		min = 3,
		max = 20,
		pattern = "^[%w_]+$",
		check = function(v)
			return v ~= "admin", "that name is reserved"
		end,
	},
})
```

## Runtime-only options

Column definitions are saved with the database, except for functions: `check` functions and function
defaults. Pass the same definition to `db:table` each time you open the database to re-attach them:

```luau
local function open()
	local db = sndb.open("app.sndb")
	db:table("users", USER_COLUMNS) -- re-attaches check/default functions
	return db
end
```

Custom types are also defined in code, so call [`sndb.define`](types.md#custom-types) before opening a
database that uses them.

## Column order

Columns are stored with the primary key first, followed by the others in alphabetical order.
[`table:columnList()`](../reference/table.md#columnlist) returns them in that order.
