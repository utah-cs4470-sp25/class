Assignment 3: Parsing I: Commands, Prefix
======================================

For this assignment you will write a parser for a subset of
JPL, focusing on three key steps:

- Defining the AST classes
- Parsing commands
- Parsing sequences

You will finish the parser in HW4 and HW5.


## JPL subset

```
cmd  : read image <string> to <argument>
     | write image <expr> to <string>
     | let <lvalue> = <expr>
     | assert <expr> , <string>
     | print <string>
     | show <expr>
     | time <cmd>   # beware! recursion!

type : int
     | bool
     | float
     | <type> [ , ... ]   # beware! possibly-empty sequence!
     | <variable>
     | void

expr : <integer>
     | <float>
     | true
     | false
     | <variable>
     | [ <expr> , ... ]  # beware! recursion! possibly-empty sequence!

argument : <variable>

lvalue : <argument>
```

Recall that a JPL program is a sequence of newline-terminated
commands (`cmd`).

Our HW3 subset has almost all the commands (only function definitions
are missing) and all of the types. It does not include statements.
Its expressions are simple, except for the array constructor,
and both arguments and lvalues are as simple as possible.

Build a good foundation this week for the upcoming parser assignments. HW4 and
especially HW5 will assume you are working from a solid foundation.

To demonstrate that your parser works, you must to implement the
parse-only mode of the [command line specification][jpl-p]. In this
mode, your compiler should parse the input file and produce output
in an S-expression format like so:

[jpl-p]: https://github.com/utah-cs4470-sp25/class/blob/main/spec.md#jpl-compiler-command-line-interface

```
(ReadCmd "photo.png" (VarArgument photo))
(AssertCmd (VarExpr a) "Photo must be 800 pixels wide")
(AssertCmd (VarExpr b) "Photo must be 600 pixels tall")
(LetCmd (ArgLValue (VarArgument middle)) (VarExpr photo))
(WriteCmd (VarExpr middle) "profile.png")
```

Your compiler must also print `Compilation failed` when a parser error
is encountered and `Compilation succeeded` when parsing works. The autograder
will look for these messages.


## Output Format

Your parser must produce output in the following S-expression format.
Every AST node should print as:

- An open parenthesis
- Followed by the name of the AST node
- Followed by each field of the AST node
  - each preceded by a space
  - sorted in the same order as in the grammar
  - if it is an AST node field, print its S-expression
  - if it is a `<string>` field, print it with quotation marks
  - otherwise, just print the field value directly
- Followed by a close parenthesis

> In general, an S-expression is a tree where each node is wrapped in
> parentheses and each leaf is a value literal. E.g. `42` and `(42)`
> and `((42) 43)` are each S-expressions, with 0, 1, and 2 nodes.

You must print the names that the autograder expects. In your code,
however, you can name AST nodes any way you prefer. We recommend
that each AST node you implement has a `toString` method (or equivalent)
that prints the node as an S-expression string.


Example: the grammatical rule for `read` is:

```
    cmd : read image <string> to <argument>
```

The corresponding AST node must print with the name `ReadCmd` and show two
fields, a string and an argument. Its `toString` method would look something
like this:

```
    public String toString() {
        out = "(ReadCmd";
        out += " " + string_to_s_expression(this.filename);
        out += this.arg.toString();
        out += ")";
        return out;
    }
```

Here the method `string_to_s_expression` takes a `String` argument and returns
that string with double quotes around it:

```
    public String string_to_s_expression(String input) {
        return "\"" + input + "\"";
    }
```


The AST node names that you must use in your S-expression output are:

```
ReadCmd
WriteCmd
LetCmd
AssertCmd
PrintCmd
ShowCmd
TimeCmd
IntType
FloatType
BoolType
VarType
ArrayType
VoidType
IntExpr
FloatExpr
TrueExpr
FalseExpr
VarExpr
ArrayLiteralExpr
VarArgument
ArgLValue
```

Printing floats is very difficult, and differs across languages, so
cast any float arguments to 64-bit integers and print those. For
example, the JPL expression `3.14159` would print as `(FloatExpr 3)`.
In C++ you can do that by printing with `%ld`; in Python with `{:d}`;
in Java, cast to a `long` and print that.

The auto-grader will automatically indent your S-expressions and split
them across lines in a standard way. Don't worry about indenting and
splitting lines in your code.

## Hints and Advice

[hints.md](./hints.md)


# Testing your code

The JPL interpreter in the "[Releases][releases]" supports the `-p`
operation you are being asked to implement. You can run it on any test
file to see the correct parsing of that file:

    ~/Downloads $ ./jplc-macos -p ~/jpl/examples/cat.jpl
    (ReadCmd "input.png" (VarArgument img))
    (WriteCmd (VarExpr img) "output.png")
    Compilation succeeded: parsing complete

Naturally, this JPL interpreter is a program and can have bugs. If you
think you've found one, contact the instructors.

[releases]: https://github.com/utah-cs4470-sp25/class/releases

For larger programs, it can be tedious to compare the S-expressions by
hand. Instead, save each output to a file and use `diff` to compare:

    diff wrong-output.txt right-output.txt

Moreover, for complex S-expressions it can still be hard to see the
difference when a whole command is all on one line. The JPL compiler
supports a special `--pp-sexp` command which pretty-prints
S-expressions. For example, if you run the parser on some long and
complex file:

    ~/Downloads $ ./jplc-macos -p ~/jpl/examples/nn/nn.jpl > out.sexp

You can then convert the output, `out.sexp`, to a pretty-printed
`out.pp` like this:

    ~/Downloads $ ./jplc-macos --pp-sexp out.sexp > out.pp

If you pretty-print both files before calling `diff` on them, it can
be easier to find the difference.

You can find the tests and expected outputs [in the auto-grader
repository](https://github.com/utah-cs4470-sp25/grader/tree/main/hw3).
These tests come in five parts, corresponding to the five directories
of tests.

The directories `ok` (Part 1) and `ok-fuzzer` (Part 2) contain valid
JPL programs, which your parser must parse correctly.

The ones named `fail-fuzzer1` (Part 3), `fail-fuzzer` (Part 4), and
`fail-fuzzer3` (Part 5) contain invalid programs that your parser must
raise an error on.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part>

Generally speaking, Part 1 is somewhat easier than the other parts.
Start there. Parts 3, 4, and 5 are the hardest. Save them for last.

Depending on how exactly you write your parser, you might find that you pass
Parts 3, 4, and 5 very early. This may happen because you haven't actually
implemented any of the JPL subset and therefore reject all programs. That means
you successfully reject invalid programs, passing Parts 3, 4, and 5. However,
don't get too excited: you're rejecting these programs for the wrong reason,
and will probably stop rejecting some of them as you implement more of the HW3
JPL subset.


# Submission and grading

This assignment is due Friday January 24.

We are happy to discuss problems and solutions with you on Discord, in
office hours, or by appointment.

Your compiler must be runnable as described in the [Testing your
code][Testing your code] section. If the auto-grader cannot run your
code, you will not receive credit. The auto-grader output is available
to you at any time, as many times as you want. Make use of it.

The weight assigned to each part is:

| Weight | Function |
|--------|----------|
| 45%    | Part 1   |
| 25%    | Part 2   |
| 10%    | Part 3   |
| 10%    | Part 4   |
| 10%    | Part 5   |

Your solutions will be auto-graded. The auto-grader will use Github
Actions and runs on Ubuntu using the tests described above.
