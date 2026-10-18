# Errors

sndb raises errors as plain strings prefixed with `sndb: `, without a position. Catch them with `pcall`:

```luau
local ok, err = pcall(users.insert, users, { name = "Ada", email = "not an email" })
if not ok then
	print(err) -- sndb: users.email: expected an email address
end
```

A failed operation never leaves a partial change behind: everything it did is rolled back first.

## Common errors

| Message | Cause |
| ------- | ----- |
| `<table>.<col> is required` | A required column has no value, default or `auto`. |
| `<table>.<col>: expected <type>, got <type>` | A value has the wrong type. |
| `<table>.<col>: <message>` | A type or column `check` failed, with its message. |
| `<table>.<col>: duplicate value <v>` | A `unique` column (or the primary key) already has this value. |
| `<table>.<col>: <n> is below the minimum of <min>` | A `min` constraint failed (`above the maximum` for `max`). |
| `<table>.<col>: doesn't match the pattern <p>` | A `pattern` constraint failed. |
| `<table>: unknown column '<col>'` | A row, change or query refers to a column that doesn't exist. |
| `<table>: no row with <pk> = <v>` | `table:update` was given a primary key that doesn't exist. |
| `no table named '<name>'` | `db:table(name)` without columns, or `dropTable`/`renameTable` on a missing table. |
| `table '<name>' already exists` | `createTable` with a name that's taken. |
| `unknown type '<name>'` | A column uses a type that isn't defined. Define custom types before opening. |
| `unknown operator '<op>'` | See [Operators](operators.md). |
| `<path> is encrypted; open it with a password` | The database needs a `password`. |
| `can't read <path>: ...` | The file is corrupt, or the password is wrong. |
| `can't save inside a transaction` | `db:save()` was called inside `db:transaction`. |
