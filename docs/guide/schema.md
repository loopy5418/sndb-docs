# Schema changes

Tables can be changed after they're created. Every change is atomic, validated against the existing rows,
and can be part of a [transaction](transactions.md).

## Columns

### Adding

```luau
users:addColumn("verified", { type = "bool", default = false })
users:addColumn("ref", { type = "uuid", auto = true }) -- existing rows get generated values
```

Existing rows are filled in from `auto` or `default`. Adding a required column without either fails if the
table has rows. Passing a new column to [`db:table`](tables.md#creating-tables) does the same thing.

### Dropping and renaming

```luau
users:dropColumn("bio")
users:renameColumn("name", "fullName")
```

The primary key can't be dropped.

### Altering

`alterColumn(name, definition, convert?)` replaces a column's definition and revalidates every value.
`convert(value, row)` lets you transform values on the way:

```luau
-- string ages -> ints
users:alterColumn("age", { type = "int", min = 0 }, function(v)
	return tonumber(v)
end)

-- make a column optional
users:alterColumn("bio", "string?")
```

If a value ends up `nil` in a required column, it's filled from the new `auto` or `default`. You can't change
which column is the primary key.

## Indexes

```luau
users:createIndex("age")            -- a plain index
users:createIndex("email", true)    -- a unique index; fails if there are duplicates
users:dropIndex("age")
```

`dropIndex` also removes `unique`. The primary key's index can't be dropped.

## Tables

```luau
db:renameTable("users", "members")
db:dropTable("sessions")
db:hasTable("users")   -- false
db:tables()            -- sorted table names
```

## Inspecting

`table:columnList()` returns each column's stored definition:

```luau
for _, c in users:columnList() do
	print(c.name, c.type, c.optional, c.unique)
end
```

!!! note
    With `sync = "journal"`, a schema change rewrites the database file instead of being appended to the
    journal.
