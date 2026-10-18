# Editing rows

## Inserting

```luau
local ada = users:insert({ name = "Ada", email = "ada@x.io", age = 36 })
print(ada.id) -- the generated primary key

local rows = users:insertMany({
	{ name = "Grace", email = "grace@x.io", age = 45 },
	{ name = "Alan", email = "alan@x.io", age = 41 },
})
```

`insert` returns a copy of the stored row, with defaults and generated values filled in. `insertMany` inserts
every row or, if any of them is invalid, none of them.

Unknown columns are an error, as are values that fail [validation](tables.md#validation).

## Reading by primary key

```luau
local row = users:get(1)     -- the row, or nil
local found = users:has(1)   -- boolean
```

## Updating

```luau
users:update(1, { age = 37 })                  -- returns the updated row
users:update(1, { bio = sndb.null })           -- set a column to nil
```

`update(pk, changes)` changes only the columns in `changes` and raises an error if no row has that primary
key. Because `nil` can't be stored in a Luau table, use `sndb.null` to clear a column.

### Upserting

`upsert(row, on?)` updates the row whose `on` column (the primary key by default) has the same value as
`row[on]`, or inserts `row` if there's none:

```luau
users:upsert({ email = "ada@x.io", name = "Ada Lovelace", age = 36 }, "email")
```

`on` is usually a `unique` column; for other columns, the first matching row is updated.

## Deleting

```luau
users:delete(1)   -- true if a row was deleted
users:clear()     -- deletes every row, returns how many
```

## Mass editing

Queries can update or delete every row they match. Each call is atomic: if one row fails validation, no row
is changed.

```luau
users:query():where("age", "<", 18):update({ role = "user" })   -- returns the number of rows
users:where("active", false):delete()
users:query():increment("visits")                               -- +1
users:where("id", 7):decrement("credits", 5)                    -- -5
```

`limit`, `offset`, `orderBy` and `search` are respected, so you can update "the 10 oldest rows", for example.

### Computed changes

A function value in `changes` receives the column's current value and a copy of the row, and returns the
new value:

```luau
users:query():update({
	name = function(name) return string.upper(name) end,
	bio = function(_, row) return `{row.name}, {row.age}` end,
})
```

Or pass a function instead of a table. It gets a copy of each row and either returns a table of changes or
edits the copy in place and returns nothing:

```luau
users:where("role", "admin"):update(function(row)
	row.age += 1
	row.bio = nil -- in-place edits can clear values directly
end)

users:query():update(function(row)
	return { active = row.age < 90 }
end)
```

!!! note
    The table methods `update`, `delete` and `get` take a primary key. To update, delete or fetch with a
    filter, start from a query: `users:where(...)` or `users:query()`.
