## Representing environments

Don't try to typecheck using one global table. You will need to add and remove
bindings as the typechecker enters and leaves functions, let blocks, and
comprehensions. Managing this environment will be hard.

Instead, create a new class, Environment (or Context, or SymbolTable)
that contains two members: a reference to a parent environment
(which will be uninitialized or empty for the root table)
and a map from strings to name information. Names refer to either struct
declarations or variable definitions.

We recommend defining three helper functions on environments:

* `add` (extend the inner-most environment in the chain)
* `get` (check inner-most and parents for a symbol)
* `has`

We also recommend adding an easy way to construct a new child of a given
environment.


## Handling variable definitions and uses

We recommend starting with variable definitions and uses, focusing on
the following new JPL constructs:

```
expr : <variable>

cmd : let <lvalue> = <expr>
    | read image <string> to <argument>

lvalue : <argument>

argument : <variable>
```

When handling a `let` command, first typecheck the binding expression.
Add the bound variable to the environment (raise an exception if
the name is already bound) before typechecking the body.

When handling a `read` command, the value being read always has the
same type, `rgba[,]`.


## Array and sum comprehensions

Once `let` and `read` are under control, extend the grammar to handle
loops:

```
expr : array [ <variable> : <expr> , ... ] <expr>
     | sum [ <variable> : <expr> , ... ] <expr>
```

When type checking `array` and `sum` loops, make sure to type check
the _body_ of the loop using a new environment. That new environment
should be the child of the current environment, with all of the
variables added to it, mapping to integer types.

You must enforce all of the other type rules for `array` and `sum`.
For example, you must ensure that all the loop bounds evaluate to
integers, that the body of a `sum` expression has a numeric type, and
you must return an array type of the proper size from an `array`
expression.

If you do it right, after you add type checking for `array` and `sum`,
you won't need to modify your `VarExpr` handling at all.


## Handling complex arguments

Next, we recommend adding support for lvalues:

```
lvalue : <variable>
       | <variable> [ <variable> , ... ]
```

This makes your handling of `let` more complex, because it is now
possible to bind multiple variables at the same time, as in:

    let a[W] = [1, 2, 3]

This expression binds `a` to the array `[1, 2, 3]` and `W` to that
array's length (`3`).

We recommend adding helper methods `add_argument` and `add_lvalue` to
the environment API. These helper methods take an lvalue and a type
and update the environment accordingly.
These methods may fail with a type error if the lvalue and type do
not match.

Make sure to test failure cases.


## Handling function calls

Next, add support for function _calls_. To do that, you will need to
store `FunctionInfo` in your symbol tables.

A `FunctionInfo` should store argument types, a return type, and an
environment. All types should be resolved. Build the environment
by typechecking the function (more on this below) and collecting bound
variables along the way; this environment will be useful later when we write
the compiler back-end.

When type checking a function call:

- Type check all subexpressions (the function arguments)
- Look up the function name
- Check that it is, in fact, a function
- Check that the right number of arguments have been passed
- Check that each subexpressions's type is equal to the corresponding
  argument type
- Return the function's return type

Make sure to include each of these checks!

To test function calls, initialize your global symbol table with all
of the built-in JPL functions like `sin` or `to_int`. Test calls to
these functions.


## Handling function definitions

Now move on to type checking function definitions.

To type-check a function definition, you need to:

- Extract the `LValue` and `Type` from each `Binding`
- Resolve the argument and return types
- Create a child environment, which will be used for function scope
- Add each `LValue` to the child symbol table, with its corresponding `ResolvedType`
- Construct a `FunctionInfo` for the function itself
- **Add that `FunctionInfo` to the top level environment** (do it now, because
  the function might be recursive)
- Type check each statement inside the function

For type checking statements, you will need to write a new `type_stmt`
function. The actual type checking should be pretty straightforward,
since you've already implemented type checking for `let` and `assert`
commands, which work the same as `let` and `assert` statements.
Make sure to return an environment.

Typing the `return` statement needs some special care, because for a
`return` statement to be valid, it must return a value of the same
type as the function's declared return type. Therefore, add an
argument to `type_stmt` for the function's declared return type,
and use that when type checking `return` statments.

Only functions that return `void` do not need return statements.

Test that recursive function calls work, as in:

```
fn plus(x : int, y : int) : int {
    return if x == 0 then y else plus(x - 1, y + 1)
}
```

Recursive function _are_ legal in JPL, but _mutually recursive_ functions are
not.

