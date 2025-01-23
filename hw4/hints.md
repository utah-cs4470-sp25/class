## Hint: Handling sequences

Many rules in this assignment's grammar rely on sequences. Keep in
mind the following rules for sequences in JPL:

- Every place in the grammar with a sequence allows for an empty
  sequence. Make sure you handle cases like empty array literals `[]`
  and empty indexing expressions `a[]`. Some of these are non-sensical
  (like empty indexing expressions), but those will be rejected by
  the type checker, not the parser.
- JPL does not allow terminal commas, so for example `[1, ]` is not a
  valid array literal.
- In JPL the delimiters are always paired, so for example `[1, 2}` is
  not a valid array or tuple literal.

You should write helper functions for parts
of the grammar that repeat in several places. For example, you may
want a helper function for sequences of expressions (shared between
calls, array indexing, and tuple and array literals) and for sequences
of `<variable> : <expr>` (shared between array and sum loops). We've
designed JPL to somewhat encourage factoring out common grammatical
features.

There is a downside to having too many helper functions, though.
More helpers may mean your parse looks less similar to the grammar.
Also, helper functions that might match zero tokens make it harder
to avoid (or track down) infinite loops.

In your S-expression output, sequences typically just mean an AST node
has a varying number of children. For example, this struct declaration:

```
struct pair {
  x: int
  y: int
}
```

should parse and print to the following S-expression:

```
(StructCmd pair x (IntType) y (IntType))
```

This longer struct declaration:

```
struct point {
  x: int
  y: int
  z: int
}
```

should print an S-expression with two more children:

```
(StructCmd point x (IntType) y (IntType) z (IntType))
```

In function definitions (`fn` commands) function arguments are
enclosed in an additional set of parentheses. For example, this function:

    fn f(i : int, j : int) : int {
        return [i, j] 
    }

has the following S-expression representation:

```
(FnCmd
 f
 (((VarLValue i) (IntType)
   (VarLValue j) (IntType)))
 (IntType)
 (ReturnStmt
   (ArrayLiteralExpr (VarExpr i) (VarExpr j))))
```

A function with zero arguments still needs two sets of parentheses
in the argument position of its S-expression:

    fn g() : int {
        return 2
    }

```
(FnCmd g (()) (VoidType) (ReturnStmt (IntExpr 2)))
```


## Avoiding Infinite Loops

The grammar given above contains "left recursion"; that is, in a rule
like `expr : <expr> . <variable>`, the first step to matching an
expression seems to be recursively matching an expression, which would
lead to an infinite loop.

Naturally, your parser must not enter infinite loops; it must
terminate on all inputs. To handle cases like these, you will need to
_first_ parse an expression without a dot, and _then_ check if
the next token is a curly brace (and similarly for other forms of
indexing), using the technique described in class of splitting a
production into a head production and a continuation production.

Make sure expressions such as `a.b.c` parse---this expression refers
to a struct `a` whose `b` field is itself a struct whose `c` field
element is then retrieved. Similarly make sure mixed postfix operators
like `a(0)[0].x` work---this expression refers to a function `a`
which is called with an argument `0` and returns an array whose `0`-th
element is a tuple, whose `0`-th element is then retrieved.

One way to avoid infinite loops is to draw out a call graph for
all of the methods in your code and, for each call from function `f`
to function `g`, determine whether it's possible that the index is the
same. Edges in the call graph where the index can be the same are dangerous.

- To parse array literals, `parse_arrayliteralexpr` will first expect
  an `LSQUARE` token and then call `parse_expr` to parse an expression.
  In this case, since `parse_arrayliteralexpr` incremented the index to
  advance through the `LSQURE` token, the `parse_arrayliteralexpr` to
  `parse_expr` call can't use the same index, so this edge is safe.
- To parse literals, `parse_expr` will first call `parse_literalexpr`.
  The edge from `parse_expr` to `parse_literalexpr` does not increment
  the index, so this edge is dangerous.

If the bad edges form a loop, then it's possible for your code to go
in an infinite loop. If the bad edges do _not_ form a loop, than every
chain of calls will eventually increment the index to the end of the
list of tokens and stop.


