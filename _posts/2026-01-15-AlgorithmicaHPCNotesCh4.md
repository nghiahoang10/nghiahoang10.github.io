---
layout: post
title:  "Algorithmica HPC Notes - Chapter 4"
date:   2026-01-15 00:02:30 -0400
categories: jekyll update
permalink: /AlgorithmicaHPCNotesCh4/
---
## Ch4: Compilation

Most of the time compilers are capable of producing near-optimal code. When they do not, it is usually because the programmer knows more about the problem than what can be inferred from the source code but failed to communicate this extra information to the compiler.

### 4.1: Stages of Compilation

4 stages:

1. Preprocessing expands macros, pulls included source from header files, and strips off comments from source code: `gcc -E source.c` (outputs preprocessed source to stdout)
2. Compiling parses the source, checks for syntax errors, converts it into an intermediate representation, performs optimizations, and finally translates it into assembly language: `gcc -S file.c` (emits an .s file)
3. Assembly turns assembly language into machine code, except that any external function calls like printf are substituted with placeholders: `gcc -c file.c` (emits an .o file, called object file)
4. Linking finally resolves the function calls by plugging in their actual addresses, and produces an executable binary: `gcc -o binary file.c`

### 4.2: Flags and Targets

### 4.3: Situational Optimizations

Unroll loops whose number of iterations can be determined at compile time or upon entry to the loop:
```cpp
#pragma GCC unroll 4
for (int i = 0; i < n; i++) {
    // ...
}
```

Inlining is best left for the compiler to decide, but you can influence it with inline keyword:
```cpp
inline int square(int x) {
    return x * x;
}
```

Likeliness of branches can be hinted by `[[likely]]` and `[[unlikely]]` attributes in if-s and switch-es:
```cpp
int factorial(int n) {
    if (n > 1) [[likely]]
        return n * factorial(n - 1);
    else [[unlikely]]
        return 1;
}
```

### 4.4: Contract Programming

### 4.5: Precomputation

Compute value of variable during compile time and turn it into a constant by embedding it into the generated machine code.

Example:

```cpp
constexpr int fibonacci(int n) {
    int a = 1, b = 1;
    while (n--) {
        int c = a + b;
        a = b;
        b = c;
    }
    return b;
}
static_assert(fibonacci(10) == 55);
```