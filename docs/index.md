---
hide:
  - navigation
---

# sndb

**sndb** is an embedded, typed database for [Sonata](https://sonata.5418.org), written in Luau, for Luau.
It keeps your data in memory, validates every value against a schema, and writes it to disk in a compact
column format, with a journal so each change is durable without rewriting the whole file.

```luau
local sndb = require("@sndb")

local db = sndb.open("app.sndb")
local users = db:table("users", {
	name = "string",
	email = { type = "email", unique = true },
	age = { type = "int", min = 0, index = true },
	role = { type = sndb.enum("user", "admin"), default = "user" },
	bio = "string?",
})

users:insert({ name = "Ada", email = "ada@x.io", age = 36 })
local admins = users:where("role", "admin"):orderBy("age", "desc"):get()
users:query():where("age", "<", 18):update({ role = "user" })
db:close()
```

<div class="grid cards" markdown>

-   :material-shield-check-outline: **Typed columns**

    ---

    Built-in types such as `int`, `email`, `uuid`, `datetime`, `enum` and `array`, plus constraints
    (`unique`, `min`/`max`, `pattern`, `check`) and your own [custom types](guide/types.md#custom-types).

-   :material-filter-variant: **Chainable queries**

    ---

    `where`, `orWhere`, 25+ [operators](reference/operators.md), ordering, pagination, aggregates and
    [mass updates](guide/editing.md#mass-editing), using indexes where possible.

-   :material-text-search: **Full-text search**

    ---

    Rank rows by relevance across text columns, with per-column weights and
    [fuzzy matching](guide/search.md).

-   :material-backup-restore: **Transactions**

    ---

    Every operation is atomic, and [`db:transaction`](guide/transactions.md) rolls back everything,
    schema changes included, if anything fails.

-   :material-harddisk: **Compact storage**

    ---

    A columnar binary format with zstd compression, or readable JSON; one file or a file per table;
    optional [encryption](guide/storage.md#encryption).

-   :material-feather: **Small & Light**

    ---

    About 3,400 lines of Luau. No external dependencies.

</div>

[Get started :material-arrow-right:](getting-started.md){ .md-button .md-button--primary }
[API reference](reference/sndb.md){ .md-button }
