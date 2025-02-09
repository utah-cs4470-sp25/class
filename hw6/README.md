Assignment 6: Type-checking Expressions
=======================================

For this assignment, you will write a type checker for structs
and some JPL expressions.
The type checker takes an AST as input, computes the type of every expression
within the AST, and raises an error if it detects an issue.


## Language

Typecheck the following subset of JPL:

```
type : int
     | bool
     | float
     | <type> [ , ... ]
     | <variable>
     | void

cmd : show <expr>
    | struct <variable> { ;
          <variable>: <type> ; ... ;
      }

expr : <integer>
     | <float>
     | <variable>
     | true
     | false
     | <expr> + <expr>
     | <expr> - <expr>
     | <expr> * <expr>
     | <expr> / <expr>
     | <expr> % <expr>
     | - <expr>
     | <expr> < <expr>
     | <expr> > <expr>
     | <expr> == <expr>
     | <expr> != <expr>
     | <expr> <= <expr>
     | <expr> >= <expr>
     | <expr> && <expr>
     | <expr> || <expr>
     | ! <expr>
     | <variable> { <expr> , ... }
     | [ <expr> , ... ]
     | ( <expr> )
     | if <expr> then <expr> else <expr>
     | <expr> . <variable>
     | <expr> [ <expr> , ... ]
```

There are two commands: `struct` declarations and `show` commands.

The expressions in this subset include all of JPL's kinds of types, like
integers, floats, booleans, arrays of various ranks, and structs.
For every expression, you must compute a type
using JPL's [type checking rules](../spec.md#Expressions).
Formal type rules are in the slides on Type Rules from Feb. 3:

* <https://utah.instructure.com/courses/1021427/pages/02-slash-03-slides-+-notes>

Make sure to include a built-in type for `rgba` structs:

```
struct rgba {
  r: float
  g: float
  b: float
  a: float
}
```

Your parser must implement the `-t` command-line flag and print `Compilation
failed` or `Compilation succeeded` when type checking. The autograder will
look for these strings.


## Printing the output

Your type checker must produce a type-annotated S-expression when compilation
succeeds. In addition to the output from HW5, the first field of each expression
AST node must print its type. For example, this program:

    show -48.3 / 24.1

Must produce the following output:

    (ShowCmd
     (BinopExpr
      (FloatType)
      (UnopExpr (FloatType) - (FloatExpr (FloatType) 48))
      /
      (FloatExpr (FloatType) 24)))

Note that the `BinopExpr`, `UnopExpr`, and `FloatExpr` nodes have the type as
the first argument---in this expression, all four nodes have `(FloatType)`.
Types are also printed as S-expressions. The `ShowCmd` does not print a
type because it is not an expression.


## Hints and Advice

[hints.md](./hints.md)

Define a top-level `typecheck` function (or method) that iterates over
the list of `cmd`s in a program. It should maintain an environment
with information about all struct declarations seen so far. It should
also typecheck each expression within each `show` command.

Define a `type_of` function that takes an expression AST node and
an environment (`ctx` below, with information about all struct declarations
that are in scope) and produces a type as output.

    def type_of(exp: ExprAST, ctx: Environment) -> Type:
        ....

`type_of` should:

- Recursively compute the type of each subexpression
- Apply the type rules to determine what the type of the output is
- Raise a type error if the type rules are violated
- Store the output type in the input AST node (useful when printing S-expressions)
- Return the output type as well (useful in recursive `type_of` calls)

In your compiler, call `typecheck` after parsing. Then use your exising
S-expression printer (modified to print types too) to print the results.


# Testing your code

The reference JPL compiler supports the `-t` flag. You can run it on any test
file to see the correct typechecking of that file.

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-t

The `hw6` autograder directories `ok` (Part 1) and `ok-fuzzer` (Part 2) contain
valid JPL programs, which you must typecheck.

The ones named `fail-fuzzer1` (Part 3), `fail-fuzzer2` (Part 4), and
`fail-fuzzer3` (Part 5) have programs with type errors.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw6


# Submission and grading

This assignment is due Friday Feb 14. Happy Valentines Day.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 45%    | Part 1   |
| 25%    | Part 2   |
| 10%    | Part 3   |
| 10%    | Part 4   |
| 10%    | Part 5   |

