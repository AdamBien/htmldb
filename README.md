# htmldb

**The storage format is also the UI.**

Zero-dependency, single-file Java 25 CLI that persists key-value data as semantic XHTML. The database is a folder of valid HTML pages: view it in any browser, parse it as XML, diff it with git. Atomic writes, no build tool, no libraries — install on PATH and use it like any shell command.

## Layout

Each table is a folder, each record is its own XHTML page holding its fields as a `<dl>` definition list. Per-table and root `index.html` pages link everything together — the database is a browsable website:

```
.
├── index.html            # root index: <nav> linking all tables
├── config/
│   ├── index.html        # <nav> linking all config records
│   └── app.html
└── users/
    ├── index.html
    ├── duke.html         # <h1>duke</h1> + <dl> with email, blog, ...
    └── jane.html
```

A record page:

```xml
<main>
  <h1>duke</h1>
  <dl>
    <dt>blog</dt>
    <dd>duke.blog</dd>
    <dt>email</dt>
    <dd>duke@airhacks.com</dd>
  </dl>
  <footer>
    <p>updated <time datetime="2026-08-03T15:31:07Z">2026-08-03T15:31:07Z</time></p>
  </footer>
</main>
```

In a table index each record is linked by its **first column's value**, falling back to the key when no columns are defined or the field is empty. A generated timestamp key identifies a record but says nothing about it, so `talks columns title,description` makes the index read as a list of titles.

Every page is valid HTML5 *and* well-formed XML. Fields follow the declared column order — in the page, in `get` and in `list` — with any fields outside the schema sorted after them; without columns a record stays alphabetical. A `set` touches only that record's file — git history and diffs are per record. `field=value` replaces a value, `field+=value` appends to it, joining with a newline so a field grows into a log; on a missing or empty field both are the same write. Table, key, and field names are filename-safe slugs (letters, digits, `_`, `-`; `index` is a reserved table/key name); field values are arbitrary text.

## Usage

```bash
htmldb users set duke email=duke@airhacks.com blog=duke.blog   # creates users/duke.html
htmldb users set duke twitter=@duke          # merges into the existing record
htmldb users set duke bio+="Java Champion"   # += appends instead of replacing

# fast entry: define the column order once, then set values positionally
htmldb users columns email,blog,twitter
htmldb users set jane "jane@airhacks.com,janes.blog,@jane"
htmldb users columns                        # print the column order

# note taking: add stores values positionally under a generated timestamp key
htmldb talks columns title,description
htmldb talks add "Java 25" "What's new in the source launcher"   # → talks/2026-08-04-142122.html

htmldb users get duke                       # all fields as field<TAB>value lines
htmldb users get duke email                 # → duke@airhacks.com
echo "Java Champion" | htmldb users set duke bio  # field value from stdin
cat notes.txt | htmldb notes set n1 body+         # trailing + appends the stdin value
htmldb users rm duke twitter                # remove one field
htmldb users rm duke                        # remove the record
htmldb users keys                           # all keys, sorted
htmldb users list                           # all records as an aligned table

# filter: case-insensitive substring, printed as key<TAB>label
htmldb users find airhacks                  # any field value, or the key
htmldb users find blog=duke                 # restrict the match to one field
htmldb users find blog=                     # records that have a blog at all
htmldb users find blog= twitter=airhacks    # several terms: all have to match
htmldb users find airhacks | cut -f1        # bare keys

htmldb users list blog=duke                 # same terms, as a readable table
htmldb notes list category=todo             # the everyday one

htmldb tables                               # → config, users
```

`find` and `list` take the same filter terms and differ only in output: `find` prints key<TAB>label to pipe, `list` renders the matching records as a table to read. A term is `<text>` (matching any field value or the key), `<field>=<text>` (matching that one field) or `<field>=` (records carrying the field at all); several terms narrow each other, so `category=todo priority=high` matches records satisfying both. A filter that matches nothing exits 1 — an unfiltered `list` of an empty table does not, since nothing was searched for.

Data goes to stdout, diagnostics and the version banner to stderr — pipe-friendly. `keys`, `get` and `find` emit tab separated values for scripting; `list` is the one command formatted for reading, padding its columns and cutting long values, so pipe `find` rather than `list`. Exit code 0 on success, 1 on missing keys, missing tables, no match, or usage errors.

## Dedicated CLIs

A symlink named after a table becomes a preconfigured tool (busybox-style) — the script derives its identity from the invoked file name and prepends it as the table:

```bash
ln -s /usr/local/bin/htmldb /usr/local/bin/talks
talks columns title,description
talks add "Java 25" "What's new in the source launcher"
```

Each symlink also reads its own global configuration (`~/.talks/app.properties`), so every dedicated CLI can point `db.dir` at its own database.

## Configuration

The database root defaults to the current directory. Override with the `db.dir` property ([zcfg](https://github.com/AdamBien/zcfg) precedence):

1. `~/.htmldb/app.properties`
2. `./app.properties`
3. `-Ddb.dir=<path>` system property

```properties
db.dir=/path/to/database
```

## Installation

Requires Java 25 or later.

```bash
curl -O https://raw.githubusercontent.com/AdamBien/htmldb/main/htmldb
chmod +x htmldb
./htmldb -help
```

Copy `htmldb` to a directory in your PATH for system-wide use:

```bash
sudo cp htmldb /usr/local/bin/
# or symlink for development
sudo ln -s $(pwd)/htmldb /usr/local/bin/htmldb
```
