JohnPavelLang Specification
======================

Lexical Syntax
--------------

Every byte of a valid JPL program has integer value 10 or 32-126
inclusive. Note that this means that space (32) and newline (10) are
the only valid whitespace characters (tabs are illegal, and only Unix
newlines are allowed).

An integer literal is a sequence of one or more digits. An integer
literal that does not map to a 64-bit two's complement value
(e.g. 9999999999999999999999) is a compile-time error; in other words
the minimum integer value is `-2^63` and the maximum integer value is
`2^63 - 1`.

A float literal is a sequence of digits, a dot, and another sequence
of digits; one of the two sequences must be non-empty. The dot is
required. Scientific notation is not supported. Literals mapping to
infinity are not supported. Do not write your own code to convert
float syntax to float values. It is much harder and subtler than you
probably think! Use the C library function `strtod` or its binding in
your language of choice (ex. Python's `float`) to perform the
conversion. If this conversion is error-free, then the literal is
legal, otherwise the JPL compiler must signal a compile-time error.

Strings are a double quote, any sequence of legal characters except
double quote and newline, and then another double quote. Character
escapes like `\n` aren't supported (you're not going to need them).
Multi-line string literals are not supported.

Variables are a letter (upper case A-Z or lower case a-z) followed by
any number of letters or digits, or underscores, *except* when
the sequence of letters and digits is a keyword. By convention,
variables that begin with double underscores are reserved for compiler
intermediates and should not be written in source code.

The keywords are: `array`, `assert`, `bool`, `else`, `false`, `float`,
`fn`, `if`, `image`, `int`, `let`, `print`, `read`, `return`, `show`,
`struct`, `sum`, `then`, `time`, `to`, `true`, `void`, `write`.

Whitespace is allowed between any two tokens and consists of any
sequence of spaces, line comments, block comments, and newline
escapes. Line comments are a `//`, followed by any sequence of
non-newline characters. Block comments are a `/*`, followed by
any sequence of characters not including `*/`, followed by `*/`.
Newline escapes are a backslash followed immediately by a newline.

Example program with newline escapes
(`let x = 4 + 4` could be on one line, without escapes):

```
let x = \
4 \
+ 4
show x
```

A single-line comment nevers contain a newline character, so the
newline character at the end of a single-line comment generates a
distinct newline token. Conversely, a multi-line comment may contain
newline characters, and those contained newline characters do not
generate distinct newline tokens.

Multiple consecutive newline tokens must be squashed into one.

Syntax
------

There are four syntax classes: types, expressions, statements, and
commands. A program is a sequence of commands, each of which must
be terminated by a newline.

There are also auxiliary syntax classes for lvalues, arguments, and
bindings.

In the grammar below, semicolons represent newline tokens.

Ellipses represent repetition with a separator. For example, `( <expr>
, ... )`means any sequence of left parenthesis, expression, comma,
expression, comma, and so on until a final expression, and then a
right parenthesis. Repetition always allows zero repetitions. Each
comma must be followed by a repetend: a final trailing comma is not
allowed.

### Type Syntax

The base types are Booleans, 64-bit signed integers, and 64-bit
(double precision) floats, of which you can form arrays and structs:

```
type : int
     | bool
     | float
     | <type> [ , ... ]
     | <variable>
     | void
```

The `<variable>` branch refers to type variables introduced with
the `struct` command; see below.

The array syntax allows any number of commas between the square
brackets; for example, `int[,,,]` means a four-dimensional array of
integers. The dimensionality of an array is called its "rank". Note
the distinction between `int[,]` (a two-dimensional or rank-2 array)
and `int[][]` (a one-dimensional or rank-1 array of one-dimensional or
rank-1 arrays). The difference is that the first is guaranteed to be
rectangular, while the second is not.

JPL does not have any implicit conversions between types, not even from
ints to floats.


### Expressions

JPL has constructors for each basic type:

```
expr : <integer>
     | <float>
     | true
     | false
     | void
```

Variables always have a static type from the environment:

```
expr : <variable>
```

Struct and array constructors use curly braces and square braces, respectively:

```
expr : <variable> { <expr> , ... }
     | [ <expr> , ... ]
```

For structs, each expression can have a distinct type (provided those types
match the corresponding struct definition). For arrays, each expression
must have the same type. Array constructors always produce rank-1 arrays; there
is no way to directly construct a higher-rank array. An empty array
constructor, `[]`, is considered type invalid.

