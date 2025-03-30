
# Peephole Optimization: Extra Challenges

#### 1

Replace a sequence of RSP changes with a single one. For example, you might
fold this:

    add rsp, 8
    add rsp, 8
    add rsp, 24
    sub rsp, 24

into this:

    add rsp, 16

We expect speedups on the order of 5% on code that does a lot of array
indexing or function calls. Other code should see roughly 1% speedups.


#### 2

Detect the pattern `x % 2 == 0` and optimize that specifically into something
like `! (x & 1)`.

Real compilers have hundreds of patterns like this that address common
code patterns.


#### 3

Replace integer division and modulus by powers of two with shifts right and
bitwise ands.

What's hard about this is that integer division rounds *to zero* while shifts
right rounds *down*. For negative numbers, they round in opposite directions.
You can do some bit tricks to correct for this. Adding the tricks is totally
worth the effort because divison and modulus are very slow (20-30 cycles!).

If you try this, be very careful to test negative numbers, powers of two,
`INT_MIN` and `0`.


#### 4

Extend the array indexing optimization from HW13:

- Opimize dot access like `i.x`,
- nested dots like `i.x.z`, and
- dots into an array index like `a[10, 12].posn`.

All of these operations move data in and out of containers. Instead of copying
data to the stack, we are better off tracking where it actually is and
referencing that position directly.

One way to start: modify each of your
`codegen_xxx_expr` functions to return the amount of bytes allocated and
the offset from `rsp` where the result is located. All
existing functions should return the size of the return type as the
number of bytes allocated and `0` as the offset from RSP. Local
variable references can then output no instructions and just return a new
offset from RSP, and dot expressions can adjust the offset instead of
outputing instructions.

In general, this optimization can be taken very far; in LLVM, the SROA pass
does something like this.

- <https://blog.regehr.org/archives/1603> (search for "SROA")
- <https://llvm.org/docs/Passes.html#sroa-scalar-replacement-of-aggregates>


# Loop Permutation: Extra challenges

It is not too hard to make our notion of tensor contraction less
restrictive.


#### 1

First of all, you can allow more complicated mathematical expressions
inside array indices. If you see an index like `A[i + j, k]`, that
should create traversal edges from both `i` and `j` to `k`. You'd need
to compute the variables used in an expression, and then add edges
from all variables used by one index to all variables used by all
later indices.


#### 2

You can also allow more JPL expressions inside the tensor contraction
body. Unary operators, tuple indices, and so on are easy to add
support for. More difficult is nested indices like `A[i][j]`. Here,
you need to add traversal edges from `i` to `j`, because you want `j`
to change more rapidly than the array it indexes into, `A[i]`. The
general rule is to add edges from any variables used to compute the
array to all of the indices into that array.


#### 3

You can also support the case where the `sum` loop bounds depend on
the `array` loop indices. The trick is that, for any `sum` loop bound,
you need to add edges to that `sum` loop variable *from* any `array`
loop variable that the bound depends on, and you'll need to generate
code to recompute the `sum` loop bounds when any of those `array`
variables changes. This is a lot of work, but conceptually not too
complex.


#### 4

You can support some cases where the `sum` loop isn't directly below
the `array` loop. For example, consider this snippet of a neural
network implementation:

    array[i : W, j : H] relu(sum[i2 : W, j2 : H] input[i2, j2] * weights[i2, j2, i, j])

Here, between the `array` and `sum` loop there is some other
expression, a call to `relu`. One could work around this, like so:

    let a = array[i : W, j : H] sum[i2 : W, j2 : H] input[i2, j2] * weights[i2, j2, i, j]
    ... array[i : W, j : H] relu(a[i, j])

There's also a cleverer thing you can do if the input and output of
this function are the same size in bytes. You can actually store the
running sum in the `array`, and then only call `relu` on the last
iteration of the last `sum` index. This tweak is a huge amount of work
and pretty conceptually tricky as well.

