## Conditionals

A conditional expression is one where a subexpression may or may not
execute. In x86_64 assembly, conditionals are done in two steps.
First, some instruction, usually `cmp`, sets the RFLAGS register.
Then, a jump instruction uses the flag register to determine whether
or not to transfer control to a given label. For example, consider the
following code:

        cmp rax, 0
        je .else

        mov rax, 7
        jmp .end

    .else:
        mov rax, 12

    .end:

Here, the `cmp` instruction compares register RAX to zero and sets the
RFLAGS accordingly. The `je` instruction, which stands for
Jump-if-Equal, jumps to the `.else` label if the two things compared
by `cmp` were equal---that is, if RAX is equal to zero. If it
_isn't_ equal to zero, the `je` instruction does nothing and continues
to the next instruction:

 - So---if RAX is zero, we jump at the `je` instruction, execute the
   `mov rax, 12` instruction, and then end up at the `.end` label.

 - On the other hand---if RAX is non-zero, don't jump at `je`, execute
   the `mov rax, 7` instruction and then execute the `jmp` instruction
   to go to the `.endcond` label.

The end result is that if RAX started out equal to zero, it ends up
equal to 7, while if it started out equal to a non-zero value, it ends
up equal to 12.

There are many different kinds of conditional jump instructions,
documented in the [Assembly Handbook](../assembly.md).

To generate code for an `if` statement, you'll need to do something
like the above snippet, with a `cmp`, a `je` to the "else" branch, a
"then" branch that ends with a `jmp` to the end, and an "else" branch
that falls through to the "end".

Tip:

 - For `if` expressions, both the "then" branch and the "else" branch
   push N bytes on the stack. (Where N is the size of the result; both
   the "then" and "else" branch have the same type so will push the
   same number of bytes). But because _only one of_ the "then" and
   "else" branches gets executed, the stack should only change by N
   total, not by 2N! This is very easy to mess up and you will need to
   handle this carefully.


## Array indexing

Array indexing is complex and consists of several steps. The stack is
supposed to look like this:

![The stack configuration for an array index](index-stack.png)

Thus, you should first push the array itself on the stack, followed by
the indices in reverse order. Note that you'll have N indices and N
lengths, if the array is rank N. Each index and length is 8 bytes, so
the i^th^ index is at `[rsp + 8i]` and the i^th^ length is at `[rsp +
8i + 8N]`.

Next, you need to check that the indices are valid. A valid array
index is *greater than or equal to* 0 and *less than* its
corresponding length. Check each array index one by one, printing an
error message if it is negative or too large.

Next, you need to compute the address on the heap that you need to
index into. Suppose you are indexing `M[i, j, k]`, and `M` has
dimensions `A * B * C` and stores data of size `S`. Then you want the
following address:

    (((0 * A + i) * B + j) * C + k) * S + ptr

where `ptr` is the data pointer inside `M`. This expression is
confusing, so let's work through a 2d array example. If you have a
matrix `M` of floats with 30 rows and 3 columns, and you want the
element at row `i` and column `j`, that means location `3i + j` in the
matrix, or in the notation above:

    loc = (0 * 30 + i) * 3 + j

Including the useless `0 * 30` term makes the code-gen easier.

To go from the location in the `M` matrix to an address, we multiply
by the size of each matrix element and add the base pointer:

    loc * 8 + ptr

Combine these steps together and you get the formula above. you can
see the specific assembly instruction you'll need in the [Assembly
Handbook](../assembly.md).

Now that you've computed the address of the data, all that's left is
to free the indices and array, and copy `S` bytes from the computed
address to the the stack.


## Loops

Loops are similar to array indices, but more complex. We recommend
starting with `sum` loops, for which the stack should look like this:

![Configuration of the stack when executing a 3D sum loop in JPL](sum-stack.png)

You will first want to make room for the counter and then compute all
of the loop bounds. Check that each of the loop bounds is *greater
than zero*.

Next, write a 0 to the counter location. Conveniently, `0` and `0.0`
are both represented by the same eight-byte value: all zero bits.
(Thank you IEEE 754 committee!)

