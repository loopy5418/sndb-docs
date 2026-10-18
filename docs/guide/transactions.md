# Transactions

Every operation in sndb is atomic on its own: an `insertMany`, mass `update`, or `alterColumn` either applies
completely or not at all. To make several operations atomic together, run them in a transaction:

```luau
db:transaction(function(db)
	local from = db.accounts:get(1)
	local to = db.accounts:get(2)
	if from.balance < 100 then
		error("insufficient funds")
	end
	db.accounts:update(1, { balance = from.balance - 100 })
	db.accounts:update(2, { balance = to.balance + 100 })
end)
```

If the function raises an error, every change it made is undone, including created, renamed or dropped
tables and column changes, and the error is raised again. Otherwise the changes are committed together.

`transaction` returns whatever the function returns:

```luau
local order = db:transaction(function()
	local o = db.orders:insert({ user = 1, total = 25 })
	db.users:query():where("id", 1):increment("orders")
	return o
end)
```

## Nesting

Transactions can be nested. An inner transaction acts as a savepoint: if it fails and you catch the error,
only the inner changes are undone and the outer transaction continues.

```luau
db:transaction(function()
	db.logs:insert({ msg = "start" })
	local ok = pcall(db.transaction, db, function()
		db.logs:insert({ msg = "risky" })
		error("nope")
	end)
	-- "risky" was rolled back, "start" is kept
end)
```

## Durability

Changes are written when the outermost transaction commits, following the database's
[sync mode](storage.md#sync-modes). `db:save()` can't be called inside a transaction.
