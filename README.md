# Compile-time optimizations of SpEC on HPCs

In this document, I describe the compile-time optimizations that led to significant performance improvements of Binary Black Hole simulations using SpEC. 
This document is relevant for HPCs with x86-64 CPUs, especially for AMD CPUs. It also implements many generic CPU optimizations.

This is the result of an experimental exercise I carried out on the Sonic HPC at ICTS-TIFR, Bengaluru India. 


## Performance and optimization
Performance and optimization are important to the development and usage of HPC codes. They improve the quality of the code, reduce the memory footprint, and make it more efficient, saving valuable computing resources and reducing the carbon footprint. 

The basic philosophy behind improving the performance of a code can be broadly classified into three categories:
1. Avoid unnecessary work. Using the right language tools to only do the work that is required, succinctly and efficiently.
2. Parallel programming. Using all available computing resources, cores, and accelerators.
3. Use all hardware capabilities. In the context of CPUS, as CPUs evolve, they are equipped with increasing hardware instruction-level capabilities that can perform the same set of operations faster and more efficiently or in a lesser number of instruction cycles. 

### Why compile-time optimization?

Optimizations in 1 and 2 above are usually done in the development phase and require changes to the source code. While certain aspects of point three require changing the source code, there are certain other aspects of it that do not require changing the source code. From one perspective, this is simpler than the other two. Compile-time optimization is one such approach. Here, we aim to use the available capabilities of the compiler to make the most of the available hardware capabilities. 

1. Modern c/c++ compilers are gaining increasing sophistication in emitting efficient and optimized code from sources tailored to the hardware. 
2. Many high-performance scientific numerical codes like SpEC depend on an array of third-party libraries that implement various mathematical operations and numerical algorithms that are widely and frequently called e.g. BLAS, GSL, etc.
3. Modern processors support SIMD advanced vector extensions that can significantly improve the throughput.
4. Modern processors, especially those from AMD (as of 2024) have large caches, leading to improved cache hit/miss ratios, resulting in immediate performance improvements.


In general software applications, the priority of the developers is towards adaptability and compatibility to a wide range of hardware capabilities and security. This brings severe limitations to the type of optimizations that can be carried out. However, for scientists/numerical relativists, performance (without loss of accuracy) is the priority. We often deal with a limited range of HPC hardware that does not change on a day-to-day basis. Thus it makes sense to invest time in carefully tailoring the software we use to the available hardware to make use of all its hardware capabilities, sacrificing portability. This document describes one such undertaking.



## General approach
1. Find out the hardware capabilities of the CPU and what major modern institutions are supported and turn them on at compile time. In most cases, most performance gains result from the use of all the native instructions, especially the AVX instruction sets. I recommend using avx2 over avx512. Although avx512 does result in performance gains over avx2, this is not always the case in my experience. One of the reasons is that avx512 is power-hungry and results in more thermal throttling. Furthermore, it is more common to find and group e.g. 8 double data types to perform a 256-bit vector operation than a 512-bit one.
2. Use of FMA leads to performance gains. However, it is to be noted that if math is not written safely, this can lead to large errors. E.g. see the following.
4. The use of static linking often leads to better performance. When profiling, it is advantageous to use dynamic libraries with PIC/PIE.
5. Using the latest version of libraries e.g. SpEC ID solver uses PETSc internally. The use of recent versions showed performance benefits.
6. Consistent compiling. Once the optimization flags are chosen, ensure the compilation of all the third-party libraries and the main application (SpEC here) with the same flags.
7. Compilers. Common choices are clang (llvm, amd), Intel (icc), gcc.
   1. On AMD machines, use AMD's new clang compiler, available in the AOCC compiler suite. This is supposed to lead to code that is adapted to AMD CPUs. However, AMD's clang could not be used to successfully compile SpEC will all optimizations turned on, due to possible bugs. I will talk about this later.
   2. On Intel machines, it is the Intel Parallel Studio or icc for the c/c++ compiler.
   3. The performance of GCC is very much comparable to that of Clang on AMD CPUs.
8. Once a compiler is chosen, compile all dependencies, third-party software, and the main application with the same compiler.
9. For all production code, use O3, and for debugging use Og.
10. Although Ofast leads to faster performance, it turns on many unsafe math operations and discards FP checks leading to loss in accuracy at the least. It is highly discouraged to turn this option on.
11. BLAS. 
    1. On AMD CPUs, openBLAS gives the best performance.
    2. On intel, it is the MKL
    3. A newer version of LAPACK has comparable performance to openBLAS on AMD systems.
    4. Intel MKL often performs better than LAPACK on AMD systems, provided a fix is implemented. More on this later.
12. OpenMP is favorable from a performance perspective. Although I am not confident of thread safety and race conditions with SpEC.
13. MPI.
    1. openMPI is favorable on AMD systems.
    2. intel MPI on Intel systems.
14. AMD has recently come up with a suite of Linear algebra, math, and low-level mem implementations (AOCL). This will definitely result in better performance, cache usage, and memory operations. However, the initial versions (4.0) were buggy and could not be successfully tested with SpEC (e.g. newer versions of hdf5 would not compile with AOCL and AOCC). Something for the future.
15. Other more time-consuming, manual, and advanced optimizations that require profiling and are iterative. These are not recommended for most people. This includes profile-guided optimizations and operations like tuning the depth of loop unrolling.
16. Running SpEC over NAS storage or any network-connected storage is not recommended. Apart from latency issues, on Sonic, I found that SpEC hangs every time a packet is dropped, and the MPI processes are exposed to race conditions, even if TCP is used.
17. For benchmarking certain third-party linear algebra libraries with various combinations of compilers, please refer to www.gitlab.com/vaishakp/benchmarks.git
18. glibc. glibc is one of the most important libraries that determines the performance. Usually, the Linux kernel is inseparable from glib versioning. This means that one cannot upgrade glibc safely and consistently without recompiling the kernel. I highly recommend using glibc > 2.34, especially on AMD systems.
    1. On older versions, glibc was not correctly parsing the available cache on most AMD and some intel systems. This was a huge disadvantage to the newer AMD processors:
    2. On older versions, a certain part of the code in glibc was forcing slower code paths on AMD systems.
    3. The implementation of various math libraries has been improved in newer glibc versions with e.g. vector intrinsic support.
    If the HPC OS is using older glibc versions, I recommend upgrading the OS.
19. Details on additional experiments with SpEC can be found at https://gitlab.com/vaishakp/spec-on-hpcs
    
## Compiling SpEC
### Versions of third-party libraries used (as of March 2023)

texinfo 7.0.2
make 4.4
gcc 11.1.0
hwloc 2.9.0
cmake 3.25.2
knem 1.1.4
xpmem 2.6.5
openmpi 4.1.4
netlib-lapack
fftw 3.3.10
gsl 2.7.1
hwloc 2.9.0
hdf5 1.14.0
papi 7.0.0
petsc 3.18.4 
amd libraries
lapack 3.11.0


#### Other
aocc-compilers 4.0.0
aocl 4.0.0
   amd-blis
   amd-libflame
   amd-fftw
   amd-libm
   amd-libmem


### Compiler flags
The following flags were used to compile ALL the software/libraries:
-mavx2, -mfma, -fPIC, -O3, -march=native

### Issues with FMA
FMA itself is IEEE compliant and leads to performance and accuracy gains. However, some isolated math expressions (which can be classified as unsafe) lead to anomalous loss of accuracy. E.g. consider

Some tests on SpEC fail at the file comparison stages if compared with output in existing  Save directories. It is recommended to re-generate tests in these cases.

## Results

### ID solver


### Evolution