Parentheses can be used to override precedence:

```
expr : ( <expr> )
```

Mathematical operators expect both operands to have the same type (either
integer or float) and yield a result of the same type as the input:

```
expr : <expr> + <expr>
     | <expr> - <expr>
     | <expr> * <expr>
     | <expr> / <expr>
     | <expr> % <expr>
     | - <expr>
```

Precedence is described below. Within a precedence class, evaluation
is left to right. Integer overflows wrap around in two's complement
fashion. There are no unsigned types or operators.

Integer division is defined to wrap to zero. Integer modulus is
defined so that `a` and `a % b` have the same sign. Integer division
or modulus by zero is an error. This matches the x86\_64 `idiv` instruction.
(Float division or modulus by zero returns `NaN`.)

Do not implement your own version of the modulus operator on floats!
Use the C standard library's `fmod` operation, or your chosen
language's equivalent.

(Python's default modulus operator is wrong for JPL. Python's `math.fmod`
is closer, but you must check for a zero right-hand side and return `NaN`
instead of throwing an exception.)

Keep in mind that a number of interesting floating point values exist,
such as -0, inf, and NaN. Operations on these values should follow
standard IEEE 754 semantics: `inf + 1.0 = inf`, and `0.0 / 0.0 = NaN`.
Floating point instructions should generally give you the desired
behavior for free, thanks to the FP hardware unit.

Inequality comparisons take either two integer subexpressions or two
float subexpressions and yield Booleans:

```
expr : <expr> < <expr>
     | <expr> > <expr>
     | <expr> <= <expr>
     | <expr> >= <expr>
```

Equality comparisons take two integers, two floats,
or two Booleans and yield Booleans:

```
expr : <expr> == <expr>
     | <expr> != <expr>
```

JPL has 3 Boolean operators. Both `&&` and `||` are short-circuiting:

```
expr : <expr> && <expr>
     | <expr> || <expr>
     | ! <expr>
```

Structs can be indexed with dot notation (by a variable name that matches
a field name from the corresponding struct definition). Array can be
indexed by integers. Array indexing is zero-based; the number of indexing
expressions must equal the array's rank:

```
expr : <expr>.<variable>
     | <expr> [ <expr> , ... ]
```

The conditional operator evaluates its test expression and then
exactly one branch. Both branches must have the same type:

```
expr : if <expr> then <expr> else <expr>
```

Looping constructs. `sum` computes integers or floats.

```
expr : array [ <variable> : <expr> , ... ] <expr>
     | sum [ <variable> : <expr> , ... ] <expr>
```

Each expression in the list of bindings (between the square brackets)
must produce an integer, and in the body of the loop (after the square
brackets) those variables are bound to integers. `array` expressions
yield an array whose rank is given by the number of bindings. `sum`
expressions yield an integer or a float, depending on the body
expression. If the list of bindings is empty for either `array` or
`sum`, the program is type-invalid.

If any expression in the list of bindings returns non-positive number,
that is a runtime error. It is invalid to create an `array` of zero
elements (of any rank) or to `sum` zero elements.

Function calls:

```
expr : <variable> ( <expr> , ... )
```

The type of a function call expression is the return type of the
function. Function calls can refer to either other functions defined
in the same file, or to built-in functions.

Precedence is necessary to disambiguate certain constructs. The
binding strength is:

- Postfix `[]` and `.` have the highest precedence
- Unary prefix `!` and `-` have the next-highest precedence
- Multiplicative binary operators `*`, `/`, and `%` have third highest
- Additive binary operators `+` and `-` are next
- Binary comparisons `<`, `>`, `<=`, `>=`, `==`, and `!=` are next
- Boolean binary operators `&&` and `||` are next
- Prefix `array`, `sum`, and `if` expressions have the lowest
  precedence

Within a precedence class, associativity (and evaluation order)
is left-to-right (left associative).

> For example,
>
>    array[i : N] if ! y[i] then 0 else 1 + 2 * x[i]
>
> is equivalent to
>
>    (array[i : N] (if (! (y[i])) then (0) else (1 + (2 * > (x[i])))))

### Statements

Statements begin with keywords so they cannot be confused with
expressions. Statements are pure but not total.

Let statements bind new variable names.

```
stmt : let <lvalue> = <expr>
```

A `let` statement's lvalue ("left value") is so called because it is
the left-hand argument to an assignment operator. These have two
formats, described below; the format of the lvalue must match the type
of the expression.

