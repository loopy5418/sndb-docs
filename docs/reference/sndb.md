# sndb

The module returned by `require("@sndb")`.

```luau
local sndb = require("@sndb")
```

## Functions

### `open`

```luau
sndb.open(path: string, options: Options?) -> Database
```

Opens the database at `path`, loading it if it exists and replaying any leftover journal. See
[Storage](../guide/storage.md#options) for the options.

### `memory`

```luau
sndb.memory() -> Database
```

Creates an in-memory database that is never written to disk. Use [`db:backup`](database.md#backup) to save a copy.

### `define`

```luau
sndb.define(name: string, spec: Spec | (...any) -> Spec)
```

Registers a [custom type](../guide/types.md#custom-types). Pass a function to make a parameterised type.
Defining a name again replaces the previous definition.

### `type`

```luau
sndb.type(name: string, ...: any) -> Type
```

Returns the type object for `name`, applying any arguments: `sndb.type("money", "EUR")`. Raises an error for
unknown types.

### `enum`

```luau
sndb.enum(...: string) -> Type
```

An `enum` type accepting the given strings, ordered as given.

### `array`

```luau
sndb.array(of: string | Type?) -> Type
```

An `array` type whose items are all of type `of` (default `any`).

## Values

### `null`

A sentinel meaning "no value". Use it to clear a column in an update, or to store `nil` in an optional column
that has a default. `groupBy` and `countBy` also use it as the key for `nil` values.

```luau
users:update(1, { bio = sndb.null })
```

### `version`

The version of sndb, e.g. `"0.1.0"`.

### `types`

The type registry. Besides `define` and `get` (the same as `sndb.define` and `sndb.type`), it has:

| Function | Description |
| -------- | ----------- |
| `types.list() -> { string }` | The sorted names of every registered type, aliases included. |
| `types.resolve(spec) -> Type` | Resolves a name, type object or `{ name, ...args }` to a type object. |
