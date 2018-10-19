# rdn - readable data notation

A data format that serves a similar purpose to JSON, but with the following features:

- Readable - less nesting, less noise
- Minimal and compactable
- Extendible with type tags
- More primitive types, plus multiline strings

## quick examples

Objects/maps/dictionaries are called tables here and are sets of key/vals

```
// Tables and nesting

table =
  child1 = 1
  child2 = 2
  child3 =
    child4 = 4

// Nested keys can be set from anywhere with a full path
table/child4/key = 'val'

// Inline can be done with semicolons and parens
inline_table = (x = 1; y = (z = 2))
```

Sequences are ordered collections of values

```
sequence = 
  # 1
  # 2
  # 3

inline_sequence = # 1 # 2 # 3

sequence_of_tables =
  # x = 1
    y = 2
    z = 3
  # a = 1
    b = 2

inline_sequence_of_tables = # (x = 1; y = 2) # (a = 1; b = 2)
```

Entries in a table can be added at any point using a path with slashes. This helps to reduce overly nested
configuration files:

```
beer =
  name = "super hefeweizen"
  category = "wheat beer"

beer/description = """
This is a very long, multi-line description.
I wouldn't want this block of text in a nested format above.
So instead I can set it down here.
"""

// Let's add some more details
beer/attributes =
  bitterness = 0.1
  shade = 0.22
  sweet_dry = 0.88
  rating = 89
```

Likewise, elements in a sequence can optionally be added at any point without nesting:

```
my_fav_numbers = # 1 # 2 # 3 # 5

my_fav_numbers/++ = 7
my_fav_numbers/++ = 11

// For appending elements, order in the document will naturally matter
```

You can tag values with `$tag`. This can be used as a type annotation:

```
my_uuid = $uuid "2a867683-f1b8-4b2e-be25-b7aa92e2e6c"
one_tenth = $ratio # 1 # 10
my_grade = $percent 91

fruit_names = $set # "apple" # "banana" # "grape"

my_customer = $customer (name = 'Bob Ross'; orders = 3)

// Tagging a table or sequence with linebreaks is easy:
my_order = $order
  product = 1234
  quantity = 2
profile_tags = $tags
  # "admin"
  # "editor"
  # "scientist"
  # "author"

users =
  # $user
    username = 'bobross'
    ip_address = $ip - 128 - 3 - 66 - 210
    login_info = $login_info
      login_count = 10
      last_login = 2018-10-19T23:11:11.244Z
```

Built-in primitives:

```
float1 = 1.0
float2 = -33e-10
int1 = 1
int2 = +99
int3 = -99

date_time = 1979-05-27T07:32:00-08:00
date = 1979-05-27
time = 09:09:11.1234565
time2 = 09:08:07

hex = 0xDEADBEEF
hex2 = 0xdeadbeef
oct = 0o755
binary = 0b11011010101

regex = /^\s+\w\n\s+\w\n\s+$/

bool1 = true
bool2 = false

single_quote_string = 'content'
double_quote_string = "content"

multiline_string = """
Hello
world
"""
```

Minification collapses a document into a single line without whitespace

```
// starting with:
x =
  y =
    # a = 1
      b = 2
    # c = 3
p =
  q = 3
x/z = 4
p/r = 5

// collapses into:

x=(y=(#a=1;b=2#c=3)z=4)p=(q=3;r=5)
```

Punctuation marks (`/`, `=`, `;`) can be escaped when used inside keys

## Acknowledgements

Insipred by: [edn](https://github.com/edn-format/edn) and [TOML](https://github.com/toml-lang/toml)
