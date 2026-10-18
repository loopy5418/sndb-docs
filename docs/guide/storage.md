# Storage

`sndb.open(path, options?)` loads a database from `path`, or starts an empty one that's written there on the
first save.

```luau
local db = sndb.open("data/app.sndb", {
	format = "binary",
	compression = "zstd",
	level = "best",
	layout = "file",
	sync = "journal",
})
```

## Options

| Option | Values | Default |
| ------ | ------ | ------- |
| `format` | `"binary"` (columnar, compact) or `"json"` (readable) | `"binary"` |
| `compression` | `"zstd"`, `"gzip"`, `"zlib"`, `"deflate"`, `"none"` | `"zstd"` for binary (`"gzip"` if zstd isn't available), `"none"` for JSON |
| `level` | a number, `"fast"`, `"default"` or `"best"` | the compressor's default |
| `layout` | `"file"` (one file) or `"dir"` (a folder with a file per table) | `"dir"` if `path` is an existing folder, otherwise `"file"` |
| `password` | a string; encrypts the data and the journal | |
| `sync` | `"journal"`, `"full"` or `"manual"`, see [below](#sync-modes) | `"journal"` |
| `journalLimit` | journal size in bytes before it's compacted | 4 MiB |
| `pretty` | `true` to indent JSON | |

**Existing files are detected**: when you open a database, its format and compression are read from the file,
so you only need `password` to reopen one. Passing `format` or `compression` for an existing database
converts it to those settings on the next save.

## Formats

### Binary

The binary format stores each column separately, which compresses well:

- integers as delta-encoded varints,
- floats as 32-bit when that's lossless, 64-bit otherwise,
- repeated strings dictionary-encoded,
- booleans and nulls as bitmaps,

and the result is compressed (zstd by default).

### JSON

JSON is useful for debugging, diffs, and data you want to edit by hand:

```luau
local db = sndb.open("app.json", { format = "json", pretty = true })
```

Without compression or a password, the file is plain JSON:

```json
{
  "sndb": 1,
  "tables": {
    "users": {
      "schema": { "columns": [ ... ], "auto": 3 },
      "rows": [ { "id": 1, "name": "Ada", ... } ]
    }
  }
}
```

`blob` values are base64-encoded. With compression or a password, the JSON is wrapped in the same compressed
container as the binary format.

## Layouts

With `layout = "file"` (the default), the whole database is one file and its journal is `<path>.journal`.

With `layout = "dir"`, `path` is a folder containing one file per table (`<table>.sndb`, or `<table>.json`
for plain JSON) and a `journal` file. Saving only rewrites the tables that changed, which is faster for large
databases where most tables are rarely touched.

```luau
local db = sndb.open("data", { layout = "dir" })
```

Files are written to a temporary file and then moved into place, so a crash during a save doesn't leave a
half-written file.

## Sync modes

`sync` decides when changes reach the disk.

=== "journal (default)"

    Each committed change is appended to a journal file right away, which is cheap no matter how big the
    database is. The journal is folded into the main file (and deleted) when you call `save()` or `close()`,
    when it grows past `journalLimit`, or after a schema change.

    If the program stops without closing the database, the journal is replayed the next time it's opened.

=== "full"

    The whole database is rewritten after every committed change. Simple and always compact, but slow for
    large databases with frequent writes.

=== "manual"

    Nothing is written until you call `db:save()` or `db:close()`. Fastest, but changes since the last save
    are lost if the program stops.

Always call `db:close()` when you're done: it saves if anything has changed since the last save.

## Encryption

```luau
local db = sndb.open("secret.sndb", { password = os.getenv("DB_KEY") })
```

With a password, the database file and every journal entry are encrypted. Opening an encrypted database
without the password, or with the wrong one, raises an error.

## Backups and in-memory databases

`db:backup(path, options?)` writes a complete copy of the database to `path`, in any format and layout,
without changing where the database itself is saved:

```luau
db:backup("backup.json", { format = "json", pretty = true })
db:backup("backup.sndb", { compression = "zstd", level = "best", password = "..." })
```

`sndb.memory()` creates a database that is never written; `backup` is how you save one.

## Stats

```luau
local s = db:stats()
-- s.path, s.layout, s.format, s.compression, s.sync,
-- s.journal (bytes in the journal), s.tables.users.rows, s.tables.users.columns
```
