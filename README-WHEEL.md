# wheel.sh Agent Guide
Practical instructions for using `./wheel.sh` to manage the WHEEL.db project database. Follow this before writing or changing code so you can reuse existing metadata instead of re-reading files.

## Setup & Environment
- Requirements: bash, `sqlite3` in PATH. Script exits with a fatal error if `sqlite3` is missing.
- Location: `wheel.sh` lives in the repo root; default database is `./WHEEL.db`.
- Executable bit: ensure `chmod +x wheel.sh` if it is not executable.
- Seeding: on first use, `./wheel.sh` looks for a sibling `WHEEL.sql`. If absent, it creates a fresh schema in `WHEEL.db` and dumps it back to `WHEEL.sql`.
- Fresh DB guidance: when both `WHEEL.db` and `WHEEL.sql` are missing, ask whether to scan the project and populate the DB; “Yes” is safest. If they decline, warn: `Warning: AGENT running without building database -- expect long waits.`
- Global options (apply to any command): `--database PATH` to target another DB (also adjusts the `.sql` dump path), `--force-params` / `--no-params` to control sqlite `.parameter` usage (auto-detected by default), `--verbose` for debug logging, `-h|--help` for the usage banner.
- Do not load `WHEEL.sql` into context; use `wheel.sh` to query instead.

## Data Model (schema)
- files: `id`, `relpath` (unique, NOT NULL), `description`
- defs: `id`, `file_id` (NOT NULL), `type` (NOT NULL), `signature`, `parameters`, `description`
- refs: `id`, `def_id` (NOT NULL), `reference_def_id` (NOT NULL)
- changes: `id`, `title` (NOT NULL), `context`, `status` (NOT NULL)
- change_files: `id`, `change_id` (NOT NULL), `file_id` (NOT NULL)
- change_defs: `id`, `change_id` (NOT NULL), `file_id` (NOT NULL), `def_id`, `description`
- todo: `id`, `change_id` (NOT NULL), `change_defs_id`, `change_files_id`, `description` (NOT NULL)
- A `.dump` to `WHEEL.sql` is refreshed after every mutating command.

## Command Map (top level)
- `query`, `search`, `insert`, `update`, `delete`, `describe`, `plan`, `raw`
- Shortcuts: `todo`, `changes`, `files`, `defs`
- Running `./wheel.sh` with no args defaults to `query` on `defs` with `--limit 50`.

## Query & Search
- `query` lets you pick the table (`--table` or first positional), select columns (`--columns` or positional), and filter.
- Multi-table joins: supply multiple `--table` entries plus `--merge colA,colB,...` (one join column per table). Specialized filters (`--type`, `--relpath`, etc.) and `--search` are single-table only.
- Filters: `--where SQL`, `--filter col=val` (LIKE with `%` auto-wrapped unless you provide `%/_`), `--id N`, `--order-by col`, `--limit N`, `--distinct`, `--count`, `--raw-sql` (echo SQL).
- Search terms: `--search TERM` ANDs over the default search columns for the chosen table (defs: relpath/signature/description/parameters/type; files: relpath/description; changes: title/context/status; change_files: file relpath; change_defs: description/relpath/def signature; todo: description; refs: def/ref signatures).
- Extra filters on defs: `--type`, `--relpath`, `--signature`, `--parameters`, `--description`, `--file-desc`, `--change`, `--refers-to DEF_ID`, `--referenced-by DEF_ID`.
- Example: list defs in a file with type filter  
  ```sh
  ./wheel.sh query defs --relpath "src/module.py" --type class --order-by "signature"
  ```
- Example: join defs + files via merge  
  ```sh
  ./wheel.sh query files.relpath defs.signature defs.description \
    --table files --table defs --merge defs.file_id,files.id
  ```
- Example: find defs touched by change 12  
  ```sh
  ./wheel.sh query defs --change 12 --order-by "files.relpath"
  ```
- Example: show refs where def 8 refers to others  
  ```sh
  ./wheel.sh query refs --refers-to 8
  ```
- `search` wraps `query --search` with a default `--limit 20` and can run across multiple tables (`--table/-t`). It prints headings when querying more than one table.

## Data Modification Commands
- `insert TABLE key=value ...` — inserts raw values.  
  ```sh
  ./wheel.sh insert files relpath=src/api.py description="HTTP handlers"
  ```