Next, push zeros on the stack to initialize each loop index. Save each
loop index's location on the stack in your `StackDescription`. (Make
sure to save the right location for each index---you'll be pushing the
indices on the stack in reverse order.)

You are now ready to start the loop body. (It needs a label so you can
jump back to it.

First, compute the body of the loop.

Once the body is computed, you'll need to add it to the loop counter.
This will require different instructions for integer versus
floating-point sums; again, details are in the [Assembly
Handbook](../assembly.md)

Finally, we need to increment the loop variables. We'll start with the
innermost loop variable, but if we increment it past the loop bound,
we'll reset it to 0 and increment the next loop variable. In the `sum
[i : X, j : Y, k : Z] body` example above, this section of the
generated code should be equivalent to the following C code:

    k ++
    if (k < Z) continue;
    k = 0;
    j++;
    if (j < Y) continue;
    j = 0;
    i++;
    if (i < X) continue;
    break;

Note that the variables here show up in reverse order. As long as
you're not at the end of the loop, one of `continue` statements should
trigger and bring you back to the start of the loop. Otherwise, the
`break` statement exits the loop.
    
If you don't understant how this code snippet works, do not start
coding! This is the part that makes it a loop! For example, make sure
you understand how the loop in [example.c](example.c) works. Feel free
to ask questions about `example.c` on Discord if it's not clear.

You can see the specific assembly code you'll need in the [Assembly
Handbook](../assembly.md).

After the loop, restore the stack by freeing all of the loop indices
and loop bounds. The counter should end up at the top of the stack.


## Array loops

JPL `array` loops are similar to `sum` loops, and we recommend sharing
code between them. The stack arrangement is similar:

![Configuration of the stack for 3D array loop](array-stack.png)

However, there are three key differences.

First of all, for a `sum` loop, you can just initialize the counter
to 0. For an `array` loop, that stack location must instead store a
heap pointer to the array contents. You will need to compute the
amount to allocate by multiplying all of the loop bounds, as in:

    X * Y * Z * S

Here `S` is the size of each array element. Do this multiplication
_after_ you check that all the loop bounds are positive.

As you multiply the loop bounds, it's very important to check for
integer overflow. For example, consider this JPL code:

    array[i : 65536, j : 65536, k : 65536, l : 65536] 1

The value 65536 is equal to 2^16^, so the total size of the array
requested is (2^16^)^4^ * 8, or 2^67^ bytes. Not only is this
impossible, but in 64-bit signed integer arithmetic, this computation
will return 0! It would be extremely bad to allocate 0 bytes and start
writing to them (`jpl_alloc` will print an error message in this
case). Variants of this issue are a common source of security
vulnerabilities.

Our compiler must avoid this error. In x86_64 assembly, you use the
`imul` instruction to multiply things:

    imul rdi, [rsp + 16]

Conveniently, this instruction sets the RFLAGS register (most integer
math instructions do), so you can follow it with:

      jno <next multiply>
        ...
      call _fail_assertion
    <next multiply>:

Here, `jno` stands for Jump-if-No-Overflow, so we only call
`fail_assertion` if there's an overflow, similar to the zero
denominator case for integer division and modulus. (Testing for
overflow is a common example of something that's really easy in
assembly, and quite difficult in C/C++.)

Once you've multiplied all of the bounds (and the size of each
element), call `jpl_alloc` to allocate memory for the array and store
it into the correct part of the stack.

Now it's time for the second key difference. At the end of a `sum`
loop, you need to add the value computed by the loop body to the
counter. At the end of an `array` loop, you instead need to compute a
location in the array to store the value computed by the loop body.
This is similar to array index operations. Like in an array index
operation, we compute the pointer to the array element we want to
store to. (You can make a helper method and generate the same exact
code.) But whereas an array index operation copied _from_ that pointer
to the stack, an array loop will copy from the stack _to_ that
pointer.

The third key difference is simply that after a `sum` loop ends you
need to free the loop indices and the loop bounds, but after an
`array` loop ends you only need to free the loop indices. The loop
bounds form part of resulting the array.

