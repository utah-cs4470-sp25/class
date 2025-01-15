## Hint: Defining AST classes

While internal implementation details of your compiler are in principle up to
you, we recommend defining your AST classes in a uniform way.

We recommend you define a top-level class called `ASTNode` or similar,
containing one field, the start position of that AST node in the
source file (as a byte position). You can set this field when you
create any AST node, based on the start position of the first token in
this AST node. This start position will be useful for producing good
error messages throughout the rest of this compiler. If your lexer
doesn't store byte positions, it's probably worth going back and doing
that.

Then, for each grammatical class---here, `cmd`, `expr`, `type`, `argument`, and
`lvalue`---define a superclass with no fields. Name them something uniform like
`Cmd`, `Expr`, and so on. (In HW4, you will add `stmt` for statements.)

Finally, for each "production"---that is, every possible rule in the
grammar---define a class. Name it after both the grammatical class and
the production itself. So for example, you might have `WriteCmd` and
`IntExpr` classes.

For each production class, give it fields for all of its non-trivial
tokens. For example, the `WriteCmd` should have a `string` field and
an `Expr` field, while an `IntegerExpr` should have an `long` field.


## Hint: Writing the parser

We recommend defining a function for each grammatical class and a
function for each production. So, for example, you'd implement a
`parse_expr` method as well as `parse_intexpr`, `parse_floatexpr`, and
so on. These can take an index into the token list as their input, and
return an AST node and a new index as their output.

We recommend _not_ doing any arithmetic on the index directly in these
methods. It is too easy to make a mistake---forget to increment the
index at some point, for example---that will be hard to debug.
Instead, we recommend writing three helper methods: `peek_token`,
which returns type of the next token, `start_token` returns the start
position for the current token, and `expect_token` with the following
type signature:

```
public class Parser {
    public String expect_token(int, TokenType)
        throws ParserError { ... }
}
```

The two arguments are an index into the token list and a token type to
expect that token to be. If the token at that position doesn't have
that type, `expect_token` raises a `ParserError` error. If it does
have that type, `expect_token` returns its contents, which will be the
variable name or the integer value or whatever.

You can then call `expect_token(i++, type)` in any parser function to
assert that the next token has a certain type and simultaneously to
increment `i`. (When you do this, put each `expect_token` on its own
line, so you don't get argument evaluation order issues.)

Each grammatical class function should call `peek_token` and then
dispatch to the appropriate production function (or raise an error).
Each production function should call `expect_token` and grammatical
class functions. We recommend only creating an AST node object at the
end of a production function, after you've successfully checked that
you've got all the right tokens.

**Use of a parser generator is prohibited.** In this class you are required to
write the parser by hand. This is more common than not in complex production
compilers (generally because parser generators produce poor error messages) and
also because we believe writing a parser by hand is important to understanding
parsing. The graders enforce rules like this with spot checks, so if you do use
a parser generator we may discover it and have to revise your grade to a 0. Do
not do it.


## Hint: Arguments and LValues

Even though `argument` and `lvalue` have only have one production each
in the subset of JPL that we are considering in this assignment, we
still recommend you define all of the associated AST node classes and
parsing functions. It'll seem annoying now, but will save you time in
later parsing assignments, which are generally harder.

Your output S-expression must contain `ArgLValue` and `VarArgument`
nodes.


## Hint: Integers and floats

When you parse an integer literal expression or a float literal
expression, you need to check JPL's [rules about numeric
constants][jpl-num]. Specifically, you will need to:

 - Make sure integer literals fit into 64-bit signed twos-complement
   integers
 - Make sure float literals fit into 64-bit double-precision IEEE-754
   floats without being infinite or `NaN`
 - Convert the string representing that integer or float into your
   language's 64-bit integer or float type.

*Use your language's built in integer and float parser for these
tasks.* Specifically, use:

 - `Long.parseLong` and `Double.parseDouble` in Java, catching
   `NumberFormatException`s to detect integer overflow and using
   `Double.isInfinite` and `Double.isNaN` to detect bad floating-point
   values.
 - `strtol` and `strtod` in C++, checking `errno` for `ERANGE` to
   detect overflow and using `isnan` in `math.h` to detect NaNs.
 - `int` and `float` in Python, catching exceptions and using
   `math.isnan` and `math.isinf` to detect bad floating-point values.
   Because integers in Python are arbitrary-size, to detect integer
   literals that are too big, you must manually compare them against
   `2 << 63 - 1` and `-2 << 63` to detect overflow.

We want to emphasize again that converting strings to numeric
values---especially converting strings to floating-point values---is
way harder than you think it is, and you should definitely not do this
on your own, because you are extremely likely to get it wrong!