- `update TABLE --set col=val [--set col=val ...] [--id N | --where SQL]` — `--id`/`--where` are mutually exclusive.
- `delete TABLE [--id N | --where SQL]`
- `describe TABLE [--id N] [--where SQL] [--schema]` — with no flags shows first 20 rows; `--schema` prints pragma info.
- `plan CHANGE_ID` — prints the change row plus related change_files, change_defs, and todo rows.
- `raw [SQL]` — no SQL opens interactive `sqlite3`; with SQL, runs it once with headers.

## Shortcut Groups
### files
- `files list` — all files ordered by relpath.
- `files search TERM...` — AND search over relpath/description.
- `files add --relpath=PATH [--description=TEXT]`
- `files del [--file_id=N | --file=REL]`
- `files update [--file_id=N|--file=REL] [--relpath=PATH] [--description=TEXT]`

### defs
- `defs list [--file=REL | --file_id=N]`
- `defs search TERM... [--file=REL]`
- `defs add --file=REL|--file_id=N --type=TYPE [--signature=SIG] [--parameters=TEXT] [--description=TEXT]`
- `defs del [--def_id=N | --def=SIG [--file=REL|--file_id=N]]`
- `defs update [--def_id=N | --def=SIG [--file=REL|--file_id=N]] [--type=TYPE] [--signature=SIG] [--parameters=TEXT] [--description=TEXT] [--new_file=REL|--new_file_id=N]`

### changes
- `changes list`
- `changes add --title=TEXT --status=STATUS [--context=TEXT]`
- `changes update CHANGE_ID [--title=TEXT] [--status=STATUS] [--context=TEXT]`

### todo
- `todo list` — shows change links plus file/def context when available.
- `todo add --description=TEXT [--file=REL|--file_id=N] [--def=SIG|--def_id=N] [--change-id=N] [--change_def_id=N] [--change_file_id=N]`  
  - Resolves file/def ids automatically; infers change_defs/change_files links when possible; requires a change id (fails if it cannot determine one).  
  - When supplying change_def/file ids, it validates consistency with the change id and file/def you passed.
- `todo del TODO_ID`
- `todo search TERM... [--file=REL|--file_id=N] [--def=SIG|--def_id=N] [--change-id=N]`

### refs (no dedicated shortcut)
- Manage with generic commands; columns are `def_id`, `reference_def_id`.  
  ```sh
  ./wheel.sh insert refs def_id=12 reference_def_id=4
  ./wheel.sh query refs --referenced-by 4
  ```

## Example Workflows
- **Bootstrap and inspect**  
  ```sh
  ./wheel.sh --verbose            # ensure DB exists, see debug output
  ./wheel.sh describe files --schema
  ```
- **Add a file + definition and link to a change/todo**  
  ```sh
  change_id=$(./wheel.sh changes add --title="Implement parser" --status=planned | tail -n1)
  file_id=$(./wheel.sh files add --relpath=src/parser.py --description="Parser entrypoints" | tail -n1)
  def_id=$(./wheel.sh defs add --file_id="$file_id" --type=function --signature="parse(data)" --description="Parse payloads" | tail -n1)
  ./wheel.sh todo add --description="Cover parse()" --def_id="$def_id" --change-id="$change_id"
  ./wheel.sh plan "$change_id"
  ```
- **Search existing work before coding**  
  ```sh
  ./wheel.sh search tokenizer --table defs --table files
  ./wheel.sh query defs --signature "%parse%"
  ```
- **Audit latest change**  
  ```sh
  last=$(./wheel.sh changes list | tail -n1 | awk '{print $1}')
  ./wheel.sh plan "$last"
  ```
- **Join change_defs with files for a report**  
  ```sh
  ./wheel.sh query change_defs.id change_defs.change_id files.relpath defs.signature \
    --table change_defs --table files --table defs \
    --merge change_defs.file_id,files.id,defs.file_id \
    --order-by "change_defs.change_id, change_defs.id"
  ```

## Tips & Caveats
- Table names accept singular/plural aliases (def/defs, file/files, change/changes, etc.).
- LIKE filters auto-wrap values in `%…%` unless you include `%`/`_` yourself.
- Mutating commands refresh `WHEEL.sql`; non-mutating ones do not.
- If parameter binding errors occur, toggle with `--force-params` or `--no-params`.
- Prefer `wheel.sh` over raw file reads to discover existing functionality; update `files/defs/refs/changes/todo` consistently when adding code.
