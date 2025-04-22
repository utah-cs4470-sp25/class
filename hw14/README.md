Assignment 14: Loop Permutation
===============================

Add a loop permutation pass to your compiler.

**April 14:** implement the `-O3` flag, not `-O2`. It's for historic reasons.

**April 16:** do not implement constant propagation. Pavel says it wasn't needed in the first place. Sorry!

There are 5 test programs:

- `col`: columnar sum routine
- `crs`: cross product computed via Levi-Civita symbols
- `mat`: matrix multiply routine
- `sft`: softmax neural network layer inference
- `dns`: dense feed-forward neural network layer inference

Each program loads data (2025: using array loops to avoid `read image` for now),
converts the data into integer or float arrays, and then runs a
tensor contraction on the arrays of numbers.

In order to get ANY test to pass, you must complete several subtasks:

- NEVERMIND SORRY: ~~0. Implement constant propagation from `let`-bound variables to their uses in array bounds.~~
- `1.` Identify loops that match a grammar of _tensor contractions_ (details below).
- `2.` Build a traversal graph and compute the topological order of loop variables.
- `3.` Generate assembly that matches the staff compiler exactly.


# Overview

In this assignment, we focus on `array` loops whose body is a `sum` loop:
~~, and
where all loop bounds are statically known:~~

    array[i : L, j : N] sum[k : M] A[i, k] * B[k, j]

> Loop permutation is much harder to do if the inner-loop bounds depend
> on variables from the outer loop, e.g. if `M` was instead `i + 1`.
> Ignore this issue for HW14. We will not ask you to optimize or
> detect inner-loop bounds that depend on outer-loop variables.

Moreover, the body of the inner `sum` loop can only contain binary
operations, constants, variables, and indexing into arrays. This
category of computations is called **tensor contraction** and is
essential to statistics, scientific computing, and machine learning.

We will build a *traversal graph* for a tensor contraction. Each
variable bound by the `array` or `sum` forms a node of the graph.
Every *indexing pair* of two variables forms an edge. Indexing pairs
(`i`, `j`) are created by:

- An array index expression `A[..., i, ..., j, ...]`
- An array loop expression  `array[..., i : ..., ..., j : ..., ...] ...`
- A sum loop expression `sum[..., i : ..., ..., j : ..., ...] ...`,
  but **only** if the sum is summing floating-point numbers.

The basic idea of this graph is that an indexing pair (`i`, `j`) means
that we want `j` to change more often than `i`. In an array index or
an `array` loop, that's because it means we will traverse the array in
a cache-efficient way. In a `sum` loop over floating-point values,
that's because floating-point addition isn't associative, so we can't
change the order of floating-point summations.

For example, consider matrix multiplication:

    array[i : L, j : N] sum[k : M] A[i, k] * B[k, j]

Here the graph nodes are `i`, `j`, and `k`, and the edges are:

- The index `A[i, k]` creates the edge (`i`, `k`)
- The index `B[k, j]` creates the edge (`k`, `j`)
- The `array[i : L, j : N] ...` loop creates the edge (`i`, `j`)

Here's another example, from a neural network inference routine:

    array[i : W, j : H] sum[i2 : W, j2 : H] input[i2, j2] * weights[i2, j2, i, j]

The edges are:

- From `input[i2, j2]`, the edge (`i2`, `j2`)
- From `weights[i2, j2, i, j]`, six edges:
  from `i2` to `j2`, `i`, and `j`;
  from `j2` to `i` and `j`;
  and from `i` to `j`
- From the `array` loop, the edge (`i`, `j`)
- From the `sum` loop, the edge (`i2`, `j2`)

Given the traversal graph, we can now create the *topological order*
of the variables. In the topological order, if there's an edge from
`i` to `j`, then `i` must come before `j` in the order. For matrix
multiplication, the only topological order is `i`, then `k`, then `j`.
For the neural network inference routine, the only topological order
is `i2`, then `j2`, then `i`, then `j`.

Finally, we want to generate code that increments the loop variables
in the topological order. Typically, the topological order isn't the
order of the variables in the program.

Speedups of 10–16x can result from switching to the topological order. On a
matrix multiplication routine, loop permutation creates speedups of about 16x
and results in slightly faster code than equivalent C compiled by LLVM 16
(which does not do loop permutation).


# Step 1: Detecting tensor contractions

Our first step is detecting tensor contraction expressions. Formally,
tensor contractions must match these rules:

- It is an `array` loop;
- Whose body is a `sum` loop;
- Whose body returns a Float and matches the following grammar for tensor contraction bodies (`tc_body`):

