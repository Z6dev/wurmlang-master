# wurmerlang Interpreter
===========================

```
 .____   __      __   ____.                              
 |   _| /  \    /  \ |_   |                              
 |  |   \   \/\/   /   |  |                              
 |  |    \        /    |  |                              
 |  |_    \__/\  /    _|  |                              
 |____|        \/    |____|                              
                             .__                         
__  _  ____ _________  _____ |  | _____    ____    ____  
\ \/ \/ /  |  \_  __ \/     \|  | \__  \  /    \  / ___\ 
 \     /|  |  /|  | \/  Y Y  \  |__/ __ \|   |  \/ /_/  >
  \/\_/ |____/ |__|  |__|_|  /____(____  /___|  /\___  / 
                           \/          \/     \//_____/  
```

## Setup and Usage

Installation:
```
git clone https://github.com/Z6dev/wurmlang-master.git
go build
```
Usage:
```
.\lang [path to program relative to lang executable]
```

Building:
```
go build
```
Optionally, you can add a ``-o`` flag to rename the binary to something else

## Language spec

wurmerlang's syntax is a cross between Javascript and Python. Like Javascript, it uses `func` to define functions (named or anonymous), requires `{` and `}` for blocks, and doesn't need semicolons. But like Python, it uses keywords for `and` and `or` and `in`. Like both those languages, it distinguishes expressions and statements.

It's dynamically typed and garbage collected, with the usual data types: nil, bool, int, str, list, map, and func. There are also several builtin functions.

Calling this a "spec" is probably a bit grandiose, but it's the best you'll get.

### Programs

A wurmerlang program is simply zero or more statements. Statements don't actually have to be separated by newlines, only by whitespace. The following is a valid program (but you'd probably use newlines in the `if` block in real life):

```
s = "world"
print("Hello, " + s)
if s != "" { t = "The end"  print(t) }
// Hello, world
// The end
```

Between tokens, whitespace and comments (`//` through to the end of a line) are ignored.

### Types

wurmerlang has the following data types: nil, bool, int, str, list, map, and func. The int type is a signed 64-bit integer, strings are immutable arrays of bytes, lists are growable arrays (use the `append()` builtin), and maps are unordered hash tables. Trailing commas are allowed after the last element in a list or map:

Type      | Syntax                                    | Comments
--------- | ----------------------------------------- | --------
nil       | `nil`                                     |
bool      | `true false`                              |
int       | `0 42 1234 -5`                            | `-5` is actually `5` with unary `-`
str       | `"" "foo" "\"quotes\" and a\nline break"` | Escapes: `\" \\ \t \r \n`
list      | `[] [1, 2,] [1, 2, 3]`                    |
map       | `{} {"a": 1,} {"a": 1, "b": 2}`           |

### If statements

wurmerlang supports `if`, `else if`, and `else`. You must use `{ ... }` braces around the blocks:

```
a = 10
if a > 5 {
    print("large")
} else if a < 0 {
    print("negative")
} else {
    print("small")
}
// large
```

### While loops

While loops are very standard:

```
i = 3
while i > 0 {
    print(i)
    i = i - 1
}
// 3
// 2
// 1
```

wurmerlang does not have `break` or `continue`, but you can `return value` as one way of breaking out of a loop early.

### For loops

For loops are similar to Python's `for` loops and Go's `for range` loops. You can iterate through the (Unicode) characters in a string, elements in a list (the `range()` builtin returns a list), and keys in a map.

Note that iteration order of a map is undefined -- create a list of keys and `sort()` if you need that.

```
for c in "foo" {
    print(c)
}
// f
// o
// o

for x in [nil, 3, "z"] {
    print(x)
}
// nil
// 3
// z

for i in range(5) {
    print(i, i*i)
}
// 0 0
// 1 1
// 2 4
// 3 9
// 4 16

map = {"a": 1, "b": 2}
for k in map {
    print(k, map[k])
}
// a 1
// b 2
```

### Functions and return

You can define named or anonymous functions, including functions inside functions that reference outer variables (closures). Vararg functions are supported with `...` syntax like in Go.

```
func add(a, b) {
    return a + b
}
print(add(3, 4))
// 7

func make_adder(n) {
    func adder(x) {
        return x + n
    }
    return adder
}
add5 = make_adder(5)
print(add5(7))
// 12

// Anonymous function, equivalent to "func plus(nums...)"
plus = func(nums...) {
    sum = 0
    for n in nums {
        sum = sum + n
    }
    return sum
}
print(plus(1, 2, 3))
lst = [4, 5, 6]
print(plus(lst...))
// 6
// 15
```

A grammar note: you can't have a "bare return" -- it requires a return value. So if you don't want to return anything (functions always return at least nil anyway), just say `return nil`.

### Assignment

Assignment can assign to a name, a list element by index, or a map value by key. When assigning to a name (variable), it always assigns to the local function scope (like Python). You can't assign to an outer scope without using a mutable list or map (there's no `global` or `nonlocal` keyword).

