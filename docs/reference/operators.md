# Operators

Operators are used as `where(col, op, value)`. They're case-insensitive, and spaces and underscores are
ignored, so `"not in"`, `"notIn"` and `"NOT_IN"` are the same operator.

## Comparison

| Operator | Aliases | Matches when the value… |
| -------- | ------- | ----------------------- |
| `=` | `==`, `is` | equals `value` |
| `!=` | `~=`, `<>`, `is not` | doesn't equal `value` (including `nil` values) |
| `<` | | is less than `value` |
| `<=` | `lte` | is less than or equal to `value` |
| `>` | | is greater than `value` |
| `>=` | `gte` | is greater than or equal to `value` |
| `between` | | is within `{ lo, hi }`, inclusive |
| `not between` | | isn't within `{ lo, hi }` |
| `in` | | is one of the items in the list `value` |
| `not in` | | is none of the items in `value` (including `nil` values) |

Ordering comparisons use the column type's ordering and never match `nil` values.

## Text

| Operator | Matches when the value… |
| -------- | ----------------------- |
| `like` | matches an SQL-style pattern: `%` is any run of characters, `_` is one character; the whole value must match |
| `ilike` | `like`, ignoring case |
| `not like` | is a string that doesn't match the pattern |
| `contains` (`has`) | contains `value` as a substring, or, for arrays, has `value` as an item |
| `icontains` | `contains`, ignoring case |
| `startsWith` | starts with `value` |
| `endsWith` | ends with `value` |
| `match` | contains a match for the Lua pattern `value` |
| `regex` | matches the regular expression `value` |
| `iregex` | `regex`, ignoring case |

```luau
users:where("email", "like", "%@example.com")
users:where("name", "ilike", "a%")
posts:where("tags", "contains", "luau")
users:where("phone", "match", "^%+44")
users:where("name", "regex", [[^\w+ \w+$]])
```

## Null

| Operator | Aliases | Matches when the value… |
| -------- | ------- | ----------------------- |
| `null` | `is null` | is `nil` |
| `not null` | `is not null` | isn't `nil` |

These ignore their operand, but one must still be passed: `where(col, "null", true)`. With only two
arguments, `where(col, "null")` compares with the string `"null"`. Prefer `whereNull(col)` and
`whereNotNull(col)`.
