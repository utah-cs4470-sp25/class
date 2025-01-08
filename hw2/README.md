Assignment 2: Writing the Lexer
===============================

Your first assignment is to build a **lexer** for JPL.
(Also known as a **lexical analyzer** or **tokenizer**.)

The lexer turns source code into either:

- a list of tokens
- or an error.

You will be tested on a collection of lexer test cases created by the
instructors.


# Your Assignment

The lexical syntax of JPL is in the [JPL Specification][lex-spec]. The
full list of tokens you should support is:

[lex-spec]: https://github.com/utah-cs4470-sp25/class/blob/main/spec.md#lexical-syntax

```
ARRAY
ASSERT
BOOL
COLON
COMMA
ELSE
END_OF_FILE
EQUALS
FALSE
FLOAT
FLOATVAL
FN
IF
IMAGE
INT
INTVAL
LCURLY
LET
LPAREN
LSQUARE
NEWLINE
OP
PRINT
RCURLY
READ
RETURN
RPAREN
RSQUARE
SHOW
STRING
STRUCT
SUM
THEN
TIME
TO
TRUE
VARIABLE
VOID
WRITE
```

Note that some tokens, like `ARRAY` or `LCURLY`, correspond to only one
possible string, while other tokens are non-trivial and correspond to
multiple possible strings (`INTVAL`, `OP`). Also note that `OP` is a
single token type covering all of the various arithmetic and boolean
operators, while punctuation characters such as `:` get their own token
type. This is done to simplify parsing somewhat.

Your lexer must read in a JPL file and output the sequence of tokens
that it contains. When lexing is successful, a special `END_OF_FILE`
token should be the last element of the list. It does not correspond
to any actual text in the input file, but rather serves as a sentinel
so that your parser will not have to keep checking for walking off the
end of the list of tokens.

Start to implement the [JPL command line interface][iface],
specifically the "lex-only" mode triggered by the `-l` command line
flag. With this flag, your compiler should print the lexed tokens, one
per line, in a format like this:

```
    FN 'fn'
    VARIABLE 'inc'
    LPAREN '('
    VARIABLE 'n'
    COLON ':'
    INT 'int'
    RPAREN ')'
    COLON ':'
    INT 'int'
    LCURLY '{'
    NEWLINE
    RETURN 'return'
    INTVAL '1'
    OP '+'
    VARIABLE 'n'
    NEWLINE
    RCURLY '}'
    NEWLINE
    PRINT 'print'
    STRING '"hello!"'
    NEWLINE
    SHOW 'show'
    VARIABLE 'inc'
    LPAREN '('
    INTVAL '33'
    RPAREN ')'
    NEWLINE
    END_OF_FILE
    Compilation succeeded: lexical analysis complete
```

Each token prints its name and its contents, except for `NEWLINE` and
`END_OF_FILE` tokens. Since your code will be auto-graded, you must
match this output format exactly. Make sure not to add an extra space
or newline after the `NEWLINE` and `END_OF_FILE` tokens; that is,
print those tokens as `"NEWLINE"`, not `"NEWLINE "`.

Your compiler must print at least `Compilation succeeded` at the end.
It must print at least `Compilation failed` when a lexer error occurs.
See the specification for further details.
(The autograder will check for these strings.)

You do not need to support the other command line flags at this time, only `-l`.

[iface]: https://github.com/utah-cs4470-sp25/class/blob/main/spec.md#jpl-compiler-command-line-interface

**Important note:** Do not try to enforce tricky properties,
such as integers or float values being out of range, in the lexer.
Enforce these properties during parsing.


# Handling Whitespace

Make sure to handle white space correctly:

 - Space characters can separate tokens.

 - Space characters do not generate tokens of their own.
 
 - Newlines do turn into tokens, and consecutive newlines in the input
   should be collapsed into a single `NEWLINE` token, even if
   separated by comments or whitespace.
   * Your lexer should never produce multiple consecutive `NEWLINE` tokens.

 - Comments do not turn into tokens.
   * One-line comments do not include the terminating newline character.
   * Multi-line comments can include newlines inside.
   * Be particularly careful to avoid emitting multiple consecutive `NEWLINE`
     tokens in the presence of comments in the input.

 - Line continuations (backslash at the end of a line) do not produce a token.
   The newline that follows a line continuation must not produce a token.

 - All other whitespace characters are illegal, and you must produce a
   lexer error upon encountering one in a file.
   * Be extra careful that you are saving files with Unix-style
     line endings (LF). Windows-style line endings (CR LF) are invalid
     in JPL.


