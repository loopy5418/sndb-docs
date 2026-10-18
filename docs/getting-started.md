# Getting started

## Installation

sndb is a Sonata package. Add it to your project with the package manager:

```sh
sn pkg add github.com/loopy5418/sndb
```

This adds it to the `dependencies` of `.sonata/project.luau` and installs it into `.sonata/deps/sndb`.
Require it by name:

```luau
local sndb = require("@sndb")
```

To pin a version, add `#<ref>` (a tag, branch or commit) to the URL: `sn pkg add github.com/loopy5418/sndb#v0.1.0`.

## Your first database

```luau
local sndb = require("@sndb")

-- Opens app.sndb, creating it on the first save if it doesn't exist
local db = sndb.open("app.sndb")

-- Creates the table, or returns it if it already exists
local posts = db:table("posts", {
	title = { type = "string", max = 120 },
	body = "text",
	tags = sndb.array("string"),
	published = { type = "bool", default = false },
	created = { type = "datetime", auto = true },
})

local post = posts:insert({ title = "Hello", body = "First post", tags = { "intro" } })
print(post.id, post.created) -- 1, 1760799600.123

posts:update(post.id, { published = true })

for _, p in posts:where("published", true):orderBy("created", "desc"):get() do
	print(p.title)
end

db:close()
```

A few things to notice:

- Every table has a **primary key**. `posts` didn't declare one, so sndb added an auto-incrementing `id` column.
- `db:table(name, columns)` is safe to call every time your program starts: it creates the table the first
  time and afterwards returns the existing one (adding any columns you've added to the definition).
- Rows you get back are **copies**. Changing them doesn't change the database; use `update` for that.
- With the default `sync = "journal"`, each change is appended to `app.sndb.journal` as soon as it's made.
  `db:close()` folds the journal back into `app.sndb`.

## In-memory databases

`sndb.memory()` gives you a database that is never written to disk, which is handy for tests and caches.
You can still save a copy of it with [`db:backup`](reference/database.md#backup):

```luau
local db = sndb.memory()
-- ...
db:backup("snapshot.json", { format = "json", pretty = true })
```

## Running the tests

From a checkout of the [sndb repository](https://github.com/loopy5418/sndb):

```sh
sn run --entry tests/run.luau
sn run --entry tests/bench.luau -- 100000
```

## Next steps

- [Tables & columns](guide/tables.md): how to define a schema.
- [Queries](guide/queries.md): filtering, ordering and aggregates.
- [Storage](guide/storage.md): formats, compression, encryption and sync modes.