```
tc : array [ <variable> : <tc_primitive> , ... ] <tc_sum>

tc_sum : sum [ <variable> : <tc_primitive> , ... ] <tc_body>

tc_body : <tc_body> <binop> <tc_body>
        | <tc_primitive> [ <tc_primitive> , ... ]
        | <tc_primitive>

tc_primitive : <integer> | <float> | <variable>

// floating-point binops
binop : + | - | * | / | % 
```

Implementation suggestions in: [hints.md](./hints.md).


# Step 2: Building the traversal graph

Add two new fields to `ArrayLoopExpr` to store the traversal graph.
These should be a list of variable names for the nodes (`tc_nodes`)
and a list of pairs of variable names for the edges (`tc_edges`).

Fill these fields in after you set the `is_tc` field on an expression.
You'll probably want to fill in `tc_nodes`, and then call a recursive
function that will traverse through the tensor contraction body to
fill in `tc_edges`.

Important rules:

- `tc_nodes` should be in order, with the `array` loop variables first
  and the `sum` loop variables afterward. The order of `tc_edges`
  doesn't matter.
- Make sure to add the edges that come from the `array` and possibly
  `sum` loop as well.
- If an array index, `array` loop, or float `sum` loop have only one
  element, they don't add any edges; if they have more than two
  elements, they add many edges.
- If an array index is an integer or a variable that isn't in
  `tc_nodes`, it doesn't add any edges. Your recursive function will
  probably need to passed the `tc_nodes` list to check this.

Debugging hints: [hints.md](./hints.md).


# Step 3: Computing the topological order

We recommend the following naive algorithm for topological ordering.
It is easy to implement, and performance shouldn't matter since
it's rare to have tensors with hundreds of dimensions.

Algorithm:

 1. The topological order starts empty
 2. Go through the nodes one by one
 3. Check each node for whether it is the target of any edge
 4. Find a node that isn't an edge target
 5. Add it to the topological order
 6. Remove it from the node list
 7. Remove any edges whose source is this node
 8. Repeat from Step 2.

Example and debugging hints: [hints.md](./hints.md).


# Step 4: Generate loop code

At this stage, the `TensorContraction` pass is done, and it's time to
use this topological order when generating code. The main challenge is
that we want to create a single loop that contains both the `array`
and `sum` iterations. We'll be able to reuse most of the existing code
generation, but we'll need to special case `array` loops that have
a topological order.

Code generation should go through the following steps:

- Allocate 8 bytes on the stack for the pointer
- Compute each loop bound and push them on the stack *in a specific order*:
  push array bounds first, then sum bounds.
- Multiply all of the array bounds (*not the sum bounds*)
  together, checking for overflow, and call `jpl_alloc` to allocate
  space for the result.
  + Be careful: the array bounds are no longer on top of the stack---there are
    sum bounds in the way---so you'll need to add an offset to your
    multiplication code.
  + You don't need to do anything special to zero out the memory; `jpl_alloc`
    does that for you.
- Save the allocated pointer to the stack in the correct place
- Push a `0` onto the stack *for each of the array and sum variables*.
  + Initialize the array variables first, then the sum variables. Both groups of variables must
    be initialized in reverse (right-to-left) order, like normal.
- Begin the loop
- Compute the body of the **sum** loop
- Compute the pointer into the array using *only the array indices*.
  + Because the sum indices and bounds are in the way,
    you'll need to adjust your stack offsets.
- Add the body to the computed pointer
- Increment the loop indices using the topological order.
- Once outside the loop, free all array and sum indices and also
  the sum bounds, but not the array bounds.

We recommend going step by step through these modifications. Each one
modifies a distinct part of the assembly code, so you should be able
to see your progress by watching the diff get shorter, for the five
benchmarks, as you implement each modification.

The stack layout for loop variables does not depend on the topological order.
Picture in: [hints.md](./hints.md).


# Testing your code

Your compiler must support the `-s` flag and the `-O3` flag.

Just like HW13, we will run your compiler two ways:

1. With only `-s`, generate the same assembly code expected for previous assignments:
   ```
   make run TEST=/grader/ok/001.jpl FLAGS=-s
   ```

2. With `-s` and `-O3`, generate loop-permuted assembly:
   ```
    make run TEST=/grader/ok/001.jpl FLAGS="-s -O3"
   ```

You can test on your computer by downloading the auto-grader and running it
like so:

    make -C <auto-grader directory> DIR=<compiler directory> PART=<part> test-hw14


# Submission and grading

This assignment is due Friday April 18.

| Weight | Function  |
|--------|-----------|
| 15%    | col       |
| 15%    | crs       |
| 20%    | mat       |
| 25%    | sft       |
| 25%    | dns       |

None of the test cases require support for function definitions.

