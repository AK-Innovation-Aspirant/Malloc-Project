# Malloc Project Writeup — Dynamic Memory Allocator (CS252 Spring 2025) - NO CODE

## High-level description

This project builds a user-space **dynamic memory allocator**: a replacement for `malloc`, `free`, and `calloc` that manages heap memory by requesting large regions from the OS and then efficiently servicing many smaller allocations and deallocations. The allocator must balance **performance (throughput)** with **memory efficiency (fragmentation/utilization)** while maintaining correctness and robustness under diverse real-world allocation patterns.

## Course / policy note

Completed for **Purdue CS252 (Spring 2025)**. Source code is **not publicly available** in order to respect course and academic integrity policies.

## Implemented functionality

- `malloc(size)` — allocate dynamic memory
- `free(ptr)` — release previously allocated blocks
- `calloc(n, size)` — allocate and zero-initialize memory
- **Extra credit:** `realloc(ptr, new_size)` — resize an allocation while preserving contents when required
- **Extra credit:** compatibility-focused robustness so that allocation-heavy, real applications (e.g., **Google Chrome** and similar programs) run correctly with this allocator

## My key learnings

- **Heap management fundamentals:** how an allocator represents blocks, tracks free space, and safely services arbitrary request sizes
- **Fragmentation tradeoffs:** understanding internal vs. external fragmentation and why “fast” can still be “wasteful” (and vice versa)
- **Coalescing and splitting as concepts:** why merging/splitting free space matters for long-running programs with churny allocation patterns
- **Alignment and metadata overhead:** how constraints like alignment and per-block bookkeeping influence both correctness and utilization
- **Correctness under stress:** why allocators must survive adversarial patterns, high churn, and edge cases without corrupting memory

## How real allocators differ (and why)

Different production allocators (system `malloc`, jemalloc, tcmalloc, etc.) make different choices depending on goals and workload:

- **Speed vs. memory usage:** some prioritize throughput (fast paths, caching), others prioritize lower fragmentation
- **Data structure strategy:** designs range from simple lists to more specialized structures (e.g., segregated size classes)
- **Workload assumptions:** interactive apps, servers, and short-lived CLI tools all stress allocators differently
- **Scalability concerns:** multi-threaded programs often need strategies that reduce contention and improve locality

### Single-arena vs. multi-arena designs

This CS252 allocator is conceptually a **single-arena** design: heap management decisions are centralized, which is great for learning core mechanisms and invariants.

Many real-world allocators go further and use **multiple arenas / per-thread heaps**:
- **Reduced lock contention:** threads can allocate/free from their “local” arena most of the time instead of fighting over one global structure
- **Better locality:** allocations made by a thread often stay close together in memory, improving cache behavior
- **Different fast paths:** small allocations may be served from thread-local caches, while larger allocations follow separate paths

The tradeoff is increased complexity and potential memory overhead (e.g., unused free space distributed across arenas), but it often pays off for highly concurrent applications.

