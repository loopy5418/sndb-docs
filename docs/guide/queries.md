# Queries

Queries are built by chaining methods, then run by a reading method such as `get`, `first` or `count`.

```luau
local adults = users
	:where("age", ">=", 18)
	:where("role", "user")
	:orderBy("name")
	:limit(10)
	:get()
```

Every query method is available directly on a table (`users:where(...)`) and on queries. Each call returns
a **new** query and leaves the original unchanged, so queries can be stored and reused:

```luau
local active = users:where("active", true)
local activeAdmins = active:where("role", "admin"):get()
local activeCount = active:count()
```

`table:query()` returns a query that matches every row, which is useful for [mass edits](editing.md#mass-editing).

## Filtering

### `where`

`where` accepts several forms:

```luau
users:where("role", "admin")                 -- equality
users:where("age", ">", 30)                  -- an operator
users:where({ role = "admin", active = true }) -- several equalities
users:where(function(row) return #row.name > 3 end) -- a predicate on the row
users:where("name", function(v) return v:sub(1, 1) == "A" end) -- a predicate on one value
```

Chained `where` calls must all match (AND). See [Operators](../reference/operators.md) for every operator;
the common ones are `=`, `!=`, `<`, `<=`, `>`, `>=`, `in`, `between`, `like`, `contains` and `startsWith`.

Comparison operators use the column type's ordering, so they work for enums, datetimes and custom types.
Rows whose value is `nil` never match `<`, `>`, `between`, and the like.

Filter values are coerced like written values, so a datetime column can be compared with a date string.

### Shortcuts

| Method | Same as |
| ------ | ------- |
| `whereIn(col, list)` | `where(col, "in", list)` |
| `whereNotIn(col, list)` | `where(col, "not in", list)` |
| `whereBetween(col, lo, hi)` | `where(col, "between", { lo, hi })` |
| `whereNull(col)` | `where(col, "null", true)` |
| `whereNotNull(col)` | `where(col, "not null", true)` |

### OR and grouping

`orWhere` starts a new alternative. A query matches if every condition in **any one** group matches, so
conditions read left to right as `(A and B) or (C and D)`:

```luau
-- (role = admin AND active) OR (age > 60)
users:where("role", "admin"):where("active", true):orWhere("age", ">", 60)
```

To group conditions differently, pass a query on the same table to `where`:

```luau
-- active AND (role = admin OR age > 60)
users:where("active", true):where(users:where("role", "admin"):orWhere("age", ">", 60))
```

### Negation

`whereNot` and `orWhereNot` take the same arguments as `where` and match rows where the conditions
**don't** all hold:

```luau
users:whereNot("role", "admin")
users:whereNot({ role = "admin", active = true }) -- not (admin and active)
```

## Ordering and paging

```luau
users:orderBy("age", "desc"):orderBy("name")         -- age descending, then name ascending
users:orderBy(function(a, b) return #a.name < #b.name end) -- a custom comparator on rows
users:limit(20):offset(40)
users:page(3, 20)                                    -- same as above
```

- `orderBy(col, dir?)` sorts by a column, `"asc"` (default) or `"desc"`. `nil` values sort last.
- Without `orderBy`, rows come back in insertion order (or by relevance when [searching](search.md)).
- `page(n, size)` is a shortcut for `offset((n - 1) * size):limit(size)`.

`paginate(page, size)` runs the query and returns the page plus the totals:

```luau
local p = posts:where("published", true):orderBy("created", "desc"):paginate(2, 10)
-- p.items, p.page (2), p.size (10), p.total (matching rows), p.pages
```

## Selecting columns

`select(...)` limits which columns the returned rows contain:

```luau
users:select("id", "name"):get() -- { { id = 1, name = "Ada" }, ... }
```

## Reading results

| Method | Returns |
| ------ | ------- |
| `get()` / `all()` | An array of matching rows. |
| `first()` | The first matching row, or `nil`. |
| `count()` | The number of matching rows. |
| `exists()` | Whether any row matches. |
| `each(fn)` | Calls `fn(row)` for each matching row. |
| `pluck(col)` | An array of one column's values. |
| `distinct(col)` | The column's distinct non-nil values, in first-seen order. |
| `sum(col)` | The sum of a column (`nil` counts as 0). |
| `avg(col)` | The average of the non-nil values, or `nil` if there are none. |
| `min(col)`, `max(col)` | The smallest or largest value, using the column type's ordering. |
| `groupBy(col)` | A table mapping each value of `col` to an array of rows. |
| `countBy(col)` | A table mapping each value of `col` to the number of rows with it. |

In `groupBy` and `countBy`, rows whose value is `nil` are grouped under the key `sndb.null`.

```luau
local byRole = users:countBy("role") -- { user = 12, admin = 2 }
local oldest = users:max("age")
local emails = users:where("active", true):pluck("email")
```

All results are copies, so you can modify them freely.

!!! note "`get` on tables"
    On a table, `get(pk)` looks up a row by primary key (see [Editing rows](editing.md)). To fetch all rows
    of a table, use `users:all()` or `users:query():get()`.

## Indexes

Most queries scan the table. When a query has no `orWhere` and one of its conditions is an equality or
`in` filter on an indexed column (`index`, `unique` or the primary key), sndb looks the rows up in the
index instead and only checks the other conditions against those rows.

```luau
db:table("orders", { user = { type = "int", index = true }, total = "float" })

orders:where("user", 42):where("total", ">", 100):get() -- uses the index on `user`
```

Add indexes with the `index` column option or [`createIndex`](schema.md#indexes).