Assertions evaluate the expression and abort the program (after
printing the user's error message) if it is false. The expression must
return a boolean:

```
stmt : assert <expr> , <string>
```

A return statement inside a function ends execution of that function:

```
stmt : return <expr>
```

If a function has no return statements, it returns void by default.
Every return statement in a function must contain an expression that
matches the declared return type of the function. Unless the declared return
type is `void`, there must be at least one return statement.


### Commands

Commands are only available at the top level (not inside functions)
and are the only way side effects occur. Commands deal largely with
input and output.

PNG images are the main input/output format. The type for a PNG is `rgba[,]`,
that is, a rank-2 array of `rgba` pixels. `rgba` is a built-in struct defined
as follows:

```
struct rgba {
  r: float
  g: float
  b: float
  a: float
}
```

Color and alpha values should be between 0.0 and 1.0, inclusive. Values below
0.0 are clipped to 0.0 and values above 1.0 are clipped to 1.0. NaN and
negative zero map to 0.0.

```
cmd  : read image <string> to <argument>
     | write image <expr> to <string>
```

The `struct` command introduces a struct type

```
cmd  : struct <variable> {
         <variable>: <type>
         ...
       }
```

The `let` command is like a `let` statement but defines a global:

```
cmd  : let <lvalue> = <expr>
```

The `assert` command is like an `assert` statement:

```
cmd  : assert <expr> , <string>
```

There are no `return` commands.

Printing and timing statements are available for debugging purposes.

```
cmd  : print <string>
     | show <expr>
     | time <cmd>
```

Printing outputs the string followed by a newline. Showing outputs the
expression's value (recommendation: print the input expression as well
in a debugging mode, such as when interpreting a program). When outputting the
expression, `show` need not preserve parentheses or whitespace, as long as the
output expression parses to the same parse tree as the expression in the source
code. Times should be as precise as possible (at least millisecond accuracy).

Function syntax:

```
cmd  : fn <variable> ( <binding> , ... ) : <type> { ;
           <stmt> ; ... ;
       }
```

Function definitions are interpreted in order of appearance (line by line), and
may not use the same name as either another function or a builtin. Recursive
calls are allowed.


### Arguments, Lvalues, and Bindings

Binding forms in JPL allow binding a single value, or simultaneously
binding an array and its dimensions, or simultaneously binding
elements of a tuple. Lvalues are used in `let` statements while
bindings are used in argument definitions and include types. Both
share the "argument" syntax for binding variables and arrays,

Arguments can be raw variable bindings:

```
argument : <variable>
```

Or arguments can bind an array and its dimensions:

```
argument : <variable> [ <variable> , ... ]
```

It is a compile-time error if the number of dimension variables does
not equal the rank of the array. Also, only the outermost layer of a nested
array can have its dimensions bound in this fashion (since nested arrays are
not guaranteed to be rectangular).

Lvalues are merely arguments
(2025-01-22: previously, lvalues could also bind struct fields):

```
lvalue : <argument>
```

Bindings are the same as lvalues but also include types:

```
binding : <argument> : <type>
```

> Here are some example arguments and the bindings they introduce:
> 
> `x : int` takes an `int` argument
> and binds `x : int`.
> 
> `x : int[]` takes an `int[]` argument
> and binds `x : int[]`.
> 
> `x[L] : int[]` takes an `int[]` argument
> and binds `x : int[]` and `L : int`.
> 
> `x[H, W] : int[,]` takes an `int[,]` argument
> and binds `x : int[,]`, `W : int`, `H : int`.
> 
> `x[H, W] : {int, float}[][,]` takes an `{int, float}[][,]` argument
> and binds `x : {int, float}[][,]`, `W : int`, `H : int`.


Semantics
---------

A JPL program has a compilation phase and an execution phase.
(It is possible that a JPL implementation will want to blur the distinction
between these phases, e.g. because it is an interpreter or a JIT compiler.)

At compile time, a JPL implementation must reject syntactically
malformed inputs (those that are not accepted by the JPL grammar) as
well as inputs that are accepted by the grammar, but that fail to type
check. For example, a program containing the expression `a < b` where
`a` has boolean type, must be rejected at compile time. Note that
array sizes are not part of the type system. Compile-time error
messages should mention the line number where the problem was first
detected and also a brief description of the problem.

### Values

Integers behave like 64-bit two's-complement signed integers.

Floats behave like by 64-bit IEEE-754 floating point values.

