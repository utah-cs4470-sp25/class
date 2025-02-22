Assignment 8: Generating C
==========================

**Feb 22: this is a draft SUBJECT TO CHANGE. Will finalize by Feb 24.**

For this assignment, compile the following subset of JPL to ugly straight-line
C code:

```
cmd : show <expr>
    | let <lvalue> = <expr>
    | assert <expr> , <string>
    | struct <variable> { ; <variable>: <type> ; ... ; }

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
```

*Your C code does not need to run.* The autograder will only check that it
matches the `.expected` files character-by-character (ignoring whitespace,
comments, etc.).

See [running.md](./running.md) for instructions on how to actually run the C
code. Ask for help on Discord.

#### TLDR hints:

* Generate `typedef struct` code for every struct command and every array
  literal in the order that they appear in the JPL program.
* Keep a counter for variable names, generate names in order: `_1`, `_2`, `_3`, ...
* Convert JPL types to C types as follows:
  - `int` -> `long long`
  - `float` -> `double`
  - `bool` -> `bool`
  - `void` -> custom struct (see header in any `.expected` file)
  - array -> struct with one int field `di` per dimension and one pointer field `data` for the elements
    + Example: `float[,]` -> `typedef struct { long long d1 ; long long d2 ; double *data ; } _a1_double;`
  - struct -> struct name (which will be defined by an earlier struct command)


Look at `runtime.c` for helpers, such as `jpl_alloc`.

`show` needs a type string as its first argument. For everything except structs, use the
type S-expression printer you wrote for HW6/HW7. For structs, print a tuple type with
the element types inlined.

Example:

```
struct taco {
  shell: int
  ingredients: int[]
}
show(taco{2, [4,5]})
```

Output:

```
...
show("(TupleType (IntType) (ArrayType (IntType) 1))", &_6);
```

For extra credit, change `runtime.c` to print `StructType`s nicely. Details below.


# Testing your code

We will run your compiler this way:

    make run TEST=/grader/ok/001.jpl FLAGS=-i

The grader directory `ok` (Part 1) contains valid hand-generated JPL programs.
The directory `ok-fuzzer` contains valid fuzzer-generated programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw8


# Submission and grading

This assignment is due Friday Feb 21.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 90%    | Part 1   |
| 10%    | Part 2   |

For a 5% extra credit, modify your compiler and `runtime.c` so that its show
function can print using a `(StructType t)` as input. It should work for any
user-defined struct. Create a diff showing your changes to `runtime.c` and
create a small but rigorous example program to demonstrate how your solution
works. Email the diff, the JPL program, and the C output for that program
to Ben and Bhargav. They will grade the attempt on a pass/fail basis; you
have one shot at the 5% bonus. (Then undo the changes to your compiler so you
can pass the autograder again!) 


