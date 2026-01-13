# Dynamic Memory Allocator (User-Space)

## Overview
This project is a user-space dynamic memory allocator: a drop-in implementation of common allocation routines that manages heap memory by requesting large regions from the operating system and then servicing many smaller allocations and deallocations efficiently.

It’s intended as a systems exercise focused on allocator design, fragmentation behavior, alignment, and correctness under stress.

## Supported API Surface
- **`malloc(size)`** — allocate dynamic memory  
- **`free(ptr)`** — release previously allocated memory  
- **`calloc(n, size)`** — allocate and zero-initialize memory  

### Optional Extensions (when enabled)
- **`realloc(ptr, new_size)`** — resize an allocation while preserving contents when required  
- **Alignment-focused routines** such as **`memalign()`**, **`valloc()`**, **`posix_memalign()`**, and **`pvalloc()`** for compatibility with allocation-heavy applications and common runtime expectations.

## Design (High Level)
At a high level, the allocator:
- Acquires memory from the OS in **large chunks** rather than per-allocation.
- Tracks allocations using **per-block metadata** (size/status) to support safe reuse.
- Reuses freed space by maintaining **structures for quickly finding suitable free blocks**.
- Controls fragmentation via **block splitting** (to avoid waste on smaller requests) and **coalescing** (to merge adjacent free space back into larger blocks).
- Ensures allocations satisfy **alignment constraints**, balancing correctness with metadata overhead.

## Key Learnings
- **Heap management fundamentals:** representing blocks, tracking free space, and safely servicing arbitrary request sizes.
- **Fragmentation tradeoffs:** internal vs. external fragmentation, and why “fast” designs can increase memory waste (and vice versa).
- **Splitting and coalescing:** how they impact long-running workloads with churny allocation patterns.
- **Alignment and overhead:** how alignment requirements and bookkeeping influence both correctness and utilization.
- **Correctness under stress:** surviving adversarial patterns, high churn, and edge cases without corrupting memory.

## How Production Allocators Differ (and Why)
Different allocators (system malloc variants, jemalloc, tcmalloc, etc.) make different engineering choices based on goals and workloads:
- **Speed vs. memory usage:** throughput-focused fast paths and caching vs. tighter memory utilization.
- **Data structure strategy:** from simpler lists to more specialized size-class schemes.
- **Workload assumptions:** interactive apps, servers, and short-lived CLI tools stress allocators differently.
- **Scalability:** multi-threaded workloads often require designs that reduce contention and improve locality.

### Centralized vs. Multi-Arena Designs
A centralized (single “arena”) approach is excellent for learning core invariants and mechanisms.  
Many real allocators go further with multiple arenas or per-thread caches to:
- Reduce contention
- Improve cache locality
- Add specialized fast paths for small allocations

The tradeoff is increased complexity and potential memory overhead from free space distributed across arenas.

## Notes
This repository intentionally avoids publishing implementation details that would function as a step-by-step solution. It’s meant to document the concept, scope, and learning outcomes rather than serve as a reference implementation.
