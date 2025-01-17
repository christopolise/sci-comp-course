---
---

# Phase 8: GPU

In this phase, you'll offload heavy computational work to a GPU. If you're already familiar with CUDA or similar, you can use it to write your code, but this lesson will assume that you're [using standard C++ and `nvc++`](../readings/gpu-programming.md#compiling). Since GPUs are so well-suited to this task, performance requirements are more strict: on a single P100 (which you can [request](../readings/schedulers.md) with `salloc --gpus 1 -C pascal ...`) the program should run on `2d-medium-in.wo` in less than 5 seconds.



## Iteration in 2 Dimensions

The [example code](https://github.com/BYUHPC/sci-comp-course-example-cxx/blob/main/src/MountainRangeGPU.hpp) shows how to use [`std::for_each`](https://en.cppreference.com/w/cpp/algorithm/for_each) and [`std::transform_reduce`](https://en.cppreference.com/w/cpp/algorithm/transform_reduce), which will be the only standard algorithms you need in your code. It does not, however, show [how to iterate over 2-dimensional ranges](https://www.nvidia.com/en-us/on-demand/session/gtcspring23-DLIT51170/?ncid=em-even-124008-vt33). For that, you'll want [this `cartesian_product.hpp`](https://github.com/gonzalobg/cpp_hpc_tutorial/blob/master/include/cartesian_product.hpp), which backports [`cartesian_product`](https://en.cppreference.com/w/cpp/ranges/cartesian_product_view) to C++20. With that included, you could sum the squares of the interior of `x` (which is a 1-dimensional `std::vector` pretending to be a 2-dimensional array of size `rows` by `cols`) thus:

```c++
auto [first, last] = two_d_range(1, rows-1, 1, cols-1);
auto sq_sum = std::transform_reduce(std::execution::par_unseq,
              /* iteration range */ first, last,
              /* initial value   */ value_type{0},
              /* reduce          */ std::plus<>(),
              /* transform       */ [cols=cols, x=x.data()](auto ij){
                                        auto [i, j] = ij;
                                        return std::pow(x[i*cols+j], 2);
                                    });
```

...where `two_d_range` is defined thus:

```c++
auto two_d_range(size_t first_x, size_t last_x, size_t first_y, size_t last_y) {
    auto range = std::views::cartesian_product(std::views::iota(first_x, last_x),
                                               std::views::iota(first_y, last_y));
    return std::array{range.begin(), range.end()};
}
```

You'll probably find it helpful to get the 1-dimensional index of the "current" cell and use it directly--for example, this makes typing out the Laplacian less tedious:

```c++
auto [i, j] = ij;
auto I = i * n + j;
// okay
auto L = (u[i*n+j-1] + u[i*n+j+1] + u[(i-1)*n+j] + u[(i+1)*n+j]) / 2 - 2 * u[i*n+j];
// better
auto L = (u[I-1] + u[I+1] + u[I-n] + u[I+n]) / 2 - 2 * u[I];
```



## Quirks

Remember to `module load nvhpc` before running CMake.

`wavesolve_gpu` will hang indefinitely if you try to run it on a login node, so get a shell on a node with P100s when you're ready to test:

```shell
salloc -G 1 -C pascal --mem 1G --time 00:10:00 # 1 P100 for 10 minutes
```

There are some limitations on the C++ you can successfully compile for GPUs with `nvc++`. [Only heap memory can be automatically managed](https://developer.nvidia.com/blog/accelerating-standard-c-with-gpus-using-stdpar/), but this won't likely be a problem since `std::vector`s store their data on the heap. One that is much more likely to bite you is the requirement that these heap data members **should be captured in the kernel function by pointer**, then indexed raw; this is shown in the example above, where indexing is manual and `x=x.data()` makes `x` a pointer within the kernel.



## Submission

Update your `CMakeLists.txt` to create `wavesolve_gpu`; as of this writing, CMake doesn't have infrastructure in place for GPU compilation with `nvc++`, so you'll have to look to the [example code](https://github.com/BYUHPC/sci-comp-course-example-cxx/blob/f5c8286e20d9aa49971dc7776d1c69c0286f80aa/CMakeLists.txt#L60) to see how to do so manually. Tag the commit you'd like me to grade from `phase8`, and push it.



## Grading

This phase is worth 20 points. The following deductions, up to 20 points total, will apply for a program that doesn't work as laid out by the spec:

| Defect | Deduction |
| --- | --- |
| Failure to compile `wavesolve_gpu` | 4 points |
| Failure of `wavesolve_gpu` to work on each of 3 test files | 4 points each |
| Failure of `wavesolve_gpu` to checkpoint correctly | 4 points |
| Failure of `wavesolve_gpu` to run on `2d-medium-in.wo` on a P100 in 5 seconds | 4 points |
| ...in 10 seconds | 4 points |
| `wavesolve_gpu` isn't a GPU program | 1-20 points |
