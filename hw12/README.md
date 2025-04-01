Assignment 12: ASM Control Flow
===============================

Generate assembly for JPL ifs, loops, and array indexing. Add support for the
built-in variables `args` and `argnum`.

Except for half the fuzzer tests, nothing in this assignment
depends on user-defined JPL functions.

```diff
cmd : show <expr>
    | let <lvalue> = <expr>
    | fn <variable> ( <binding> , ... ) : <type> { ;
          <stmt> ; ... ;
      }

stmt : let <lvalue> = <expr>
     | return <expr>

lvalue : <variable>
       | <variable> [ <variable> , ... ]

binding : <lvalue> : <type>

type : int
     | bool
     | float
     | <type> [ , ... ]

 expr : <variable>
      | <integer>
      | <float>
      | true
      | false
      | - <expr>
      | ! <expr>
      | <expr> + <expr>
      | <expr> - <expr>
      | <expr> * <expr>
      | <expr> / <expr>
      | <expr> % <expr>
      | <expr> < <expr>
      | <expr> > <expr>
      | <expr> == <expr>
      | <expr> != <expr>
      | <expr> <= <expr>
      | <expr> >= <expr>
      | [ <expr> , ... ]
+     | if <expr> then <expr> else <expr>
+     | <expr> [ <expr> , ... ]
+     | array [ <variable> : <expr> , ... ] <expr>
+     | sum [ <variable> : <expr> , ... ] <expr>
```

More info: [hints.md](./hints.md)


# Testing your code

We will run your compiler with the `-s` flag:

    make run TEST=/grader/ok/001.jpl FLAGS=-s

The grader's `hw12/` folder has six directories:
- `if/` (Part 1): conditionals
- `index/` (Part 2): array indexing
- `sum/` (Part 3): sum loops
- `array/` (Part 4): array loops and more indexing
- `args/` (Part 5): uses of args and argnum, more array indexing
- `fuzzer/` (Part 6): 10 easier and 10 harder auto-generated files

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw12


# Submission and grading

This assignment is due Friday April 4.

We are happy to discuss problems and solutions with you on Discord, in
office hours, or by appointment.

Most tests are worth 1% of the total. The fuzzer tests are worth 0.5% each.

| Weight | Function     |
|--------|--------------|
| 15%    | Part 1       |
| 12%    | Part 2       |
| 10%    | Part 3       |
| 28%    | Part 4       |
| 25%    | Part 5       |
| 10%    | Part 6       |

