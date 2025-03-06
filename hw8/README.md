Assignment 8: Generating C
==========================

For this assignment, compile the following subset of JPL to ugly straight-line
C code:

**Update Feb 27: added `print` and call expressions because autograder test `hw8/ok/084.jpl`
uses calls to built-in functions and several `ok-fuzzer` tests use `print`.**

```diff
cmd : show <expr>
    | let <lvalue> = <expr>
    | print <string>
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
+    | <var> ( <expr> , ... )
```

*Your C code does not need to run.* The autograder will only check that it
matches the `.expected` files character-by-character (ignoring whitespace,
comments, etc.).

See [running.md](./running.md) for instructions on how to actually run the C
code. Ask for help on Discord.

#### Hints


**Feb 25:** `rgba[,]` has a built-in type: `_a2_rgba`. Use this built-in type instead of generating one.

* Truncate floats when printing. Example: print `3.999` as `3.0`.
  - **Contact the instructors if you have issues printing floats** we may relax the autograder.
* Generate `typedef struct` code for every struct command and every array
  literal in the order that they appear in the JPL program.
* Keep a counter per generated function (including `jpl_main`) for variable names, generate names in order: `_0`, `_1`, `_2`, ...
* Convert JPL types to C types as follows:
  - `int` -> `int64_t`
  - `float` -> `double`
  - `bool` -> `bool`
  - `void` -> custom struct (see header in any `.expected` file)
  - array -> custom struct with ...
    + one int field `di` per dimension
    + one pointer field `data` for the elements
    + a name of the form `_aRANK_TYPE` where `RANK` is an integer rank and `TYPE` is the C type of its elements (with any whitespace replaced by an underscore)
    + Example: `int[,]` -> `typedef struct { int64_t d1 ; int64_t d2 ; int64_t *data ; } _a1_long_long;`
  - struct -> struct name (which will be defined by an earlier struct command)

Use the helper functions provided by `runtime.c` in your output, such as `jpl_alloc`.

`show` needs a type string as its first argument. For everything except structs, use the
type S-expression printer you wrote for HW6/HW7. For structs, you must **print a tuple type** with
the element types inlined.

Example:

```
struct taco {
  shell: int
  ingredients: int[]
}
show taco{2, [4,5]}
```

Output:

```
...
show("(TupleType (IntType) (ArrayType (IntType) 1))", &_6);
```

`assert` must compile to an `if` that tests for nonzero input and either falls through to an
assertion or jumps forward to safety.

Example:

```
assert 1 == 2, "stop"
```

Output:

```
...
bool _2 = _0 == _1;
if (0 != _2)
goto _jump1;
fail_assertion("stop");
_jump1:;
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

This assignment is due Friday ~Feb 28~ March 7.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 90%    | Part 1   |
| 10%    | Part 2   |


#### Extra Credit

For a 5% extra credit, modify your compiler and `runtime.c` so that its show
function can print using a `(StructType t)` as input. It should work for any
user-defined struct. Create a diff showing your changes to `runtime.c` and
create a small but rigorous example program to demonstrate how your solution
works. Email the diff, the example JPL program, and the C output for that program
to Ben and Bhargav. (Then undo the changes to your compiler so you can pass the
autograder again!) They will grade the attempt on a pass/fail basis. You have
one shot at the 5% bonus. Everyone can earn the bonus; it is not a race.


