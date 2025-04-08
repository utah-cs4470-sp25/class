Assignment 13: Peephole Optimization
====================================

Add the following optimizations to your compiler:

 - P1. Replace short constants with immediates (`ok1`)
 - P2. Simplify boolean-to-integer casts (`ok2`)
 - P3. Faster index expressions (`ok3`)
 - P4. Multiplications by powers of 2 as shifts (`ok4`)
 - P5. Reducing array copies for indexing (`ok5`)

**April 8**: the 5 parts of this assignment are NOT independent. Parts 3 and 4
depend on one another. There may be other dependencies.


# Optimization list

> The following descriptions assume that your compiler has functions named
> `cg_X` for every AST node `X`, where `cg` is short for `codegen`. If your
> code is not organized that way, that's fine, but you will need to figure out
> the right way to modify your codebase.


## P1. Replace short constants with immediates

Without optimizations, a constant expression like `17` generates the
following assembly code:

    mov rax, [rel CONST]
    push rax

This is inefficient because it involves a read from memory.
The memory may not even be in cache. For small constants, it is
more efficient to generate this assembly:

    push qword 17

Here the `qword` annotation is mandatory, because the assembler needs
to know whether you intended to push a 64-bit, 32-bit, or 16-bit value.

On x86_64, only 32-bit values work as immediates. So, for
example, the following are legal in assembly:

    push qword 2147483647
    push qword -2147483648

These values are the largest and smallest signed 32-bit integers.

However, the following is invalid:

    push qword 2147483648

This is invalid because this number does not fit in 32 bits.

Modify your `cg_int_expr` and `cg_bool_expr` functions to use
immediates for constants that fit in 32 bits. If the constant does not
fit in 32 bits, continue to use the slower load-from-constant code. In
many languages, you can test if a value fits into 32 bits like this
(_make sure this computation happens with 64-bit integers):

    x & ((1 << 31) - 1) == x

With optimizations on, the data portion of your assembly file should
not have any integer constants that fit into 32 bits.


## P2. Simplify boolean-to-integer casts

In JPL, booleans and integers are separate types. To convert a boolean into an
integer, you might write the following code:

    if b then 1 else 0

Without optimizations, this results in the following assembly code:

    ; Assume b is on the stack
    pop rax
    cmp rax, 0
    je .ELSE
    mov rax, [rel CONST1]
    push rax
    jmp .END
    .ELSE:
    mov rax, [rel CONST0]
    push rax
    .END:

None of this code is necessary, because the representation of a
boolean is already a 64-bit integer containing either 1 (for true) or
0 (for false). Add code to `cg_if_expr` to recognize the specific
pattern above and replace it with just the computation of `b`.

