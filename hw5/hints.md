## AST Recommendations

`UnopExpr` covers two different operators, while `BinopExpr`
covers thirteen different operators. For these AST classes, the
S-expression syntax includes the operator, such as:

    (UnopExpr ! (BinopExpr (VarExpr a) < (VarExpr b)))

We recommend creating a `UnopExpr` and `BinopExpr` class internally to
represent these types of expressions, with the operator itself stored
as enumerated types `Binop` and `Unop`. This is because the handling
for these operators is very similar throughout the rest of the
compiler, so giving them the same AST nodes will simplify your
implementation of future assignments.

Additionally, in the `ArrayLoopExpr` and `SumLoopExpr` AST classes,
the S-expression output contains a sequence of stand-alone variable
names (representing the `<variable>` tokens in the sequence of loop
bounds) and expressions (representing the `<expr>` phrase in the
sequence of loop bounds), followed by one final expression for the
loop body. That is,

    sum[i : H, j : W] f(i, j)

has the following S-expression representation:

    (SumLoopExpr i (VarExpr H) j (VarExpr W)
     (CallExpr (VarExpr i) (VarExpr j)))

Note that in the `SumLoopExpr`, the `i` and `j` are `<variable>`s, so
they are just printed directly, but inside the loop body, the `i` and
`j` are `<expr>`s, so they become `(VarExpr i)` and `(VarExpr j)`.

We recommend internally representing the bounds in a `SumLoopExpr`s or
`ArrayLoopExpr`s as a vector of string/expression pairs. You might
make both of these classes a subclass of a `LoopExpr` superclass,
because you'll have somewhat similar logic for `array` and `sum`
loops.

That said, it is ultimately up to you how your AST classes are
represented internally, as long as they print correctly.


## Handling precedence

JPL has seven levels of precedence; until now, your implementation
only had one (for postfix indexing expressions) or perhaps two (as you
may have separated out literal expressions in your implementation).

To handle precedence, you'll need to split the grammar given above
into having multiple levels of expressions---seven or perhaps eight
depending on how you do it. We recommend giving them memorable names
like `expr_comparison`, `expr_additive`, `expr_unop`, and similar.

Writing a disambiguated grammar is tricky, and you may not get it
right on the first try. We recommend you write out the disambiguated
grammar before you start implementing anything, and test it on various
tricky expressions. For example, consider expressions like:

    1 + 2 + 3
    1 + 2 * 3
    array[] x + 1

and try to check if your grammar can accept the "wrong" parse. You do
this by writing out the "wrong" parse tree and checking if each phrase
matches a rule from your grammar. For example, this "wrong" parse:

    (array[] x) + 1

does match the grammar given at the top of this assignment because:

    x is an <expr> via "expr : <variable>"
    (array[] x) is an <expr> via "expr : array [ ... ] <expr>"
    (array[] x) + 1 is an <expr> via "expr : <expr> + <expr>"

This tells you that the grammar at the top of this assignment is
incorrect and needs further disambiguation.

Make sure to test the "right" parse trees too, because it is easy to
accidentally write a grammar that rejects too much. For example,
consider this expression:

    -sum[i : 50] i

this is unambiguous and only has one parse tree, even with the
ambiguous grammar at the top of this assignment. Your disambiguated
grammar must therefore continue to parse it, and it's easy to
accidentally write a grammar that rejects it. Likewise, check nested
`if` expressions and combinations of `if` and `array` to make sure
they are still parsed by your disambiguated grammar.

You are done disambiguating when your grammar rejects all the "wrong"
parse trees but still accepts the "right" parse trees.


