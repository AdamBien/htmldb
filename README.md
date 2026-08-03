# htmldb

Zero-dependency, single-file Java 25 CLI that persists key-value data as semantic XHTML. The database is a folder of valid HTML pages: view it in any browser, parse it as XML, diff it with git. Atomic writes, no build tool, no libraries — install on PATH and use it like any shell command.

## Layout

Each table is a folder with an `index.html` holding its entries as a `<dl>` definition list. A root `index.html` links all tables:

```
.
├── index.html          # root index: <nav> linking all tables
├── config/index.html   # <dl> with all config entries
└── users/index.html    # <dl> with all user entries
```

Every page is valid HTML5 *and* well-formed XML. Entries are sorted by key, so writes produce minimal, stable git diffs.

## Usage

```bash
htmldb users set joe joe@airhacks.com   # creates users/index.html on demand
htmldb users get joe                    # → joe@airhacks.com
echo "8080" | htmldb config set port    # value from stdin
htmldb users keys                       # all keys, sorted
htmldb config list                      # key<TAB>value lines
htmldb users rm joe
htmldb tables                           # → config, users
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
