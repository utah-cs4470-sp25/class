Assignment 4: Parsing II: Functions, Recursion
======================================

For this assignment you will continue working on your compiler's parser;
modifying it to parse most JPL syntax, especially the recursive parts.


## JPL subset, Part 2

Extend your parser from [Homework 3](../hw3/README.md) with the following
additional options:

```diff
+type : int
+     | bool
+     | float
+     | <type> [ , ... ]
+     | <variable>
+     | void

 cmd  : read image <string> to <lvalue>
      | write image <expr> to <string>
      | let <lvalue> = <expr>
      | assert <expr> , <string>
      | print <string>
      | show <expr>
      | time <cmd>
+     | fn <variable> ( <binding> , ... ) : <type> { ;
+           <stmt> ; ... ;
+       }
+     | struct <variable> { ;
+           <variable>: <type> ; ... ;
+       }

+stmt : let <lvalue> = <expr>
+     | assert <expr> , <string>
+     | return <expr>

 expr : <integer>
      | <float>
      | true
      | false
      | <variable>
      | [ <expr> , ... ]
+     | variable> { <expr> , ... }
+     | ( <expr> )
+     | <expr> . <variable>
+     | <expr> [ <expr> , ... ]
+     | <variable> ( <expr> , ... )

 lvalue : <variable>
+       | <variable> [ <variable> , ... ]

+binding : <lvalue> : <type>
```

Note that in this grammar the semicolon `;` represents a `NEWLINE`
token. Informally, this subset is all of JPL except for most of the
expressions.

The new AST nodes that your S-expression output must include are:

```
IntType
FloatType
BoolType
ArrayType
VoidType
StructType
ArrayIndexExpr
DotExpr
CallExpr
FnCmd
StructCmd
LetStmt
AssertStmt
ReturnStmt
ArrayLValue
```

Parenthesized expressions, as in the grammar rule `expr : ( <expr> )`,
do not produce AST nodes.

## Hints and Advice

[hints.md](./hints.md)


# Testing your code

We will run your compiler the same as for HW3:

    make run TEST=/grader/ok/001.jpl FLAGS=-p

As in Homework 3, the `-p` flag must cause your compiler to produce
S-expression output and then print `Compilation failed` or `Compilation
succeeded`. We will expect this output format when testing your code.

The directories `ok` (Part 1) and `ok-fuzzer` (Part 2) contain valid
JPL programs, which your parser must parse correctly.

The ones named `fail-fuzzer1` (Part 3), `fail-fuzzer` (Part 4), and
`fail-fuzzer3` (Part 5) contain invalid programs that your parser must
raise an error on.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw4

Generally speaking, Part 1 is somewhat easier than the other parts and
we recommend you start there. Parts 3, 4, and 5 are the hardest. Save
them for later.


# Submission and grading

This assignment is due Friday Jan 31.

We are happy to discuss problems and solutions with you on Discord, in
office hours, or by appointment.

The weight of each part is:

| Weight | Function |
|--------|----------|
| 45%    | Part 1   |
| 25%    | Part 2   |
| 10%    | Part 3   |
| 10%    | Part 4   |
| 10%    | Part 5   |
