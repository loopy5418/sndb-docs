# Query

Created by `table:query()` or by calling any of these methods on a table. Building methods return a new
query; the original is unchanged.

## Filtering

| Method | Description |
| ------ | ----------- |
| `where(col, value)` | `col` equals `value`. A function `value` is called with the column's value and must return `true`. |
| `where(col, op, value)` | `col` compared with `value` using an [operator](operators.md). |
| `where({ [col]: value })` | Every column equals its value. |
| `where(fn)` | `fn(row)` returns `true`. |
| `where(query)` | The rows matched by another query on the same table (for grouping). |
| `orWhere(...)` | Starts a new alternative group: matches rows matched by the previous groups **or** this one. |
| `whereNot(...)` | The given conditions don't all hold. |
| `orWhereNot(...)` | `orWhere` + `whereNot`. |
| `whereIn(col, list)` | `col` is one of `list`. |
| `whereNotIn(col, list)` | `col` is none of `list`. |
| `whereBetween(col, lo, hi)` | `lo <= col <= hi`. |
| `whereNull(col)` | `col` is `nil`. |
| `whereNotNull(col)` | `col` isn't `nil`. |
| `search(text, options?)` | Full-text search, ranked by relevance. See [Search](../guide/search.md). |

## Shaping

| Method | Description |
| ------ | ----------- |
| `orderBy(col, dir?)` | Sort by a column, `"asc"` (default) or `"desc"`. Can be chained. |
| `orderBy(fn)` | Sort with a comparator `fn(a, b) -> boolean` on rows. |
| `limit(n)` | At most `n` rows. |
| `offset(n)` | Skip the first `n` rows. |
| `page(n, size)` | Page `n` (starting at 1) of `size` rows. |
| `select(...)` | Only include these columns in returned rows. |

## Reading

| Method | Returns |
| ------ | ------- |
| `get()`, `all()` | `{ Row }` |
| `first()` | `Row?` |
| `count()` | `number` |
| `exists()` | `boolean` |
| `each(fn)` | Calls `fn(row)` for each row. |
| `pluck(col)` | `{ any }`, one column's values. |
| `distinct(col)` | `{ any }`, distinct non-nil values. |
| `sum(col)` | `number` |
| `avg(col)` | `number?` |
| `min(col)`, `max(col)` | The smallest / largest non-nil value, or `nil`. |
| `groupBy(col)` | `{ [value]: { Row } }`, with `nil` values under `sndb.null`. |
| `countBy(col)` | `{ [value]: number }`, with `nil` values under `sndb.null`. |
| `paginate(page, size)` | `{ items, page, size, total, pages }` |

## Editing

All editing methods are atomic and return the number of rows affected.

| Method | Description |
| ------ | ----------- |
| `update(changes)` | Sets columns on every matching row. Function values are called as `fn(oldValue, row)` and return the new value. |
| `update(fn)` | Calls `fn(row)` with a copy of each row; it returns a table of changes, or edits the copy and returns nothing. |
| `increment(col, n?)` | Adds `n` (default 1); `nil` counts as 0. |
| `decrement(col, n?)` | Subtracts `n` (default 1). |
| `delete()` | Deletes every matching row. |

On tables, `update`, `delete` and `get` take a primary key (see [Table](table.md)); start from
`table:query()` or a `where` to use the query versions.
