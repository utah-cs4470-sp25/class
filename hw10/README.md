Assignment 10: Assembly Expressions
===================================

Compile an expression-heavy subset of JPL to x86_64 assembly.

```
cmd : show <expr>

expr : <integer>
     | <float>
     | true
     | false

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

expr : [ <expr> , ... ]
```

The expressions are roughly grouped from easiest to hardest.

*Your assembly output does not need to run.* The autograder will only check that it
matches the `.expected` files character-by-character (ignoring whitespace,
comments, etc.).

Still, you should try to run the output.
See [assembly.md](../assembly.md) for instructions on how to actually run the output.
Ask for help on Discord.


#### Hints

Short hints:

* The autograder tries to normalize floats. Print floats in the natural way for
  your language and hopefully the tests will pass.
* (*March 21*) By popular demand we have fixed a bug in the staff compiler output
  for % on floats. It now aligns the stack before `call _fmod` similar to how
  calls to `_pow` look. (If you submitted HW10 before this change you do not need
  to resubmit, we will use your past action score. But you will need to update
  % ouput for HW11 and beyond.)
* 

More hints: [hints.md](./hints.md)


# Testing

We will run your compiler with the `-s` flag:

    make run TEST=/grader/ok/001.jpl FLAGS=-s

The grader directory `ok` (Part 1) contains valid hand-generated JPL programs.
The directories `ok-fuzzer1` (Part 2) and `ok-fuzzer2` (Part 3) contain valid
fuzzer-generated programs.

You can run these tests on your computer by downloading the
auto-grader and running it like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw10


# Submission and grading

This assignment is due Friday March 21.

We are happy to discuss problems and solutions with you online, in office
hours, or by appointment.

| Weight | Function |
|--------|----------|
| 80%    | Part 1   |
| 15%    | Part 2   |
|  5%    | Part 3   |


