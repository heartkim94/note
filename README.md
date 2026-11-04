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

middlewares = [
  ; cors
  ; rate_limit max=100 window=60
]

health_check.query = <sql>
  SELECT id, status FROM nodes
  WHERE active = true;
  <sql>
```

## 1. Why Note?

* **Easy to Read and Write**: Makes structured data clean and simple to type, with no comma or indentation errors.

* **JSON Compatible**: Shares the same data types and structures as JSON, making it easy to adopt where JSON is used.

* **Minimal Keystrokes**: Eliminates commas (`,`), reduces quotes (bare strings, `.`, `>`) and nested brackets (`;`, `.`), and requires no Shift key for core structural symbols (`[ ]`, `=`, `;`, `.`).

* **Comments & Multi-line Strings**: Supports inline and block comments (`{ }`) and raw multi-line strings (`<tag>`).

## 2. Syntax

Note uses spaces and newlines as delimiters instead of commas (`,`).

### ① Data Structures

Both arrays and objects use `[ ]`, distinguished by whether they contain an assignment symbol (`=`).

Key-value pairs and standalone values cannot be mixed within the same brackets.

```text
{ Object }
user = [ name='John Doe' occupation='Developer' ]

{ Array }
skills = [ 'javascript' 'python' 'rust' ]
```

### ② Empty Structures

Empty arrays and objects are denoted as follows:

```text
arr = [ ]   { JSON: [] }
obj = [=]   {{ JSON: {} }}
```

### ③ Implicit Root

The root `[ ]` is omitted and implicitly treated as `[ ... ]`. The root type is determined by the format of the internal content (an empty file evaluates to an empty array `[]`).

```text
{ Implicit object root }
name = 'Project A'
version = 1.0

{ Implicit array root }
'first item'
'second item'
```

### ④ Keys

Strings can be used for keys.

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

### ⑤ Key Paths

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

### ⑥ Objects in Arrays

Within an array, a semicolon (`;`) starts an object without nested brackets, grouping subsequent `key=value` pairs until the next `;`, standalone value, or `]`.

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

{ Objects with other values }
event = [ notify ; level=warn sound=true 'Disk space is low' ]
  { [ notify [ level=warn sound=true ] 'Disk space is low' ] }
```

### ⑦ Line & Multi-line Strings

Line (`>`) and multi-line (`<tag>`) strings are raw strings.

* **Line String (`>`)**: Reads until the end of the line (`\n`). Trimmed on both sides.
* **Multi-line String (`<tag> ... <tag>`)**: Leading whitespace up to the closing delimiter's indentation is stripped from each line, the newline after the opening delimiter and before the closing delimiter is ignored, and line endings are normalized to `\n`.
* **`<tag>`**: Can be empty (`<>`) or contain any character except whitespace and `< >` (e.g., `<sql>`, `<md>`, `<##>`).

```text
{ Line string (>) }
notice => Maintenance starts at 02:00. Please save your work.
format => [{time}] {level}: {message} (user={user}; ip={ip})

{ Multi-line string (<tag>) }
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

### ⑧ Comments

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

* **String**: Bare, `.`, `' '`, `" "`, `>`, `<tag>`
  * Quoted strings (`' '`, `" "`) support escape sequences: `\n`, `\t`, `\r`, `\b`, `\f`, `\'`, `\"`, `\\`, and `\u{x...}` (e.g., `\u{41}`, `\u{1F600}`).

* **Number**: `1024`, `100_000`, `3.14`, `-5`, `1.2e-3`

* **Boolean**: `true`, `false`

* **Null**: `nil`

* **Array**: `[ val1 val2 ]`, `[ ]`

* **Object**: `[ key=val ]`, `[=]`

## 4. Use Cases

### ① Configuration Files

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

### ② Log Data

**logs.jsonl**

```json
{"id": 1, "level": "info", "msg": "Server started"}
{"id": 2, "level": "warn", "msg": "High memory usage"}
{"id": 3, "level": "error", "msg": "Database connection failed"}
{"id": 4, "level": "info", "msg": "User logout"}
{"id": 5, "level": "debug", "msg": "Cache cleared"}
```

**logs.note**

```text
; id=1 level=info  msg='Server started'
; id=2 level=warn  msg='High memory usage'
; id=3 level=error msg='Database connection failed'
; id=4 level=info  msg='User logout'
; id=5 level=debug msg='Cache cleared'
```

### ③ Menus

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

### ④ Documents

**notebook.ipynb**

```json
{
  "nbformat": 4,
  "nbformat_minor": 5,
  "metadata": {
    "kernel": "python3"
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {},
      "source": "# Data Analysis\nInstall the required package:\n\n```bash\npip install pandas\n```"
    },
    {
      "cell_type": "code",
      "execution_count": 1,
      "metadata": {},
      "outputs": [],
      "source": "import pandas as pd\n\ndf = pd.read_csv(\"users.csv\")\nprint(df.head())"
    }
  ]
}
```

**notebook.note**

````text
nbformat = 4
nbformat_minor = 5
metadata.kernel = python3

cells = [
  ; cell_type = markdown
    metadata  = [=]
    source    = <md>
      # Data Analysis
      Install the required package:

      ```bash
      pip install pandas
      ```
      <md>

  ; cell_type       = code
    execution_count = 1
    metadata        = [=]
    outputs         = [ ]
    source          = <py>
      import pandas as pd

      df = pd.read_csv("users.csv")
      print(df.head())
      <py>
]
````

### ⑤ Rich Text

**article.json**

```json
{
  "title": "Cloud Mesh v2.4 Release Notes",
  "author": "dev-team",
  "tags": ["release", "proxy", "performance"],
  "body": [
    ["h2", "1. Highlights"],
    [
      "p",
      "This release significantly reduces memory overhead in the routing engine. ",
      "Target version ",
      ["badge", { "color": "blue" }, "2.4.0"],
      " will be rolled out across ",
      ["b", 12],
      " nodes with zero downtime."
    ],
    [
      "callout",
      { "type": "warning", "title": "Migration Notice" },
      [
        "p",
        "Legacy config files (",
        ["code", ".mesh.conf"],
        ") are ",
        ["del", "deprecated"],
        " and will be migrated automatically on startup."
      ],
      [
        "p",
        "Please refer to the ",
        ["a", { "href": "https://example.com/guide", "target": "_blank" }, "migration guide"],
        " for details."
      ]
    ]
  ]
}
```

**article.note**

```text
title  => Cloud Mesh v2.4 Release Notes
author = dev-team
tags   = [ release proxy performance ]

body = [
  [ h2 '1. Highlights' ]

  [ p
    'This release significantly reduces memory overhead in the routing engine. '
    'Target version ' [badge ;color=blue .2.4.0] ' will be rolled out across ' [b 12] ' nodes with zero downtime.'
  ]

  [ callout ; type=warning title='Migration Notice'
    [ p 'Legacy config files (' [code '.mesh.conf'] ') are ' [del 'deprecated'] ' and will be migrated automatically on startup.' ]
    [ p 'Please refer to the ' [a ;href=.https://example.com/guide target=_blank 'migration guide'] ' for details.' ]
  ]
]
```
