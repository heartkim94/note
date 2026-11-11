# Note

**File Extension:** `.note`

**Encoding:** UTF-8

**License:** CC0

An easy-to-read, easy-to-write data format with JSON compatibility.

```text
{ Application & Server Configuration }
app = [
  name        = 'Cloud Mesh'
  environment = production
  version     = .2.4.0
  description => Lightweight service proxy
  debug       = false
]

server.host = .0.0.0.0
server.port = 8080        { Default HTTP port }

features = [ caching logging monitoring ]

replicas = [
  ; host=.10.0.0.2 port=5432 active=true
  ; host=.10.0.0.3 port=5432 active=false
]

routes = [
  [ /api    ; auth mode=jwt  ; proxy to=backend ]
  [ /static ; cache ttl=3600 ; file_server root=/var/www ]
]

fallback = [ respond ; status=503 'Service temporarily unavailable' ]

health_check.query = <sql>
  SELECT id, status FROM nodes
  WHERE active = true;
  <sql>
```

## 1. Why Note?

* **Easy to Read and Write**: Makes structured data clean and simple to type, with no comma or indentation errors.

* **JSON Compatible**: Shares the same data types and structures as JSON, making it easy to adopt where JSON is used.

* **Minimal Keystrokes**: Eliminates commas (`,`), reduces quotes (bare strings, `.`, `>`) and nested brackets (`;`, `.`), and requires no Shift key for core structural symbols (`[ ]`, `=`, `;`, `.`).

* **Comments & Multi-line Strings**: Supports inline and block comments (`{ }`) and raw multi-line strings (`<word>`).

## 2. Syntax

### 2.1 Data Structures

Both arrays and objects use `[ ]`, distinguished by whether they contain an assignment symbol (`=`).

Items are separated by spaces and newlines.

Key-value pairs and standalone values cannot be mixed within the same brackets.

```text
{ Object }
user = [ name='John Doe' occupation='Developer' ]

{ Array }
skills = [ 'javascript' 'python' 'rust' ]
```

### 2.2 Empty Structures

Empty arrays and objects are denoted as follows:

```text
arr = [ ]   { JSON: [] }
obj = [=]   {{ JSON: {} }}
```

### 2.3 Implicit Root

The root `[ ]` is omitted and implicitly treated as `[ ... ]`. The root type is determined by the format of the internal content (an empty file evaluates to an empty array `[]`).

```text
{ Implicit object root }
name = 'Project A'
version = 1.0

{ Implicit array root }
'first item'
'second item'
```

### 2.4 Keys

Bare, dot (`.`), and quoted (`' '`, `" "`) strings can be used as keys.

Bare strings can contain any character except whitespace and reserved characters (`[ ] { } < > = ; . ' " , :`), and cannot start with a digit (or `-` followed by a digit) or match reserved words (`true`, `false`, `nil`).

Dot strings (`.`) start with `.` and read a single word as a raw string until following whitespace or `[ ] = ;`.

Quoted strings (`' '`, `" "`) can contain whitespace, reserved characters, and reserved words.

```text
{ Bare keys }
app_version = '2.4.0'
_internal_id = 101
service-node-01 = 'active'

{ Dot keys (.) }
.1st_tier = true
.v1.0.alpha = 'released'  { 'v1.0.alpha' = 'released' }
..gitignore = true        { '.gitignore' = true }

{ Quoted keys }
"user's choice" = 'dark'
'env=prod' = true
```

### 2.5 Key Paths

Keys within nested objects can be represented in a single line by joining bare or quoted keys directly with a dot (`.`) without whitespace.

Missing or non-object keys are automatically created as empty objects.

If already an object, keys are added or overwritten.

```text
{ Automatic creation and merging }
db.port = 5432                       { db=[ port=5432 ] }
db.host = localhost                  { db=[ port=5432 host=localhost ] }

{ Overwriting non-object values }
timeout = 5000
timeout.connect = 3000               { timeout=[ connect=3000 ] }

{ Key paths with quoted keys }
project.'v1.0.alpha'.status = active { project=[ 'v1.0.alpha'=[ status=active ] ] }
app."user's choice".theme = dark     { app=[ "user's choice"=[ theme=dark ] ] }
```

### 2.6 Objects in Arrays

Within an array, a semicolon (`;`) starts an object without nested brackets, grouping subsequent `key=value` pairs until the next `;`, `>`, standalone value, or `]`.

A `;` object can start with a name key, wrapping subsequent `key=value` pairs under that key.

A `;` object must contain at least a name key or a `key=value` pair.