The result is much less code, fewer reads from the data section, and fewer
jumps (which avoids polluting the CPU's branch prediction register).


## P3. Faster index expressions

In array index and array loop expressions, we need to compute a
pointer to the array entry. For example, in this `array` loop:

    array[i : 1024, j : 512, k : 256] body

We need to compute a pointer into the array being constructed to store
the body into it. That assembly code looks like this:

    mov rax, 0
    imul rax, [rsp + OFFSET_1024]
    add rax, [rsp + OFFSET_i]
    imul rax, [rsp + OFFSET_512]
    add rax, [rsp + OFFSET_j]
    imul rax, [rsp + OFFSET_256]
    add rax, [rsp + OFFSET_k]
    imul rax, SIZE
    add rax, [rsp + OFFSET_ptr]

Some of this code is wasteful. First of all, the first three
instructions set `rax` to zero, then multiply it by something
(resulting in 0), and then add the value at `[rsp + OFFSET2]`.
Instead, we can simply load that value directly:

    mov rax, [rsp + OFFSET_i]
    imul rax, [rsp + OFFSET_512]
    add rax, [rsp + OFFSET_j]
    imul rax, [rsp + OFFSET_256]
    add rax, [rsp + OFFSET_k]
    imul rax, SIZE
    add rax, [rsp + OFFSET_ptr]

The value here is avoiding an `imul`, which can contend on the
multiplication port.

Make this change to in both `cg_array_index_expr` and `cg_array_loop_expr`. If
you factored the array indexing logic into a helper method, you may only need
to change that one method.

Moreover, in `array` loop examples like above, the memory argument to
the `imul` expression is fixed---it's 1024, 512, and 256. We can use
that to simplify the code further:

    mov rax, [rsp + OFFSET_i]
    imul rax, 512
    add rax, [rsp + OFFSET_j]
    imul rax, 256
    add rax, [rsp + OFFSET_k]
    imul rax, SIZE
    add rax, [rsp + OFFSET_ptr]

This reduces the use of address arithmetic ports, which can allow
these instructions to execute with more parallelism. Make sure to only
apply this optimization when the bound fits in 32 bits.

Implement this optimization in `cg_array_loop_expr`. Only apply this
optimization if the loop bounds are integer constants.


## P4. Multiplications by powers of 2

When you execute the JPL expression `x * 256`, this generates the
following assembly code:

    ; compute x
    mov rax, [rel CONST_256]
    push rax
    pop r10
    pop rax
    imul rax, r10
    push rax

But multiplications by powers of two can be represented with a shift:

    ; compute x
    pop rax
    shl rax, 8
    push rax

Note that `8` is the log-base-2 of 256. Make this optimization in
`cg_binop_expr` when the right-hand operand is an integer constant
that is also a power of two. In most languages you can test that an
integer `x` is a power of two with this code (using both logical
AND `&&` and bitwise AND `&`):

    x >= 0 && x & (x - 1) == 0

The following loop computes the log-base-2 of a number that is a power of two:

    int n;
    for (n = 0; (1 << n) < x; n++);

It's a good idea to put your is-power-of-two and log-base-2 codes into helper
functions.

Make sure to handle **all** multiplications that appear in generated assembly.
You can probably search your compiler for `imul` instructions to find
them all.

More notes:

 - Handle multiplying ith a power of two on the left or right. That
   is, both `x * 256` and `256 * x` should be optimized.

 - If you see something like `256 * 256`, handle it like `256 * x`.
   (A real compiler would constant-fold this.)

 - Optimize multiplications in the array indexing portion of an
   `array` loop if the bounds of the array are integer constants that
   are powers of two. For example, if `array[i : 256, j : 256] body`,
   make sure the array indexing logic shifts by 8 instead of
   multiplying by 256.

 - **Do not** optimize the multiplications when allocating
   the array in an `array` loop, because `shl` doesn't set the
   overflow flag in the expected way.

Probably the most important one of these is optimizing multiplications
in array indexing, because indexes appear in `array` loops.


## P5. Reducing array copies for indexing

When you write an expression like `a[10, 12]`, which is an array index
into a variable, the generated assembly goes through these steps.

 - Copy the array `a` to the top of the stack
 - Evaluate the indices in reverse order
 - Check that each index is in bounds
 - Compute the array index
 - Drop everything from the stack
 - Copy the array index to the stack

If the array is already in a local variable, the first step is
wasteful, because the array is already on the stack---it just might
not be at the top of the stack.

In assembly terms, the normal assembly for indexing into `a[10, 12]`
looks like this:

    mov rax, 0
    imul rax, [rsp + 0 + 24]
    add rax, [rsp + 0]
    imul rax, [rsp + 8 + 24]
    add rax, [rsp + 8]
    imul rax, [rsp + 16 + 24]
    add rax, [rsp + 16]
    imul rax, SIZE
    add rax, [rsp + 24 + 24]

Here, there are two kinds of address: `[rsp + (8 * K)]` indexes into
the indices, which are on top of the stack, while `[rsp + (8 * K) + 24]`
indexes into the array, which is just below the indices.

Modify your array indexing code to allow the array to be elsewhere on
the stack. To do so, compute a `GAP` value, which without
optimizations is 8 times the rank of the array, and generate this
assembly code:

    mov rax, 0
    imul rax, [rsp + 0 + GAP]
    add rax, [rsp + 0]
    imul rax, [rsp + 8 + GAP]
    add rax, [rsp + 8]
    imul rax, [rsp + 16 + GAP]
    add rax, [rsp + 16]
    imul rax, SIZE
    add rax, [rsp + 24 + GAP]

When optimizations are enabled, *if the array is a local variable*, do
not copy it to the top of the stack. Instead, set `GAP` to be the 8
times the rank of the array *plus* the difference between the array's
offset from `RBP` and the stack size at the beginning of
`cg_array_index_expr`. For example, if the current stack size is 48
bytes, and the array is at `RBP + 16`, meaning an offset of -16, and
the array has size 24 bytes, then the `GAP` should be the array offset
should be 48 - (-16) + 24 = 88.

When this optimization is enabled, since you don't copy the array to
the top of the stack, make sure you also don't free it! Also make sure
this optimization works together with all earlier optimizations, such
as the better array indexing and the multiplication-to-shifts
optimization.

The upshot of this optimization is that we read the array bounds and
pointer out of whereever the array is located on the stack, even if
it's not at the top of the stack.

> More detail on the `GAP` computation: Imagine an array of rank `R`. Without
> optimization, the array is copied to the top of the stack, and then `R`
> indices are pushed on top of it, meaning that the array is `[rsp + 8R]`.
> That's why the `GAP` is `8*R` without optimizations.
>
> However, suppose the array is at `[rbp - OFFSET]` instead. Well,
> `RBP = RSP + stack_size`, so the the array is at:
>
>     [rsp + (stack_size - OFFSET)]
>
> Then `R` indices are still pushed on top, so ultimately you get `GAP =
> stack_size - OFFSET + 8*R`.

The reason this optimization is important is that we expect a lot of important
code, like a matrix multiply, to index into arrays inside a tight loop.
Moreover, arrays can be big. Copying arrays is a waste.


# Testing your code

Your compiler must implement both the `-s` flag and the `-O1` flag.

We will run your compiler two ways:

1. With only `-s`, generate the same assembly code expected for previous assignments.
   ```
   make run TEST=/grader/ok/001.jpl FLAGS=-s
   ```

2. With `-s` and `-O1`, generate peephole-optimized assembly.
   ```
    make run TEST=/grader/ok/001.jpl FLAGS="-s -O1"
   ```

You can test on your computer by downloading the auto-grader and running it
like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw13


# Submission and grading

This assignment is due Friday April 11.

| Weight | Function     |
|--------|--------------|
| 15%    | Part 1       |
| 15%    | Part 2       |
| 20%    | Part 3       |
| 25%    | Part 4       |
| 25%    | Part 5       |

There is no fuzzer test.