Structs are laid out contiguously in memory, with all values 32-bit
aligned.

Arrays are a list of 64-bit dimension sizes and a 64-bit data pointer.

Everything is passed by value, by which we mean that the data pointer
inside an array is a reference but the rest of the array data (the
list of dimension sizes) is copied.

By splitting array sizes from their data, it makes it easy to
rematerialize that info.

### Binding

Variable bindings are lexical, meaning that every time a function is
invoked it introduces a new variable scope.

Type aliases are merely aliases, interchangable with their definition.

It is always a compile-time error for a JPL program to refer to a name
that has not yet been bound. So, for example, while this program looks
like it should typecheck, it is not legal JPL because the body of `f()`
refers to function `g()` which has not yet been bound:

```
fn f(x : int) : int {
   return g(x)
}

let y = f(3)

fn g(x : int) : int {
   return y
}
```

Shadowing is always illegal in JPL: it is a compile time error to bind
a name that is already visible from the current scope, including a
type alias. Thus, no JPL program can contain two functions with the
same name, and it is always an error to introduce a function with the
same name as a built-in function. It is not even legal to have a
function-scoped variable with the same name as a global.

JPL compilers must provide builtin functions including the following
math functions:

+ Of one `float` argument, returning a `float`:
  `sqrt`, `exp`, `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, and `log`
+ Of two `float` arguments, returning a `float`:
  `pow` and `atan2`
+ The `to_float` function, which converts an `int` to a `float`
+ The `to_int` function, which converts a `float` to an `int`, with
  positive and negative infinity converting into the maximum and
  minimum integers, and NaN converting to 0.

They can also provide builtin functions whose name begins with
`__JPL__` for the compiler to use during compilation.

JPL must provide a global `args` variable of type `int[]` containing
integers provided in the command line program's command line, and a
variable `argnum` of type `int` containing the number of arguments.
The program name itself should not be part of that list or that count.
It is an error to invoke a JPL program with command-line arguments
that are not 64-bit signed integer values.

> Consider the following JPL program:
>
> ```
> show args
> ```
> 
> Call this program like so:
>
>     ./program 1 2 3 4
>
> It must print at least `[1, 2, 3, 4]`
>
> It may print `args = [1, 2, 3, 4]`.

### Errors

At run time, a JPL implementation must detect erroneous conditions. If
any such condition occurs, the JPL program must be cleanly terminated
(no segfaults or other OS-level traps!) and a brief, appropriate error
message must be displayed that begins with the text `[abort]`.
Run-time errors include:

- an integer division or modulus operation by zero
  * Floating-point divisions and modulus operations with zero right-hand
    arguments are not erroneous; they return NaN.

- out-of-bounds array access

- `sum` or `array` with non-positive bounds

- any failing assertion

Or they may be *external* errors:

- attempting to read a file that does not exist

- attempting to write a file to a full disk

- attempting to write to a write-protected location

- any other I/O function failing

- out of memory

When an internal or external error occurs, the process running the
compiled JPL program should exit with non-zero status code.

JPL compilers must preserve internal errors: the compiler should not
change a program with internal errors into one without, or vice versa.
JPL compilers must therefore emit code to check integer division
arguments, array bounds, and assertions. It is acceptable to not emit
that code when the JPL compiler can prove that the assertion cannot
fail. JPL compilers must also preserve termination and
non-termination.

JPL compilers need not strictly preserve external errors---that is
impossible without a strict contract from the operating system---but
should preserve the order and arguments of system calls, except for
the time information in `time` commands. In practice, this is not
hard, because only the top-level commands have I/O effects.

JPL compilers also need not preserve the type of internal or external
error (for example, bounds checks could be implemented as assertions).

There are some rarer exceptional conditions, like stack overflow,
where JPL programs are allowed to segfault or otherwise terminate
uncleanly, and need not be preserved. As long as the compiler does not
go out of its way to mess with this things should be fine.

Elaboration
-----------

Compiler implementors may want to make their jobs easier by
elaborating certain JPL constructs into other ones that need to be
implemented anyway.

### Short-circuiting

It is convenient to elaborate short-circuting `&&` and `||` via `if`
statements:

    A && B -> if A then B else false
    A || B -> if A then true else B

### Errors

A JPL implementation may implement error checks by inserting
assertions into the program as it is being compiled. For example, a
function like this:

    fn example(i : int, j : int) : int {
      return i / j
    }

can be transformed into:

    fn example(i : int, j : int) : int {
        return __JPL__divide(i, j)
    }

where `__JPL__divide` is provided by the JPL implementation and looks like:

    fn __JPL__divide(i : int, j : int) : int {
        assert j != 0, "Error: Division by zero"
        return i / j
    }

### Arrays

It is helpful to introduce a type, `data`, for a pointer to data.
Then an array like `int[,]` can be represented in memory by the tuple
`{int, int, data}`, where the first two integers are the dimension.
Since arrays are immutable, it doesn't matter whether the tuple is
passed by reference or by value (though `data` should always be passed
by reference!) and in many cases it won't need to be, since the array
size going to be computable from other sources.

### Commands

It is convenient to convert commands into calls to builtin functions.

Since commands often deal with strings it's convenient to convert
strings to arrays of integers. For example, in the command

```
read image "test.png" to a
```

the string `"test.png"` could be converted to the array

```
let filename = [ 116, 101, 115, 116, 46, 112, 110, 103, 0 ]
```

where a null terminator is inserted for compatibility with C-style
strings. Then the `read` statement itself might be converted to a call
to a `__read__image` function (note the underscores, marking it as a compiler
internal):

```
let a = __read__image(filename)
```

Once all commands have been converted to builtin functions, the top
level contains only statements and can be converted to a function.

Implementation Limits
---------------------

A JPL compiler is not obligated to support a nesting depth (as
measured by the height of the program AST following the grammar in
this specification) larger than 64, nor is it required to support
arrays of rank larger than 64, tuples wider than 64 elements, or
functions that take more than 64 arguments.  Basically, almost any
occurrence of `...` in this specification only needs to be expanded 64
times by a JPL compiler. An exception is the `...` indicating the
repetition of statements in a function body: this should be limited
only by the amount of memory on the machine running the JPL compiler.
Similarly, the number of elements in an array should be limited only
by memory size.

JPL Compiler Command Line Interface
-----------------------------------

Every valid invocation of a JPL compiler should print
`Compilation failed` or `Compilation succeeded`, followed
by a colon (`:`), a brief explanation, and a newline.

Examples:

```
Compilation succeeded: running the code complete

