# Note

**File Extension:** `.note`

**Encoding:** UTF-8

**License:** CC0

An easy-to-read, easy-to-write data format with JSON compatibility.

```text
{ Application & Server Configuration }
app = [
  name        = 'Cloud Mesh'
  version     => 2.4.0
  description =>> Lightweight service proxy
  debug       = false
]

server.host => 0.0.0.0
server.port = 8080        { Default HTTP port }

features = [ >caching >logging >monitoring ]

replicas = [
  ; host   => 10.0.0.2
    role   => primary
    active = true
  ; host   => 10.0.0.3
    role   => backup
    active = false
]

health_check.query = <sql>
  SELECT id, status FROM nodes
  WHERE active = true;
  <sql>
```

## 1. Why Note?

* **Easy to Read and Write**: Makes structured data clean and simple to type, with no comma or indentation errors.

* **JSON Compatible**: Shares the same data types and structures as JSON, making it easy to adopt where JSON is used.

* **Minimal Keystrokes**: Eliminates commas (`,`), reduces closing quotes (`>`, `>>`) and nested brackets (`;`, `.`), and requires no Shift key for core structural symbols (`[ ]`, `=`, `;`, `.`).

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

### ② Keys

Bare keys can contain any character except whitespace and reserved characters (`[ ] { } < > = ; . ' " , :`).

For keys containing whitespace, reserved characters, or matching numbers or reserved words (`true`, `false`, `nil`), quoted keys must be used.

```text
{ Bare keys }
app_version = '2.4.0'
_internal_id = 101
1st_tier = true
service-node-01 = 'active'

{ Quoted keys }
'v1.0.alpha' = 'released'
"user's choice" = 'dark'
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

### ④ Dotted Keys

Keys within nested objects can be represented in a single line using the dot (`.`).

Missing or non-object keys are automatically created as empty objects.

If already an object, keys are added or overwritten.

```text
{ Automatic creation and merging }
db.port = 5432                         { db=[ port=5432 ] }
db.host = 'localhost'                  { db=[ port=5432 host='localhost' ] }

{ Overwriting non-object values }
timeout = 5000
timeout.connect = 3000                 { timeout=[ connect=3000 ] }

{ Dotted keys with quoted keys }
project.'v1.0.alpha'.status = 'active' { project=[ 'v1.0.alpha'=[ status='active' ] ] }
app."user's choice".theme = 'dark'     { app=[ "user's choice"=[ theme='dark' ] ] }
```

### ⑤ Objects in Arrays

Within an array, a semicolon (`;`) starts an object without nested brackets, grouping subsequent `key=value` pairs until the next `;`, standalone value, or `]`.

```text
{ Array of objects }
employees = [
  ; id=101 name='Alice' department='Design'
  ; id=102 name='Bob'   department='Engineering'
]

{ Objects with other values }
tasks = [
  'Task List Start'
  ; id=1 title='Initial Design' status='done'
  '--- separator ---'
  ; id=2 title='Feedback Loop' status='pending'
]
```

### ⑥ Empty Structures

Empty arrays and objects are denoted as follows:

```text
arr = [ ]            { JSON: [] }
obj = [=]            {{ JSON: {} }}

obj_arr  = [;]       {{ JSON: [{}] }}
two_objs = [ ; ; ]   {{ JSON: [{}, {}] }}
```

### ⑦ Unquoted & Multi-line Strings

Unquoted and multi-line strings are raw strings.

* **Word String (`>`)**: Reads until whitespace or `[ ] { } < > ;`. Trimmed on both sides.
* **Line String (`>>`)**: Reads until the end of the line (`\n`) or a comment (`{ }`). Trimmed on both sides.
* **Multi-line String (`<tag> ... <tag>`)**: Leading whitespace up to the closing delimiter's indentation is stripped from each line, the newline after the opening delimiter and before the closing delimiter is ignored, and line endings are normalized to `\n`.
* **`<tag>`**: Can be empty (`<>`) or contain any character except whitespace and `< >` (e.g., `<sql>`, `<md>`, `<##>`).

```text
{ Word string (>) and Line string (>>) }
path   => C:\Users\admin\config.note
tags   = [ >web >api >cloud ]
notice =>> [Urgent] Maintenance at 02:00; please save your work.

{ Multi-line string (<tag>) }
message = <>
  Hello, World!
  Welcome to Note.
  <>

banner = <##>
  <> Production API Server <>
  Status: Online (v2.4.0)
  <##>

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
host => localhost
port = 8080          { Default HTTP port }

{
Block comment:
host => api.production-site.com
port = 443
}

{{ Comment containing braces: { "test": true } }}
```

## 3. Data Types

Supports the same data types as JSON.

* **String**: `' '`, `" "`, `>`, `>>`, `<tag>`
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
  "database": {
    "main": { "host": "10.0.0.1", "port": 5432 },
    "replicas": [
      { "host": "10.0.0.2", "port": 5432, "active": true },
      { "host": "10.0.0.3", "port": 5432, "active": false }
    ]
  },
  "services": {
    "auth": { "enabled": true, "provider": "oidc" },
    "storage": { "bucket": "assets", "region": "us-east-1" }
  },
  "features": ["caching", "logging", "monitoring"]
}
```

**system_config.note**

```text
project = [
  name = 'Cloud Mesh'
  environment => production
  version => 2.4.0
]

database.main = [ host=>10.0.0.1 port=5432 ]
database.replicas = [
  ; host=>10.0.0.2 port=5432 active=true
  ; host=>10.0.0.3 port=5432 active=false
]

services.auth = [ enabled=true provider=>oidc ]
services.storage = [ bucket=>assets region=>us-east-1 ]

features = [ >caching >logging >monitoring ]
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
; id=1 level=>info  msg='Server started'
; id=2 level=>warn  msg='High memory usage'
; id=3 level=>error msg='Database connection failed'
; id=4 level=>info  msg='User logout'
; id=5 level=>debug msg='Cache cleared'
```

### ③ Recursive Structures

**menu.json**

```json
{
  "title": "Documentation",
  "items": [
    {
      "title": "Getting Started",
      "path": "/intro"
    },
    {
      "title": "Guides",
      "items": [
        {
          "title": "Installation",
          "path": "/guides/install"
        },
        {
          "title": "Advanced",
          "items": [
            {
              "title": "Plugins",
              "path": "/guides/plugins"
            },
            {
              "title": "Security",
              "path": "/guides/security"
            }
          ]
        }
      ]
    }
  ]
}
```

**menu.note**

```text
title => Documentation
items = [
  ; title = 'Getting Started'
    path  => /intro
  ; title => Guides
    items = [
      ; title => Installation
        path  => /guides/install
      ; title => Advanced
        items = [
          ; title => Plugins
            path  => /guides/plugins
          ; title => Security
            path  => /guides/security
        ]
    ]
]
```

### ④ Document Containers

**docs.json**

```json
{
  "title": "Cloud Mesh",
  "version": "2.4.0",
  "overview": "# Overview\n**Cloud Mesh** is a lightweight service proxy.",
  "quickstart": "# Quick Start\n1. Install the CLI tool.\n2. Run the command:\n\n```bash\nmesh start --port 8080\n```"
}
```

**docs.note**

````text
title   = 'Cloud Mesh'
version => 2.4.0

overview = <md>
  # Overview
  **Cloud Mesh** is a lightweight service proxy.
  <md>

quickstart = <md>
  # Quick Start
  1. Install the CLI tool.
  2. Run the command:

  ```bash
  mesh start --port 8080
  ```
  <md>
````
