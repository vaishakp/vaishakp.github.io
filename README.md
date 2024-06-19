# Compile-time optimizations of SpEC on HPCs

In this document I describe the compile-time optimizations that led to significant performance improvements of Binary Black Hole simulations using SpEC. 
This document is relevant for HPCs with x86-64 CPUs, especially for AMD CPUs. It also implements many generic CPU optimizations.

This is the result of an experimental exercise I carried out on the Sonic HPC at ICTS-TIFR, Bengaluru India. 


## Performance and optimization
Performance and optimization are important to the development and usage of HPC codes. They improve the quality of the code, reduce the memory footprint, and make it more efficient, saving valuable computing resources and reducing the carbon footprint. 

The basic philosophy behind improving the performance of a code can be broadly classified into three categories:
1. Avoid unnecessary work.
   Using the right tools to do only the work that is required succinctly and efficiently.
2. Parallel programming. Using all available computing resources, cores, and accelerators.
3. Use all hardware capabilities. In the context of CPUS, as CPUs evolve, they are equipped with increasing hardware instruction-level capabilities that can perform the same set of operations faster and more efficiently or in a lesser number of instructions. 

Why compile-time optimization?
Optimizations in 1 and 2 above are usually done in the development phase and require changes to the source code. While certain aspects of point three require changing the source code, there are certain other aspects of it that do not require changing the source code. From one perspective, this is simpler than the other two. Compile-time optimization is one such approach. Here, we aim to use the available capabilities of the compiler to make the most of the available hardware capabilities. 

1. Modern c/c++ compilers are gaining increasing sophistication in emitting efficient and optimized code from sources tailored to the hardware. 
2. Many high-performance scientific numerical codes like SpEC depend on an array of third-party libraries that implement various mathematical operations and numerical algorithms that are widely and frequently called e.g. BLAS, GSL, etc.
3. Modern processors support SIMD advanced vector extensions that can significantly improve the throughput.
4. Modern processors, especially those from AMD (as of 2024) have large cache, leading to improved cache hit/miss ratios, leading to immediate performance improvements.


In general software applications, the priority of the developers is towards adaptability and compatibility to a wide range of hardware capabilities and security. This brings severe limitations to the type of optimizations that can be carried out. However, for scientists/numerical relativists, performance (without loss of accuracy) is the priority. We often deal with a limited range of HPC hardware that does not change on a day-to-day basis. Thus it makes sense to invest time in carefully tailoring the software we use to the available hardware so as to make use of all its hardware capabilities, sacrificing portability. This document describes one such undertaking.



## General approach
1. Find out the hardware capabilities of the CPU and what major modern institutions are supported . This includes avx2 (256bit), avx512, 
