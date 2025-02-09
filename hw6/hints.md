* **Do not** modify your parser to restrict JPL programs to the
  HW6 subset. Use your normal parser. We promise to test with
  programs that fall within the subset.

* Define a new set of types for type trees, separate from your types for type
  AST nodes. Suggested names for the top-level class are: `Type` or `TypeValue`
  or `ResolvedType`. Subtypes of this class should cover integer types,
  array types, and struct types.

* Add a `TypeValue` field (or whatever you named it) to the `Expr` superclass
  of all expression AST nodes, whose value is uninitialized by default and
  later filled in by the type checker. (That said, you can do whatever you want
  in the AST internals as long as output prints correctly.)

* Define a helper method checks if two types are equal (perhaps by
  overriding the `==` method if you're in C++ or Python). This equality method
  should be recursive in the case of arrays.


## For C++ specifically

In C++, we recommend using `shared_ptr` in the type tree. This is
because it is convenient to store multiple pointers to the same part
of some type tree; for example, the type systems states than an
`ArrayIndexExpr` has the same type as the base expression's base type;
this involves having two pointers to the base type. Type trees should
never form cycles, so using `shared_ptr` in this context is safe. The
`shared_ptr` class will automatically handle de-allocation for you.

We recommend storing types on `Expr` nodes with a `type` field on
marked `mutable` (and declared as a `share_ptr<Ty>`). Marking the
field `mutable` ensures that it can be modified even through a `const`
pointer to the `Expr`, which is what we'll be doing in the `type_of`
method.

For traversing the AST, we recommend having the `type_of` method take
a standard pointer to a constant expression node, that is, a `const
Expr*`. While using a raw pointer is generally a bad idea, we should
neither be allocating, nor modifying, nor deallocating AST nodes in
the type checker. (Except to store types on the AST nodes, allowed by
marking that field `mutable`.) At the moment, there isn't a standard
idiom in C++ for "borrowed" ownership of the kind we need in our type
checker, so this is a reasonable solution.

