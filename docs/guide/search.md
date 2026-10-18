# Full-text search

`search(text, options?)` finds rows containing the words in `text` and ranks them by relevance.

```luau
local results = posts:search("luau database"):limit(10):get()
```

The text is split into words (lowercased, punctuation removed). By default, a row matches if **every** word
appears in at least one of the searched columns. It can be combined with any other query method:

```luau
posts:where("published", true):search("sonata", { columns = { "title", "body" } }):paginate(1, 20)
```

## Options

| Option | Default | Description |
| ------ | ------- | ----------- |
| `columns` | every `string`-like and `any` column | The columns to search. |
| `mode` | `"all"` | `"all"`: every word must match. `"any"`: at least one word must match. |
| `fuzzy` | `false` | Also match words with small typos. `true` allows 1 edit for words of up to 4 letters and 2 for longer ones; a number sets the allowed edit distance. |
| `weights` | `{}` | A multiplier per column, e.g. `{ title = 3 }` to rank title matches higher. |

Array columns are searched too: their string and number items are joined into one text.

## Ranking

Each word is scored against each column and the best column wins:

| Match | Score |
| ----- | ----- |
| The whole value equals the word | 4 |
| The word starts a word in the value | 2 |
| The word appears inside the value | 1 |
| A fuzzy match | 0.5 |

Scores are multiplied by the column's weight and added up over all words. Rows are returned from the highest
score to the lowest, with ties in insertion order. Adding an `orderBy` replaces the relevance order.

```luau
posts:search("databse", { fuzzy = true, weights = { title = 3 }, mode = "any" }):get()
```

An empty search (no words) matches every row.
