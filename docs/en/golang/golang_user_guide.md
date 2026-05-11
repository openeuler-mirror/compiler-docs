# Introduction to Go Optimization

## Overview

The Go compiler is one of the core tools of the Go programming language (also referred to as Golang), and is responsible for converting readable Go source code into machine code that can be executed by a computer. It is known for its high efficiency, simplicity, and high integration.

## New Features

| No.| Feature         | Description              | How to Use             |
| ---- | ----------------- | ------------------ | --------------------- |
| 1    | Optimizing the number of span pages| Increases the number of pages in a span.| GOEXPERIMENT=pagenum  |
| 2    | Step optimization solution 2    | Optimizes the implementation of the step function.| GOEXPERIMENT=stepopt2 |

## Feature Usage Description

### Optimizing the Number of Span Pages

The memory page partition expansion optimization solution lies in the generation and optimization of 67 different size classes. These size classes are generated before the service code is compiled, and then are compiled in combination with the content of the runtime library and the memory manager, and finally are linked to the service code to form an executable file. When memory needs to be allocated, the memory manager determines, based on the size class information variable, the page table data volume required by an object. This effectively reduces time consumed in allocating a large quantity of small objects.

![sizeclasses_span.png](images/sizeclasses_span.png)

This design increases the number of objects that can be contained in a single mSpan. As shown in the preceding figure, the memory of 8,192 B can be divided into 314 free object spaces for 24 B objects, and 8 × 8,192 B can be divided into 2,730 free object spaces. In this way, the native space can accommodate multiple times of objects than before. When a large number of small objects are used, the number of times for applying for and operating an mSpan can be effectively reduced.

Option enabling: This option is controlled by parameters during service compilation.

```bash
# Perform service compilation or a singleton test.
GOEXPERIMENT="pagenum" GOMAXPROCS=1 go test -bench=BenchmarkMalloc -v -run=^$ runtime/
```

### Optimizing the step Function

The `readvarint` function is executed one or two times within a loop in the step function in most cases (accounting for more than 99.9%). The one-iteration loop and two-iteration loop are discussed separately and optimized using the Load Pair (LDP) instructions.

```bash
# Perform service compilation or a singleton test.
GOEXPERIMENT="stepopt2" GOMAXPROCS=1 go test -bench=BenchmarkMalloc -v -run=^$ runtime/
```
