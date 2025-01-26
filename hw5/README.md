Assignment 5: Parsing III: Expressions, Infix, Prefix
=================================

For this assignment you will finish the parser by extending it to handle JPL's
full expression syntax, including the correct handling of precedence.


## The JPL subset

Formally, for this assignment you'll extend your parser from [HW 4](../hw4/README.md)
with the new expressions highlighted in the full grammar below:

```diff
 type : int
      | bool
      | float
      | <type> [ , ... ]
      | <variable>
      | void

 cmd  : read image <string> to <lvalue>
      | write image <expr> to <string>
      | let <lvalue> = <expr>
      | assert <expr> , <string>
      | print <string>
      | show <expr>
      | time <cmd>
      | fn <variable> ( <binding> , ... ) : <type> { ;
            <stmt> ; ... ;
        }
      | struct <variable> { ;
            <variable>: <type> ; ... ;
        }

 stmt : let <lvalue> = <expr>
      | assert <expr> , <string>
      | return <expr>

 expr : <integer>
      | <float>
      | true
      | false
      | <variable>
      | void
      | [ <expr> , ... ]
      | variable> { <expr> , ... }
      | ( <expr> )
      | <expr> . <variable>
      | <expr> [ <expr> , ... ]
      | <variable> ( <expr> , ... )
+     | <expr> + <expr>
+     | <expr> - <expr>
+     | <expr> * <expr>
+     | <expr> / <expr>
+     | <expr> % <expr>
+     | - <expr>
+     | <expr> < <expr>
+     | <expr> > <expr>
+     | <expr> == <expr>
+     | <expr> != <expr>
+     | <expr> <= <expr>
+     | <expr> >= <expr>
+     | <expr> && <expr>
+     | <expr> || <expr>
+     | ! <expr>
+     | if <expr> then <expr> else <expr>
+     | array [ <variable> : <expr> , ... ] <expr>
+     | sum [ <variable> : <expr> , ... ] <expr>

 lvalue : <variable>
        | <variable> [ <variable> , ... ]

 binding : <lvalue> : <type>
```


Importantly, this grammar is ambiguous as written, and must be
disambiguated using JPL's [precedence rules](../spec.md#Expressions):

| Operation                                            | Associativity |
|------------------------------------------------------|---------------|
| Indexing with `{ <integer> }` and `[ <expr> , ... ]` | Postfix       |
| Unary inverse `!` and negation `-`                   | Prefix        |
| Multiplicative operations `*`, `/`, and `%`          | Left          |
| Additive operations `+` and `-`                      | Left          |
| Comparison `<`, `>`, `<=`, and `>=`, `==`, `!=`      | Left          |
| Boolean operators `&&` and `\|\|`                    | Left          |
| Prefix operators `array`, `sum`, and `if`            | Prefix        |

The first row is the highest precedence. Note that unlike many
languages, `&&` and `||` have the same precedence in JPL, so an
expression like `a || b && c` is parsed as `(a || b) && c`. This
makes parsing simpler. All operators are left-associative.

Even if you think that both parse trees are equally good (like for
`1 + 2 + 3`), your parser *must* return the parse tree specified by
the associativity and precedence above (that is, `(1 + 2) + 3`). This
goes even for expressions where the types don't work; for example, you
*must* parse `a < b < c` as `(a < b) < c` even though that is type
invalid.

The new AST nodes that your S-expression output must include are:

```
UnopExpr
BinopExpr
IfExpr
ArrayLoopExpr
SumLoopExpr
```

## Hints and Advice

[hints.md](./hints.md)

The grammar above is NOT something that you can directly implement.
**Email the instructors** (before the deadline please) if you'd like feedback
on your disambiguated grammar before you start implementing it.


# Testing your code

We will run your compiler the same as for HW3 and HW4:

    make run TEST=/grader/ok/001.jpl FLAGS=-p

The directories `ok` (Part 1) and `ok-fuzzer` (Part 2) contain valid
JPL programs, which your parser must parse correctly.

The ones named `fail-fuzzer1` (Part 3), `fail-fuzzer2` (Part 4), and
`fail-fuzzer3` (Part 5) contain invalid programs that your parser must
raise an error on.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw5

Like last time, start with Part 1.


# Submission and grading

This assignment is due Friday Feb 7.

We are happy to discuss problems and solutions with you on Discord, in
office hours, or by appointment.

| Weight | Function |
|--------|----------|
| 45%    | Part 1   |
| 25%    | Part 2   |
| 10%    | Part 3   |
| 10%    | Part 4   |
| 10%    | Part 5   |

