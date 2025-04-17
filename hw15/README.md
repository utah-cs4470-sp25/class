Assignment 15: More ASM
=======================

Compile the full JPL language to x86_64.
 
```diff
cmd : show <expr>
    | let <lvalue> = <expr>
+   | assert <expr> , <string>
+   | struct <variable> { ; <variable>: <type> ; ... ; }
+   | read image <string> to <lvalue>
+   | write image <expr> to <string>
+   | print <string>
+   | time <cmd>
    | fn <variable> ( <binding> , ... ) : <type> { ;
          <stmt> ; ... ;
      }

 stmt : let <lvalue> = <expr>
+     | assert <expr> , <string>
      | return <expr>

lvalue : <variable>
       | <variable> [ <variable> , ... ]

binding : <lvalue> : <type>

type : int
     | bool
     | float
     | <type> [ , ... ]
+    | <variable>
+    | void

expr : <integer>
     | <float>
     | true
     | false
     | <variable>
+    | void

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
+    | <variable> { <expr> , ... }
     | [ <expr> , ... ]
     | <expr> [ <expr> , ... ]
+    | <expr> . <variable>
     | if <expr> then <expr> else <expr>
+    | <expr> && <expr>
+    | <expr> || <expr>
     | array [ <variable> : <expr> , ... ] <expr>
     | sum [ <variable> : <expr> , ... ] <expr>
```


# Testing your code

There are 6 test folders:

- `ok1/` many small programs
- `ok2/` a few interesting programs
- `ok3/` programs to optimize (`-O3`)
- `fail-fuzzer1/` programs that should not typecheck
- `ok-fuzzer1/` valid auto-generated programs
- `ok-fuzzer2/` more auto-generated programs

For Part 3, we will run your code once with `-s` and once with `-s -O3`:

    make run TEST=/grader/ok/001.jpl FLAGS=-s
    make run TEST=/grader/ok/001.jpl FLAGS="-s -O3"

For Part 4 (`fail-fuzzer1`), we will only typecheck. The programs should all fail:

    make run TEST=/grader/ok/001.jpl FLAGS=-t

For the other parts, we will run your code with the `-s` flag:

    make run TEST=/grader/ok/001.jpl FLAGS=-s

You can run all tests by downloading the auto-grader and running this command:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw15


# Submission and grading

This assignment is due as late as possible: Friday April 30.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 40%    | Part 1   |
| 20%    | Part 2   |
| 10%    | Part 3   |
| 20%    | Part 4   |
|  5%    | Part 5   |
|  5%    | Part 6   |