```text
{ Array of objects }
employees = [
  ; id=101 name=Alice department=Design
  ; id=102 name=Bob   department=Engineering
]

{ Named objects in arrays }
steps = [
  ; connect host=localhost port=5432  { [ connect=[ host=localhost port=5432 ] ] }
  ; auth.login user=admin             { [ auth=[ login=[ user=admin ] ] ] }
  ; disconnect                        { [ disconnect=[=] ] }
]

{ Objects with standalone values }
event = [ notify ; level=warn sound=true 'Disk space is low' ]

stages = [
  [ build
    ; checkout
    ; run cmd='npm ci'
    ; run cmd='npm test'
  ]
]
```

### 2.7 Line & Multi-line Strings

Line (`>`) and multi-line (`<word>`) strings are raw strings used for values.

* **Line String (`>`)**: Used after `=`. Reads until the end of the line (`\n`), trimmed on both sides.
* **Multi-line String (`<word> ... <word>`)**: Leading whitespace up to the closing delimiter's indentation is stripped from each line, the newline after the opening delimiter and before the closing delimiter is ignored, and line endings are normalized to `\n`.
* **`<word>`**: Can be empty (`<>`) or contain any character except whitespace and `< >` (e.g., `<sql>`, `<md>`, `<##>`).

```text
{ Line string (>) }
notice => Maintenance starts at 02:00. Please save your work.
format => [{time}] {level}: {message} (user={user}; ip={ip})

{ Multi-line string (<word>) }
quote = <>He said, "It's working!"<>

message = <>
  Hello, World!
  Welcome to Note.
  <>

query = <sql>
  SELECT id, name FROM users
  WHERE status = 'active'
    AND role = 'admin';
  <sql>
```

### 2.8 Flow Text

Within an array, `>` (when not following `=`) starts flow text mode, reading raw text until the closing `]` and splitting it into array elements.

* **Trimming**: Whitespace immediately after `>`, before the closing `]`, and around each newline is trimmed; other inline spaces are preserved, and empty text segments are omitted.
* **Newlines**: The newline immediately after `>` and before the closing `]` is ignored. Each internal newline is inserted as a standalone `"\n"` element (normalized to `\n`).
* **Nested `[ ]`**: Parsed as normal Note structures and inserted between text elements; empty arrays (`[ ]`) are omitted.

```text
{ Single-line flow text }
heading = [ h1 > Configuration Guide ]
{ [ 'h1' 'Configuration Guide' ] }

{ Multi-line flow text with inline structures }
paragraph = [ p
  > When the server starts, it looks for a [code .config.note]
  file in the working directory.
]
{ [ 'p' 'When the server starts, it looks for a ' [ 'code' 'config.note' ] '\n' 'file in the working directory.' ] }

{ Bracket text with ['...'] }
message = [ > System ['[notice]']: Server will restart soon. ]
{ [ 'System ' [ '[notice]' ] ': Server will restart soon.' ] }

{ Line split and leading spaces }
lines = [
  > First part [{ comment across
  lines }]second part
  []    indented line
]
{ [ 'First part ' 'second part' '\n' '    indented line' ] }
```

### 2.9 Comments

Inline and block comments use `{ }`, allowing variable-length braces (e.g., `{{ }}`, `{{{ }}}`) to prevent delimiter collision.

```text
{ Inline and single-line comments }
host = localhost
port = 8080          { Default HTTP port }

{
Block comment:
host = .api.production-site.com
port = 443
}

{{ Comment containing braces: { "test": true } }}
```

## 3. Data Types

Supports the same data types as JSON.

* **String**: Bare, `.`, `' '`, `" "`, `>`, `<word>`
  * Quoted strings (`' '`, `" "`) support escape sequences: `\n`, `\t`, `\r`, `\b`, `\f`, `\'`, `\"`, `\\`, and `\u{x...}` (e.g., `\u{41}`, `\u{1F600}`).

* **Number**: `1024`, `100_000`, `3.14`, `-5`, `1.2e-3`

* **Boolean**: `true`, `false`

* **Null**: `nil`

* **Array**: `[ val1 val2 ]`, `[ ]`

* **Object**: `[ key=val ]`, `[=]`

## 4. Use Cases

### 4.1 Configuration Files

**system_config.json**

```json
{
  "project": {
    "name": "Cloud Mesh",
    "environment": "production",
    "version": "2.4.0"
  },
  "services": {
    "auth": { "enabled": true, "provider": "oidc" },
    "storage": { "bucket": "assets", "region": "us-east-1" }
  },
  "features": ["caching", "logging", "monitoring"],
  "database": {
    "main": { "host": "10.0.0.1", "port": 5432 },
    "replicas": [
      { "host": "10.0.0.2", "port": 5432, "active": true },
      { "host": "10.0.0.3", "port": 5432, "active": false }
    ]
  },
  "middlewares": [
    { "cors": {} },
    { "rate_limit": { "max": 100, "window": 60 } },
    { "cache": { "ttl": 300 } }
  ]
}
```

