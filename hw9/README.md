Assignment 9: More C
====================

Compile the full JPL language to Ugly C.
 
```diff
cmd : show <expr>
    | let <lvalue> = <expr>
    | assert <expr> , <string>
    | struct <variable> { ; <variable>: <type> ; ... ; }
+   | read image <string> to <lvalue>
+   | write image <expr> to <string>
+   | print <string>
+   | time <cmd>
+   | fn <variable> ( <binding> , ... ) : <type> { ;
+         <stmt> ; ... ;
+     }

+stmt : let <lvalue> = <expr>
+     | assert <expr> , <string>
+     | return <expr>

lvalue : <variable>
       | <variable> [ <variable> , ... ]

binding : <lvalue> : <type>

type : int
     | bool
     | float
     | <type> [ , ... ]
     | <variable>
     | void

expr : <integer>
     | <float>
     | true
     | false
     | <variable>
     | void

expr : - <expr>
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

expr : <variable> { <expr> , ... }
     | [ <expr> , ... ]
     | <expr> [ <expr> , ... ]
     | <expr> . <variable>
     | if <expr> then <expr> else <expr>

+expr : <expr> && <expr>
+     | <expr> || <expr>
+     | array [ <variable> : <expr> , ... ] <expr>
+     | sum [ <variable> : <expr> , ... ] <expr>
```

As with HW8, your C code does not need to run. The autograder will compare `.c`
files directly.

See [../hw8/running.md](../hw8/running.md) for instructions on how to actually run the C
code. Ask for help on Discord.


#### Hints



# Testing your code

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-i

The grader directory `ok` (Part 1) contains valid hand-generated JPL programs.
The directory `ok-fuzzer` contains valid fuzzer-generated programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw9


# Submission and grading

This assignment is due Friday March 7 (just before Spring Break).

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 90%    | Part 1   |
| 10%    | Part 2   |

