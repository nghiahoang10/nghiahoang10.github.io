---
layout: post
title:  "Algorithmica HPC Notes - Chapter 5"
date:   2026-01-16 00:02:30 -0400
categories: jekyll update
permalink: /AlgorithmicaHPCNotesCh5/
---
## Ch5: Profiling

3 main techniques:
- Instrumentation lets you time the program as a whole or by parts and count specific events you are interested in.
- Statistical profiling lets you go down to the assembly level and track various hardware events such as branch mispredictions or cache misses, which are critical for performance.
- Program simulation lets you go down to the individual cycle level and look into what is happening inside the CPU on each cycle when it is executing a small assembly snippet.

### 5.1: Instrumentation

Inserting timers and other tracking code into programs.

```cpp
clock_t start = clock();
do_something();
float seconds = float(clock() - start) / CLOCKS_PER_SEC;
printf("do_something() took %.4f", seconds);
```

### 5.2: Statistical Profiling

Tools:
- perf on Linux
- VTune from Intel for non-Linux

Example:
```cpp
void setup() {
    for (int i = 0; i < n; i++)
        a[i] = rand();
    std::sort(a, a + n);
}

int query() {
    int checksum = 0;
    for (int i = 0; i < n; i++) {
        int idx = std::lower_bound(a, a + n, rand()) - a;
        checksum += idx;
    }
    return checksum;
}
```

Compile with `g++ -O3 -march=native example.cc -o run` and run `perf stat ./run` gives:
```
 Performance counter stats for './run':

        646.07 msec task-clock:u               # 0.997 CPUs utilized          
             0      context-switches:u         # 0.000 K/sec                  
             0      cpu-migrations:u           # 0.000 K/sec                  
         1,096      page-faults:u              # 0.002 M/sec                  
   852,125,255      cycles:u                   # 1.319 GHz (83.35%)
    28,475,954      stalled-cycles-frontend:u  # 3.34% frontend cycles idle (83.30%)
    10,460,937      stalled-cycles-backend:u   # 1.23% backend cycles idle (83.28%)
   479,175,388      instructions:u             # 0.56  insn per cycle         
                                               # 0.06  stalled cycles per insn (83.28%)
   122,705,572      branches:u                 # 189.925 M/sec (83.32%)
    19,229,451      branch-misses:u            # 15.67% of all branches (83.47%)

   0.647801770 seconds time elapsed
   0.647278000 seconds user
   0.000000000 seconds sys
```
The execution took 0.53 seconds or 852M cycles at an effective 1.32 GHz clock rate, over which 479M instructions were executed. There were also 122.7M branches, and 15.7% of them were mispredicted.

perf record <cmd> records profiling data and dumps it as a perf.data file, and then call perf report to inspect it:

```
Overhead  Command  Shared Object        Symbol
  63.08%  run      run                  [.] query
  24.98%  run      run                  [.] std::__introsort_loop<...>
   5.02%  run      libc-2.33.so         [.] __random
   3.43%  run      run                  [.] setup
   1.95%  run      libc-2.33.so         [.] __random_r
   0.80%  run      libc-2.33.so         [.] rand
```

### 5.3: Program Simulation

Cachegrind of Valgrind can be used to profile caches access and branches prediction

```
g++ -O3 -g sort-and-search.cc -o run
valgrind --tool=cachegrind --branch-sim=yes --cachegrind-out-file=cachegrind.out ./run
cg_annotate cachegrind.out --auto=yes --show=Dr,D1mr,DLmr,Bc,Bcm
```

```
Dr         D1mr      DLmr Bc         Bcm       
         .         .    .          .         .  int binary_search(int x) {
         0         0    0          0         0      int l = 0, r = n - 1;
         0         0    0 20,951,468 1,031,609      while (l < r) {
         0         0    0          0         0          int m = (l + r) / 2;
19,951,468 8,991,917   63 19,951,468 9,973,904          if (a[m] >= x)
         .         .    .          .         .              r = m;
         .         .    .          .         .          else
         0         0    0          0         0              l = m + 1;
         .         .    .          .         .      }
         .         .    .          .         .      return l;
         .         .    .          .         .  }
```