Compilation succeeded

Compilation failed: test.jpl:2:11: Mixed left- and right-types float, int

Compilation failed: test.jpl:5:0: Unknown command beginning with <class 'lexer.Identifier'>
```

Print to the standard output stream (stdout). Do not print anything to the
standard error stream (stderr).

Compilation should succeed if the input program is legal JPL. Compilation
should fail if the input program is not legal JPL, and the explanation should
describe what is wrong with the input program. A JPL compiler should produce no
other output, except in cases described below.

A JPL compiler is required to support the following command line options,
which may occur in any order:

- Exactly one filespec, which specifies both a path and a filename. A
  filespec may be relative to the current working directory, or it may
  be absolute. The filespec describes the location of a file
  containing JPL code to be compiled.

- Zero or one flags indicating actions taken by the compiler:

  - `-l` (lex): Perform lexical analysis only, printing the tokens to
    stdout. In this case, the compilation is considered to be
    successful if the input file contains only the lexemes described
    in this spec; otherwise, the compilation fails.

  - `-p` (parse): Perform lexical analysis and parsing only, pretty-printing
    the parsed program back to ASCII text in a format based on
    s-expressions (to be described in homework assignments).  In this
    case, the compilation is considered to be successful if the input
    program corresponds to the grammar described in the current
    assignment; otherwise, the compilation fails.

  - `-t` (typecheck): Perform lexical analysis, parsing, and type checking (but not
    code generation). In this case, the compilation is considered to be
    successful if the input program is fully legal JPL; otherwise
    the compilation fails.

  - `-i` (IR): Perform lexical analysis, parsing, and type checking, then output
    intermediate representation C code.

  - `-s` (assembly): Perform lexical analysis, parsing, and type checking, then
    generate assembly code.

If the command line options to the JPL compiler do not meet these
requirements, or if the specified file does not exist or cannot be
accessed, print a brief error message and exit. Do not print
`Compilation failed` (and definitely not `Compilation succeeded`).

It is permissible for a JPL compiler to accept additional single-letter command
line flags. For example, `-d` might be used to ask the JPL compiler to produce
debugging output. When such a flag is specified, it is understood that the JPL
compiler is operating outside of this spec. Thus, you can print whatever output
you want. Additional flags must be off by default.