**system_config.note**

```text
project = [
  name = 'Cloud Mesh'
  environment = production
  version = .2.4.0
]

services.auth = [ enabled=true provider=oidc ]
services.storage = [ bucket=assets region=us-east-1 ]

features = [ caching logging monitoring ]

database.main = [ host=.10.0.0.1 port=5432 ]
database.replicas = [
  ; host=.10.0.0.2 port=5432 active=true
  ; host=.10.0.0.3 port=5432 active=false
]

middlewares = [
  ; cors
  ; rate_limit max=100 window=60
  ; cache      ttl=300
]
```

### 4.2 Log Data

**logs.jsonl**

```json
{"time": "2026-11-08T16:00:00", "level": "info", "msg": "Server started"}
{"time": "2026-11-08T16:00:05", "level": "warn", "msg": "High memory usage"}
{"time": "2026-11-08T16:00:10", "level": "error", "msg": "Database connection failed"}
{"time": "2026-11-08T16:00:15", "level": "info", "msg": "User logout"}
{"time": "2026-11-08T16:00:20", "level": "debug", "msg": "Cache cleared"}
```

**logs.note**

```text
; time=.2026-11-08T16:00:00 level=info  msg='Server started'
; time=.2026-11-08T16:00:05 level=warn  msg='High memory usage'
; time=.2026-11-08T16:00:10 level=error msg='Database connection failed'
; time=.2026-11-08T16:00:15 level=info  msg='User logout'
; time=.2026-11-08T16:00:20 level=debug msg='Cache cleared'
```

### 4.3 Menus

**menu.json**

```json
{
  "title": "Main Menu",
  "items": [
    {
      "label": "File",
      "items": [
        { "label": "New File", "command": "file.new", "shortcut": "Ctrl+N" },
        { "label": "Save", "command": "file.save", "shortcut": "Ctrl+S" },
        {
          "label": "Export",
          "items": [
            { "label": "PDF", "command": "export.pdf" },
            { "label": "HTML", "command": "export.html" }
          ]
        }
      ]
    },
    {
      "label": "Help",
      "items": [
        { "label": "Documentation", "url": "/docs" },
        { "label": "About", "command": "app.about" }
      ]
    }
  ]
}
```

**menu.note**

```text
title = 'Main Menu'
items = [
  ; label = File
    items = [
      ; label='New File' command=.file.new  shortcut=Ctrl+N
      ; label=Save       command=.file.save shortcut=Ctrl+S
      ; label = Export
        items = [
          ; label=PDF  command=.export.pdf
          ; label=HTML command=.export.html
        ]
    ]
  ; label = Help
    items = [
      ; label=Documentation url=/docs
      ; label=About         command=.app.about
    ]
]
```

### 4.4 Documents

**article.json**

```json
[
  "article",
  ["h1", "Configuration Guide"],
  [
    "p",
    "When the server starts, it automatically looks for a ",
    ["code", "config.note"],
    "\n",
    "file in the working directory and falls back to built-in defaults if none is found."
  ],
  ["h2", "Environment Overrides"],
  [
    "p",
    "Any setting can be overridden at runtime using environment variables. For example, setting",
    "\n",
    ["code", "PORT=9000"],
    " overrides the default port ",
    ["b", "8080"],
    " without modifying the file."
  ],
  [
    "p",
    "Log levels accept ",
    ["code", "debug"],
    " or ",
    ["code", "info"],
    ", and relative file",
    "\n",
    "paths are resolved from the directory containing the configuration file."
  ],
  [
    "blockquote",
    [
      "p",
      "Sensitive credentials such as database passwords should ",
      ["em", "never"],
      "\n",
      "be committed to version control. Refer to the",
      "\n",
      ["a", { "href": "https://example.com/security", "target": "_blank" }, "security handbook"],
      " for secret management."
    ]
  ]
]
```

**article.note**

```text
article

[ h1 > Configuration Guide ]

[ p
  > When the server starts, it automatically looks for a [code .config.note]
  file in the working directory and falls back to built-in defaults if none is found.
]

[ h2 > Environment Overrides ]

[ p
  > Any setting can be overridden at runtime using environment variables. For example, setting
  [code 'PORT=9000'] overrides the default port [b .8080] without modifying the file.
]

[ p
  > Log levels accept [code debug] or [code info], and relative file
  paths are resolved from the directory containing the configuration file.
]

[ blockquote
  [ p
    > Sensitive credentials such as database passwords should [em never]
    be committed to version control. Refer to the
    [a ;href=.https://example.com/security target=_blank > security handbook] for secret management.
  ]
]
```