# Implementation Hints

The list of tokens should use a suitable list or array data structure
in your compiler implementation language. For example, in C++ it might be:

```
std::vector<token> tokens;
```

where a `token` might be:

```
struct token {
  tok_type t;
  int start;
  std::string text;
};
```

(In class, we discussed a different style using Python classes and
subclasses.)

> Note that the `start` field is not required in this assignment, but is
> essential to producing good error messages. Since the only user of
> your compiler is you, good error messages are an investment worth
> making. We recommend saving the name and contents of input file in
> memory, from which you can reconstruct the line and column number from
> any byte position.

You may use regular expressions to define complex tokens such as `INTVAL`,
`FLOATVAL`, `VARIABLE`, and `STRING`, as well as whitespace.
Regular expressions are not required. Regular expressions are not
particularly recommended. Use them sparingly if you do.

Rigorously think through both what strings a token should match as
well as which tokens it should *not* match. For example, make sure
`1.0.0` and `.` are not considered valid `FLOATVAL`s! (Though `1.0.0`
_is_ two valid `FLOATVAL`s in a row!) In some cases, you can use order
to help; for example, instead of writing a `VARIABLE` regular
expression that excludes all keywords, it may be easier to match
keywords first, and only match variables if that fails.


# Testing your code

The staff JPL compiler supports the `-l` option.

```
    ~/Downloads $ ./jplc-macos -l ~/jpl/examples/gradient.jpl
    NEWLINE
    FN 'fn'
    VARIABLE 'gradient'
    LPAREN '('
    VARIABLE 'i'
    COLON ':'
    INT 'int'
    COMMA ','
    VARIABLE 'j'
    COLON ':'
    ...
```

For large programs, it can be tedious to compare the list of tokens
by hand. Instead, save each output to a file and use `diff` to
compare:

    diff wrong-output.txt right-output.txt

This will print all the lines that differ.

Once things are working, push everything to your repository. Make sure
you can run your compiler like so:

```
    make run TEST=input.jpl
```

Here `input.jpl` is a file that we will supply. *Your makefile is
responsible* for passing the `-l` flag to your compiler. Additionally,
the `make compile` command must complete successfully.

You can find the tests and expected outputs [in the auto-grader
repository](https://github.com/utah-cs4470-sp25/grader/tree/main/hw2).
The auto-grader uses these tests in three different ways.

- Part 1 asks your compiler to lex each file in `test-lexer1` and
  compares your compiler's output to the reference output from our JPL compiler.
  There are 180 tests, and your grade on this portion is the number of these
  tests that you pass.

- Part 2 asks your compiler to lex each file in `test-lexer2` and
  verifies that your compiler successfully lexes each file. There are 222
  files, and your grade on this portion is the number of these tests
  that you pass.

- Part 3 asks your compiler to lex each file in `test-lexer3` and
  verifies that your compiler issues a lexing error. There are 546
  files, and your grade on this portion is the number of these tests
  that you pass. A lot of these tests are pretty repetitive, focusing
  mostly on invalid characters.

You can run these tests on your computer by downloading the
auto-grader and running:

```
    make -C <auto-grader directory> DIR=<compiler directory> test-hw2
```

Part 1 is the hardest. Do it first! Go in order, test by test.


# Submission and grading

Submit a link to your preferred GitHub commit on Canvas.

Your solutions will be auto-graded. The auto-grader will use Github
Actions and runs on Ubuntu using the tests described above.

Your compiler must be runnable as described in the [Testing your code]
section. If the auto-grader cannot run your code, you will not receive
credit. The auto-grader output is available to you at any time, as
many times as you want. Make use of it.

The weight assigned to each part is:

| Weight | Function |
|--------|----------|
| 70%    | Part 1   |
| 15%    | Part 2   |
| 15%    | Part 3   |

