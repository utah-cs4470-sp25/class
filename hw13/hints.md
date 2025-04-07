# Organizing your code

All of the parts of this assignment require you to modify the code
generation phase in your compiler. We recommend adding an
`optimization_level` field to your `Assembly` object, initialized in the
constructor based on command-line arguments, and then having the various
`codegen_xxx` function check that field to determine whether to enable these
optimizations.

Broadly, peephole optimizations will look like this:

    class Function:
        def codegen_int_expr(expr : Expr) -> list[str]:
            if self.assembly.optimization_level >= 1:
                # generate optimized assembly
                return
            # generate unoptimized assembly

In the simpler peephole optimizations, the `if` statement at the top
is the only thing you are adding. In more complex ones, you may need
to have more complex logic---but whatever logic you add, you should
keep the un-optimized form of the code unchanged from what you have now.


