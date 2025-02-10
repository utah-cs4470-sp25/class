Assignment 7: Typing Variables and Functions
===

For this assignment, extend your type checker to handle variable
declarations, functions, and everything else in JPL. Your typechecker
will need to manage environments (aka symbol tables).

Don't forget the JPL built-ins, such as `argnum`. They all need types:

<https://github.com/utah-cs4470-sp25/class/blob/main/spec.md#binding>


## Language

Typecheck the full JPL language. The diff below shows what your HW6
typechecker must be extended to handle:


```diff
 type : int
      | bool
      | float
      | <type> [ , ... ]
      | <variable>
      | void

+cmd  : read image <string> to <lvalue>
+     | write image <expr> to <string>
+     | let <lvalue> = <expr>
+     | assert <expr> , <string>
+     | print <string>
      | show <expr>
+     | time <cmd>
+     | fn <variable> ( <binding> , ... ) : <type> { ;
+           <stmt> ; ... ;
+       }
      | struct <variable> { ;
            <variable>: <type> ; ... ;
        }

+stmt : let <lvalue> = <expr>
+     | assert <expr> , <string>
+     | return <expr>

 expr : <integer>
      | <float>
      | true
      | false
+     | <variable>
      | void
      | [ <expr> , ... ]
      | variable> { <expr> , ... }
      | ( <expr> )
      | <expr> . <variable>
      | <expr> [ <expr> , ... ]
      | <variable> ( <expr> , ... )
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
      | if <expr> then <expr> else <expr>
+     | array [ <variable> : <expr> , ... ] <expr>
+     | sum [ <variable> : <expr> , ... ] <expr>

+lvalue : <variable>
+       | <variable> [ <variable> , ... ]

+binding : <lvalue> : <type>
```

You must also support all JPL built-ins. Their names and types are in
the specification:

<https://github.com/utah-cs4470-sp25/class/blob/main/spec.md#binding>


## Output format

As in HW6, your compiler must support the `-t` flag. If typechecking succeeds,
print an S-expression representation of the program. Each expression in the
output must include its type as its first field.


## Hints and Advice

[hints.md](./hints.md)


# Testing your code

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-t

The directories `ok` (Part 1) and `fail` (Part 2) contain valid and
invalid hand-written JPL programs to guide you through this
assignment.

The `ok-fuzzer` (Part 3), and `fail-fuzzer` (Part 4) directories
contain valid and invalid auto-generated JPL programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw7


# Submission and grading

This assignment is due Friday Feb 21.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 30%    | Part 1   |
| 30%    | Part 2   |
| 25%    | Part 3   |
| 15%    | Part 4   |

