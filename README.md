# htmldb

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
    ├── jane.html
    └── joe.html          # <h1>joe</h1> + <dl> with email, blog, ...
```

A record page:

```xml
<main>
  <h1>joe</h1>
  <dl>
    <dt>blog</dt>
    <dd>adambien.blog</dd>
    <dt>email</dt>
    <dd>joe@airhacks.com</dd>
  </dl>
</main>
```

Every page is valid HTML5 *and* well-formed XML. Fields are sorted, and a `set` touches only that record's file — git history and diffs are per record. Table, key, and field names are filename-safe slugs (letters, digits, `_`, `-`; `index` is a reserved table/key name); field values are arbitrary text.

## Usage

```bash
htmldb users set joe email=joe@airhacks.com blog=adambien.blog   # creates users/joe.html
htmldb users set joe twitter=@AdamBien      # merges into the existing record
htmldb users get joe                        # all fields as field<TAB>value lines
htmldb users get joe email                  # → joe@airhacks.com
echo "Java Champion" | htmldb users set joe bio   # field value from stdin
htmldb users rm joe twitter                 # remove one field
htmldb users rm joe                         # remove the record
htmldb users keys                           # all keys, sorted
htmldb users list                           # key<TAB>field<TAB>value lines
htmldb tables                               # → config, users
```

Data goes to stdout, diagnostics and the version banner to stderr — pipe-friendly. Exit code 0 on success, 1 on missing keys, missing tables, or usage errors.

## Configuration

The database root defaults to the current directory. Override with the `db.dir` property ([zcfg](https://github.com/AdamBien/zcfg) precedence):

1. `~/.htmldb/app.properties`
2. `./app.properties`
3. `-Ddb.dir=<path>` system property

```properties
db.dir=/path/to/database
```

## Installation

Requires Java 25+.

```bash
chmod +x htmldb

# install system-wide (pick one)
sudo cp htmldb /usr/local/bin/
# or symlink for development
sudo ln -s $(pwd)/htmldb /usr/local/bin/htmldb
```
