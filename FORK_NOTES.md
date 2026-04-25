# Fork notes

This is a thin fork of [tursodatabase/go-libsql](https://github.com/tursodatabase/go-libsql) with one change: the unconditional date-parsing of TEXT columns has been removed from the row scanner.

## Why

Upstream's `TYPE_TEXT` handler tries to parse every TEXT value through a list of date/time formats and returns a `time.Time` on a match. When the caller scans into `*string`, `database/sql` formats the `time.Time` via RFC3339Nano, so a value stored as `"2024-01-20"` comes back as `"2024-01-20T00:00:00Z"`. This silently corrupts the format of every date string in apps that store dates as TEXT, which is the common SQLite convention.

The fix here is to always return TEXT as `string`. Apps that want a `time.Time` should scan into `*time.Time` explicitly — that's the standard `database/sql` convention.

## Diff vs upstream

- `libsql.go`: removed the date-format parse loop in the `TYPE_TEXT` case of the row scanner; removed the now-unused `Outerloop:` label.
- `go.mod`: module path changed to `github.com/oathead/go-libsql-nodateparse`.

## Used by

- strength-trainer via a `replace` directive in `go.mod`.

## Rebasing on upstream

```bash
git remote add upstream https://github.com/tursodatabase/go-libsql.git
git fetch upstream
git rebase upstream/main
# resolve the libsql.go conflict by re-removing the parse loop
git push --force-with-lease origin main
```

## Upstream PR

A long-term fix is to land an opt-out flag upstream (e.g. `WithRawTextScan(true)`) so this fork is no longer needed.
