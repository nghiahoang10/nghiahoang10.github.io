---
layout: post
title:  "Algorithmica HPC Notes"
date:   2026-01-14 00:02:30 -0400
categories: jekyll update
permalink: /AlgorithmicaHPCNotes/
---
## Ch2: Instruction-Level Parallelism

Modern CPUs use pipelining: after an instruction passes through the first stage, they start processing the next one right away, without waiting for the previous one to fully complete.

### 3.1: Pipeline Hazards

The next instruction might not execute on the following clock cycle:

- A structural hazard happens when two or more instructions need the same part of CPU (e.g., an execution unit).
- A data hazard happens when you have to wait for an operand to be computed from some previous step.
- A control hazard happens when a CPU can’t tell which instructions it needs to execute next.

### 3.2: The Cost Of Branching

```cpp
volatile int s;

for (int i = 0; i < N; i++)
    if (a[i] < 50)
        s += a[i];
```
~14 CPU Cycles per element

Replacing the hardcoded 50 with a tweakable parameter P that effectively sets the probability of the < branch:

```cpp
    for (int i = 0; i < N; i++)
    if (a[i] < P)
        s += a[i];
```

<img src="../photos/3.2.1.svg" alt="performance graph"/>

Note that it costs almost nothing to check for a condition that never or almost never occurs.

Pattern Detection:

```cpp
for (int i = 0; i < N; i++)
    a[i] = rand() % 100;

std::sort(a, a + n);
```

Hinting Likeliness of Branches:

```cpp
for (int i = 0; i < N; i++)
    if (a[i] < P) [[likely]]
        s += a[i];
```

Read more: [stackoverflow](https://stackoverflow.com/questions/11227809/why-is-conditional-processing-of-a-sorted-array-faster-than-of-an-unsorted-array)
