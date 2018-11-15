# rdn - readable data notation

A data format that serves a similar purpose to JSON, but with the following features:

- Readable - less nesting, less noise
- Minimal and compactable
- Extendible with type tags
- More primitive types, plus multiline strings

## quick examples

Objects/maps/dictionaries are called tables here and are sets of key/vals

```
# Tables and nesting

table =
  child1 = 1
  child2 = 2
  child3 =
    child4 = 4

# Nested keys can be set from anywhere with a full path
table/child4/key = 'val'

# Inline tables/collections can be done with semicolons and parens
inline_table = (x = 1; y = (z = 2))
```

Parens and semicolons serve a similar purpose, separating chunks of syntax.

Collections a group of values without keys

```
sequence = 1 2 3

# asterisks can be used to start nested lines to denote collection elements
sequence_of_tables =
  * x = 1
    y = 2
    z = 3
  * a = 1
    b = 2

inline_version = (x = 1; y = 2; z = 3) (a = 1; b = 2)

more_nested_version = (x = 1; y = (z = 3)) (a = 1)

# Equivalent to:
more_readable =
  * x = 1
    y = (z = 3)
  * a = 1
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

# Let's add some more details
beer/attributes =
  bitterness = 0.1
  shade = 0.22
  sweet_dry = 0.88
  rating = 89
```

Likewise, elements in a sequence can optionally be added at any point without nesting:

```
my_fav_numbers = 1 2 3 5 7
my_fav_numbers/++ = 7
my_fav_numbers/++ = 11

# For appending elements, order in the document will naturally matter
```

You can tag values with `$tag`. This can be used as a type annotation on any value:

```
my_uuid = $uuid "2a867683-f1b8-4b2e-be25-b7aa92e2e6c"
one_tenth = $ratio | 1 | 10
my_grade = $percent 91

fruit_names = $set | "apple" | "banana" | "grape"

my_customer = $customer (name = 'Bob Ross'; orders = 3)

# Tagging a table or sequence with linebreaks is easy:
my_order = $order
  product = 1234
  quantity = 2
profile_tags = $tags
  | "admin"
  | "editor"
  | "scientist"
  | "author"

users =
  | $user
    username = 'bobross'
    ip_address = $ip | 128 | 3 | 66 | 210
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

All rdn structures can be collapsed into a single line without whitespace:

```
# starting with:
x =
  y =
    | a = 1
      b = 2
    | c = 3
p =
  q = 3
x/z = 4
p/r = 5

# collapses into:

x=(y=(|a=1;b=2|c=3)z=4)p=(q=3;r=5)
```

Punctuation marks (`/`, `=`, `;`, `$`, `++`, `#`, `|`, `*`, `'`, `"`), when needed inside keywords, can be escaped by
either prefixing it with a backslash or surrounding the keyword in quotes 

## Namespaces, references, expressions, functions, and processes

Given the same rdn file, you may interpret it differently. The most basic interpretation is to load everything as static data structures, similar to JSON or YAML.

Another interpretation is to treat the data as evaluatable references, procedures, and processes.

A **parser** takes an rdn file and loads it as static data structures. 

An **evaluator** takes an rdn file and:
* evaluates it as a code module
* pipes stdin into it
* sets any arguments
* imports and exports other modules
* runs it as a one-off or persistent process

The evaluator holds a "namespace" while it interprets a file. The namespace is an rdn document that the evaluator keeps at-hand, providing references to a set of data in your module.

Every rdn module has `stdin` and `args` in its namespace as top-level keys. It also has a set of built-in functions for working with files, strings, and math.

Each rdn module has some special top-level keywords:
* import: a table of modules to import based on file paths
* output: stdout that this module returns when run as a process
* exports: rdn data that this module exports when imported as a module

A module is evaluated from top to bottom. Every key is added into the module's namespace (overriding any existing keys). You can use the `*` punctuation mark to call a function. Any keys inside the expression can reference keys in the namespace

```
a = 1
a_ref = a
```

The above evaluates into `(a = 1; a_ref = 1)`. 

```
imports =
  my_util = "./my_util.rdn"

exports =
  item1 = 1
  item2 = 2
```

Imports all use system file paths pointing to other rdn modules. Paths prefixed with `"#/"` means the importer looks for modules in a global installation space using a manifest in the root of the project. Everything else is treated as relative paths to specific files. If pointing to a directory, then it imports the `main.rdn` file in that directory. The `.rdn` extension can be left off.

`exports` define the data that will be assigned in an import. If we were to import the above module and assign it to the `example` key, we can reference `example/item1` and `example/item2`.

Functions are always called with a simple prefix notation, similar to any lisp.

All functions take a single rdn document as input. It can be a scalar value, a list, or a table.

Highly nested expressions can use parens

```
a = 1
b = 2

# add takes any list of numbers as input
result = add a (mul b (div a b))
```

Functions can be defined using a special tagged table with keys for `params` and `output`:

```
say_hello = $func
  input = (type = string)
  output = concat "Hello " input "!!!!"
```

Loops can be defined using a tag:

```
# Sum of all numbers from 1 to 100
# This gets evaluated immediately
sum_to_100 = $loop
  state = (n = 1; sum = 1)
  end_when = eq n 100
  repeat =
    n = add 1 n
    sum = add n sum
  output = sum
```

The `sum_to_100` key in the above will be evaluated to a single integer.

Streams can be used to iterate lazily and infinitely:

```
stream_fibs = $stream
  state = (fib1 = 0; fib2 = 1)
  output = fib2
  repeat =
    fib1 = fib2
    fib2 = add fib1 fib2

# Request 10 values from generate_fibs and output the 10th one
fib_10 = yield_nth 10 stream_fibs
```

```
get_fib = $func
  input = (type = integer)
  output = yield_nth input stream_fibs 
```

You can do explicit currying by using the special `_` mark inside an expression

```
incr = add 1 _
eleven = incr 10

# The above is equivalent to a function like:
incr = $func
  input = (type = integer)
  output = add 1 input
```

The standard way to define multiple functional parameters is to use a table, meaning all parameters are named by keys

```
http_get = $func
  input =
    type = table
    keys =
      url = (type = string)
      params = (type = table)
  output = do_http_magic

response = http_get
  url = 'http://spacejam.com'
  params = (verbose = 1))

output = response/text
```

## Acknowledgements

Inspired by: [edn](https://github.com/edn-format/edn), [TOML](https://github.com/toml-lang/toml), and clojure