To help with object-oriented programming, `obj.foo = bar` is syntactic sugar for `obj["foo"] = bar`. They're exactly equivalent.

```
i = 1
func nochange() {
    i = 2
    print(i)
}
print(i)
nochange()
print(i)
// 1
// 2
// 1

map = {"a": 1}
func change() {
    map.a = 2
    print(map.a)
}
print(map.a)
change()
print(map.a)
// 1
// 2
// 2

lst = [0, 1, 2]
lst[1] = "one"
print(lst)
// [0, "one", 2]

map = {"a": 1, "b": 2}
map["a"] = 3
map.c = 4
print(map)
// {"a": 3, "b": 2, "c": 4}
```

### Binary and unary operators

wurmerlang supports pretty standard binary and unary operators. Here they are with their precedence, from highest to lowest (operators of the same precedence evaluate left to right):

Operators      | Description
-------------- | -----------
`[]`           | Subscript
`-`            | Unary minus
`* / %`        | Multiplication
`+ -`          | Addition
`< <= > >= in` | Comparison
`== !=`        | Equality
`not`          | Logical not
`and`          | Logical and (short-circuit)
`or`           | Logical or (short-circuit)

Several of the operators are overloaded. Here are the types they can operate on:

Operator   | Types           | Action
---------- | --------------- | ------
`[]`       | `str[int]`      | fetch nth byte of str (0-based)
`[]`       | `list[int]`     | fetch nth element of list (0-based)
`[]`       | `map[str]`      | fetch map value by key str
`-`        | `int`           | negate int
`*`        | `int * int`     | multiply ints
`*`        | `str * int`     | repeat str n times
`*`        | `int * str`     | repeat str n times
`*`        | `list * int`    | repeat list n times, give new list
`*`        | `int * list`    | repeat list n times, give new list
`/`        | `int / int`     | divide ints, truncated
`%`        | `int % int`     | divide ints, give remainder
`+`        | `int + int`     | add ints
`+`        | `str + str`     | concatenate strs, give new string
`+`        | `list + list`   | concatenate lists, give new list
`+`        | `map + map`     | merge maps into new map, keys in right map win
`-`        | `int - int`     | subtract ints
`<`        | `int < int`     | true iff left < right
`<`        | `str < str`     | true iff left < right (lexicographical)
`<`        | `list < list`   | true iff left < right (lexicographical, recursive)
`<= > >=`  | same as `<`     | similar to `<`
`in`       | `str in str`    | true iff left is substr of right
`in`       | `any in list`   | true iff one of list elements == left
`in`       | `str in map`    | true iff key in map
`==`       | `any == any`    | deep equality (always false if different type)
`!=`       | `any != any`    | same as `not ==`
`not`      | `not bool`      | inverse of bool
`and`      | `bool and bool` | true iff both true, right not evaluated if left false
`or`       | `bool or bool`  | true iff either true, right not evaluated if left true

### Builtin functions

`append(list, values...)` appends the given elements to list, modifying the list in place. It returns nil, rather than returning the list, to reinforce the fact that it has side effects.

`args()` returns a list of the command-line arguments passed to the interpreter (after the wurmerlang source filename).

`char(int)` returns a one-character string with the given Unicode codepoint.

`exit([int])` exits the program immediately with given status code (0 if not given).

`find(haystack, needle)` returns the index of needle str in haystack str, or the index of needle element in haystack list. Returns -1 if not found.

`int(str_or_int)` converts decimal str to int (returns nil if invalid). If argument is an int already, return it directly.

`join(list, sep)` concatenates strs in list to form a single str, with the separator str between each element.

`len(iterable)` returns the length of a str (number of bytes), list (number of elements), or map (number of key/value pairs).

`lower(str)` returns a lowercased version of str.

`print(values...)` prints all values separated by a space and followed by a newline. The equivalent of `str(v)` is called on every value to convert it to a str.

`range(int)` returns a list of the numbers from 0 through int-1.

`read([filename])` reads standard input or the given file and returns the contents as a str.

`rune(str)` returns the Unicode codepoint for the given 1-character str.

`slice(str_or_list, start, end)` returns a subslice of the given str or list from index start through end-1. When slicing a list, the input list is not changed.

`sort(list[, func])` sorts the list in place using a stable sort, and returns nil. Elements in the list must be orderable with `<` (int, str, or list of those). If a key function is provided, it must take the element as an argument and return an orderable value to use as the sort key.

`split(str[, sep])` splits the str using given separator, and returns the parts (excluding the separator) as a list. If sep is not given or nil, it splits on whitespace.

`str(value)` returns the string representation of value: `nil` for nil, `true` or `false` for bool, decimal for int (eg: `1234`), the str itself for str (not quoted), the wurmerlang representation for list and map (eg: `[1, 2]` and `{"a": 1}` with keys sorted), and something like `<func name>` for func.

`type(value)` returns a str denoting the type of value: `nil`, `bool`, `int`, `str`, `list`, `map`, or `func`.

`upper(str)` returns an uppercased version of str.
