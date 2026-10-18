# Types

Every column has a type, which decides which values it accepts, how they're ordered, and how they're stored.

## Built-in types

| Type | Aliases | Accepts | Notes |
| ---- | ------- | ------- | ----- |
| `string` | `text` | strings | |
| `int` | `integer` | whole numbers up to ±2^53 | Can be `auto` (a counter). |
| `float` | `number` | any number except NaN | |
| `bool` | `boolean` | `true` / `false` | `false` sorts before `true`. |
| `blob` | | strings (binary data) | Base64-encoded in JSON files. |
| `any` | `json` | `nil`, booleans, numbers, strings and tables | Tables are deep-copied on read and write. |
| `datetime` | | a number of seconds since the Unix epoch | Strings are parsed (e.g. `"2026-10-18T12:00:00Z"`). `auto` stores the current time. |
| `uuid` | | UUID strings | `auto` generates a UUIDv7. |
| `email` | | email address strings | |
| `enum(...)` | | one of the given strings | Sorts in declaration order. |
| `array(of)` | | arrays whose items are all of type `of` | `of` defaults to `any`. |

Append `?` to a type name to make the column optional: `"string?"`, `"datetime?"`.

### Enums and arrays

`enum` and `array` take arguments. Create them with `sndb.enum` and `sndb.array`, or write them as
`{ name, ...args }`:

```luau
db:table("tasks", {
	status = { type = sndb.enum("todo", "doing", "done"), default = "todo" },
	priority = { "enum", "low", "high" },
	tags = sndb.array("string"),
	points = { type = sndb.array("int"), max = 10 }, -- at most 10 items
})
```

Because enums sort in declaration order, `orderBy("status")` gives `todo`, `doing`, `done`, and
`where("status", "<", "done")` matches the first two.

### Datetimes

A `datetime` is stored as a float number of seconds (with sub-second precision). You can write either a
number or a string the [`datetime`](https://sonata.5418.org/docs) library can parse, and the same goes for
filters:

```luau
local events = db:table("events", {
	name = "string",
	at = { type = "datetime", index = true },
	created = { type = "datetime", auto = true },
})

events:insert({ name = "launch", at = "2026-11-01T09:00:00Z" })
events:where("at", ">=", "2026-10-01T00:00:00Z"):get()
```

## Custom types

`sndb.define(name, spec)` registers a new type. A custom type extends a **base** type (`any` if not given)
and can add a check, change the ordering, or convert values to and from the base type for storage.

```luau
sndb.define("slug", {
	base = "string",
	check = function(v)
		return string.match(v, "^[a-z0-9%-]+$") ~= nil, "expected a slug"
	end,
})

db:table("pages", { slug = { type = "slug", unique = true } })
```

### Spec fields

| Field | Description |
| ----- | ----------- |
| `base` | The type this one is built on: a name, type object or `{ name, ...args }`. Defaults to `"any"`. |
| `check` | `(value) -> (ok, err?)`. Without `serialize`, it runs after the base type's check. |
| `compare` | `(a, b) -> boolean`, true when `a` sorts before `b`. Used by `orderBy`, `<`, `between`, `min`/`max`. |
| `coerce` | `(value) -> value`, applied before checking, on writes and filter values. |
| `serialize` | `(value) -> base value`, converts a value into the base type for storage. Requires `deserialize`. |
| `deserialize` | `(base value) -> value`, the reverse of `serialize`. |
| `generate` | `() -> value`, lets the column be `auto`. |

### Converting values

With `serialize`/`deserialize`, a column can hold values the base type can't, such as tables, while being
stored compactly as the base type. The serialised form is also what's used for equality, indexes and
`unique`, so two values are equal if they serialise the same way.

```luau
sndb.define("vec2", {
	base = "string",
	check = function(v)
		return type(v) == "table" and type(v.x) == "number" and type(v.y) == "number", "expected {x, y}"
	end,
	serialize = function(v)
		return `{v.x},{v.y}`
	end,
	deserialize = function(s)
		local x, y = string.match(s, "^(.-),(.*)$")
		return { x = tonumber(x), y = tonumber(y) }
	end,
	compare = function(a, b) -- optional; defaults to comparing the serialised values
		return a.x * a.x + a.y * a.y < b.x * b.x + b.y * b.y
	end,
})

local points = db:table("points", { pos = { type = "vec2", unique = true } })
points:insert({ pos = { x = 1, y = 2 } })
points:where("pos", { x = 1, y = 2 }):first()
```

When `serialize` is given, the base type's check isn't run on the original value, so put every rule in
`check`.

### Parameterised types

Pass a function instead of a spec to make a type that takes arguments. The function receives the arguments
and returns a spec:

```luau
sndb.define("money", function(currency)
	return {
		base = "int",
		check = function(v)
			return v >= 0, `negative {currency}`
		end,
	}
end)

db:table("orders", {
	total = { type = { "money", "EUR" } },
	-- or: total = sndb.type("money", "EUR")
})
```

The type name and its arguments are saved with the table, so arguments should be plain values (strings,
numbers, booleans or other types).

### Defining types before opening

A database file stores only the names (and arguments) of the types its columns use. Define every custom
type before calling `sndb.open` on a database that uses it, or opening fails with `unknown type`.

When a database is opened, sndb also checks that each column's type still has the same underlying storage
kind (for example `string` or `int`) as when it was saved.

## Type objects

`sndb.type(name, ...)` returns the resolved type object, which you can pass anywhere a type is accepted.
`sndb.types.list()` returns the names of every registered type. See [`sndb`](../reference/sndb.md).
