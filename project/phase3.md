---
---

# Optimization

This is a guided assignment that should give you an idea of what to look for when optimizing C++ code.

Download [`optimize.cpp`](https://rc.byu.edu/course/optimize.cpp) on the supercomputer. Compile it with optimization and see how fast it runs:

```shell
g++ -std=c++20 -O3 -o optimize optimize.cpp # the "O" in "-O3" is the letter
time ./optimize
```

You'll submit an informal "essay" detailing the changes you made and the associated performance gains, so **write down answers to the bolded questions that follow as you get to them**.

Throughout the assignment, the result needs to remain correct--if the output simulation time changes you'll need to fix it before proceeding.



## Eliminating Unnecessary Work

Many of the functions in `optimize.cpp` take copies of large variables. To see how huge an effect this can have, change the `laplacian` function to take `x` by [const reference](https://www.learncpp.com/cpp-tutorial/pass-by-const-lvalue-reference/) rather than by value. Re-compile and time it again after doing so. **How much of a speedup do you get? Why do you think it's so dramatic?**

Set `rows` to 800, where it will stay for the rest of the assignment, and compile and time once more for comparison to subsequent changes.

Fix any other functions (along with [lambdas](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture), [range-based for loops](https://en.cppreference.com/w/cpp/language/range-for#Example), etc.) that should take arguments by const reference rather than by value. When you're done and have recompiled, the `system` field from `time`'s output should be close to zero. **How much more of a speedup do these changes yield? Why isn't it as significant**?



## Cache Efficiency

Some of the for loops in `optimize.cpp` iterate over columns first, then rows. Change the order of iteration on these loops, iterating over `i` then `j` rather than `j` then `i`. **How much faster does `optimize` run with this change? Why do you think that is?**



## Vectorization

To see the results of this change, you'll need to coax the compiler to vectorize; in this case, you can do so by adding `-funsafe-math-optimizations` to your compile flags. Do so, then time `optimize` again for a point of comparison.

Remove the `if (...) continue;` statements in the inner loops of `energy` and `step`, instead iterating over the correct bounds. **How much of a speedup results, and why?**



## Threading

Now that most of the program's inefficiencies are eliminated, it's worth turning to threading. Add [OpenMP threading](../readings/openmp.md) to your program, parallelizing each outer loop in `energy` and `step`. You'll need a [`reduction` clause on each loop in `energy` to correctly sum `E`](https://en.wikibooks.org/wiki/OpenMP/Reductions#A_parallel_way_of_summing). You can look to the `#pragma openmp ...` lines in [`MountainRangeOpenMP.hpp`](https://github.com/BYUHPC/sci-comp-course-example-cxx/blob/main/src/MountainRangeOpenMP.hpp) for an example; there's no need to overthink it, OpenMP threading is very easy. **What sort of speedup do you get when you run with 2 threads? 4? 8? 16? Why do you think the diminishment of returns is so severe?**



## Other Optimizations

Since you'll probably be applying some of the changes you made above to your own code anyway, feel free to use it as the basis for any further optimizations. I only care that the [`optimize` binary that your `CMakeLists.txt` outputs](#submission) is created, behaves correctly, and is fast.

Make any other optimizations you think are prudent. Some things to consider:

- Does the problem's symmetry lend itself to algorithmic optimization?
- Does the energy calculation need to continue once the floor has been exceeded?
- Does the potential energy calculation really require two separate loops over `u`?
- Is iterating over separate, rather than contiguous, vectors the most efficient?
- Would it be more efficient to interleave `u` and `v`?
- Could an iterative method (that yields *exactly* mathematically equivalent results, of course) speed things up?

**How much more of a speedup were you able to get? What seemed to help the most, and why do you think that's the case? Did you find out anything interesting or unexpected during the course of this optimization?**



## Submission

Update your `CMakeLists.txt` to create two more binaries, `optimize` and `solver_openmp`.

`optimize` should give the same result as the binary that results from compiling the original `optimize.cpp`, but needs to be much faster: on a full `m9` node (which you can request with `salloc -p m9 -N 1 -n 28 ...`) with 8 OpenMP threads (`export OMP_NUM_THREADS=8`) it should run in less than a second. Feel free to [fine-tune compilation flags in your `CMakeLists.txt`](https://coderefinery.github.io/cmake-workshop/flags-definitions-debugging/#controlling-compiler-flags) to get an efficient binary.

`solver_openmp` will be much like [`solver_serial`](phase2.md), but will add OpenMP threading and has speed requirements. It must run on `2d-medium-in.wo` from [`wavefiles.tar.gz`](https://rc.byu.edu/course/wavefiles.tar.gz) with 8 OpenMP threads on a full `m9` node in 20 seconds.

Add an informal plain text "essay" titled `cpp-optimization.txt` (or `cpp-optimization.md` if you want markdown formatting) that contains answers to the bolded questions above.

Tag the commit you'd like me to grade from `phase3` and push it.



## Grading

This phase is worth 20 points. The following deductions, up to 20 points total, will apply for a program that doesn't work as laid out by the spec:

| Defect | Deduction |
| --- | --- |
| Failure to compile | 2 points per binary |
| Failure of `solver_openmp` to work on each of 3 test files | 2 points each |
| Failure to run successfully (e.g. due to a segmentation fault or hang) | 2 points per binary |
| Failure of `solver_openmp` to checkpoint correctly | 2 points |
| Failure of `solver_openmp` to run on `2d-medium-in.wo` on `m9` with 8 threads in 20 seconds | 4 points |
| ...in 60 seconds | 4 points |
| Failure of `optimize` to output "5.39", and only "5.39", to stdout | 4 points |
| Failure of `optimize` to run within a second on `m9` with 8 threads | 4 points |
| `optimize` isn't relevant to the assignment, or just prints "5.39" or similar | 8 points |
| The informal essay doesn't answer the bolded questions above, or the answers are nonsensical | 1-8 points |