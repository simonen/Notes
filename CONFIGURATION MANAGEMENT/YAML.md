---
tags:
  - ansible
  - yaml
---
[YAML Ain't Markup Language](https://en.wikipedia.org/wiki/YAML)

YAML data is structured in a hierarchy using **indentation** to indicate nesting, rather than using brackets or braces. YAML is **whitespace-sensitive**; each level of indentation represents nesting or a new hierarchy level.

YAML Structure:

1. Key-value pairs: Basic data in YAML

``` yaml
name: "Ansible Configuration"
version: 2.9
```

2. Indentation is achieved with spaces, not tabs. Each level is intended with 2 spaces (or 4)

``` yaml
project:
	name: "Web App"
	version: 1.0
```

3. List (arrays) items in YAML start with a dash ( - ) and are aligned at the same indentation level

``` yaml
packages:
 - nginx
 - python3
 - git
```

``` python
packages = [nginx, pyhon3, git]
```

4. Dictionaries:

``` yaml
alice:
  name: Alice
  email: alice@example.com
vince: 
  name: Vince
  email: vince@ohio.cc
```

```
alice = {name: "Alice", email: "alice@example.com"}
```

5. Comments: Start with `#` and can e used anywhere outside of the data

``` yaml
version: 1.0 # Inline comment
```

6. Multi-line Strings: 

	Literal Block (`|`): Preserves newlines within a multi-line string.

``` yaml
env: |-
  DB_HOST="database.host.com"
  DB_DATABASE="testdb"
```

Folded block (`>`): Ignores new lines. Everything is treated as a singe line.

``` yaml
motd: >
	Welcome to your new system 
	where everything from this 
	message should be on the same line.
```

List of dictionaries

``` yaml
items:
    - part_no:   A4786
      descrip:   Water Bucket (Filled)
      price:     1.47
      quantity:  4

    - part_no:   E1628
      descrip:   High Heeled "Ruby" Slippers
      size:      8
      price:     133.7
      quantity:  1
```

``` python
items = [
	{
		part_no: A4786, 
		descript: "Water Bucket (Filled)", 
		price: 1.47, 
		quantity: 4
	},
	{
		part_no: E1628, 
		descript: High Heeled Sleepers, 
		price: 133, 
		quantity: 1
	},
]
```



#### YAML Data Types

String

Strings can be quoted with `''` or `""` or unquoted. Quoted strings are useful when the text includes special characters or YAML keywords

``` yaml
greeting: "Hello, Wordl!"
```

Number

Numbers can be integers or floats. Do not require quotes

``` yaml
count: 5
pi: 3.14159
```

Boolean

Lowercase `true` or `false`

``` yaml
is_active: true
```

Null

``` yaml
middle_name: null
```

#### YAML Examples

Simple Key-Value pairs

```yaml
name: "My App"
version: 1.2
description: "A simple description"
```

Nested dictionaries

``` yaml
app:
	name: "Website"
	version: 1.0
	owner:
		name: "Alice"
		email: "alice@ohio.cc"
```

List of dictionaries

``` yaml
users:
	- name: "Alice"
	  role: "admin"
	- name: "Bob"
	  role: "user"
```

``` python
users = [
	{ 
	name: "Alice",
	  role: "admin"},
	{ name: "Bob",
	  role: "user",
	},
]
```

