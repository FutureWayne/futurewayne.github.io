---
title: "Custom Memory Allocator"
summary: "A custom memory allocator system in C++"
categories: ["Post","Blog",]
tags: ["post","C++"]
showSummary: true
date: 2024-01-01
draft: false
---



# Custom Memory Allocator

## Project Overview

This project is a custom memory allocator system in C++, designed for efficient memory management in performance-critical applications like video games. It replaces the default `new/delete` with a tailored allocator that minimizes fragmentation and improves allocation speed. The allocator consists of two core components:

- **FixedSizeAllocator** for small, fixed-size allocations
- **HeapManager** for general dynamic allocations

By handling memory in a game-specific way (e.g. pooling small objects, aligning data), this system helps gameplay programmers manage resources more predictably and efficiently than the standard heap. It demonstrates low-level memory management skills that are highly relevant for gameplay programming.

---

## Technical Architecture

### Overview

- Two-tier architecture: multiple fixed-size pools + general heap.
- A large contiguous memory buffer is initialized.
- FixedSizeAllocator instances are created for specific sizes (e.g. 16B, 32B, etc.).
- Remaining buffer space is managed by the HeapManager.
- Allocation routing:
  - Small/fixed-size → FixedSizeAllocator
  - Others → HeapManager

This ensures that frequent small allocations (like bullets, particles) are efficient and reduce heap fragmentation.

---

## FixedSizeAllocator

The **FixedSizeAllocator** manages a pool of equal-size blocks and is optimized for fast allocation/deallocation of objects of the same size. Key characteristics of this component include:

### Key Characteristics

- **BitArray for Tracking**:  
  Each FixedSizeAllocator uses a compact bit array to track which blocks are free or in use. This allows O(1) checks and updates of allocation status by manipulating bits instead of heavier data structures. For example, setting a bit marks a block as occupied, and clearing it marks the block free. The bit array and related utility functions (like finding the first free bit) provide an efficient way to find available blocks without searching through complex lists.

- **Guardbands for Safety**:  
  To catch buffer overruns, the allocator can insert **guardband** patterns around each allocated block. These are small canary values (e.g. `0xDEADBEEF`) placed before and after the block data in memory. In debug builds, the guardbands are enabled via a macro (`ENABLE_GUARDBANDS`) and are checked upon deallocation to detect any corruption (e.g. if a buffer overflow overwrote a guard value). In release builds, guardbands can be disabled to eliminate the overhead. This feature adds a layer of memory safety particularly useful during development and testing.

- **Constant-Time Allocation**:  
  Allocating from a FixedSizeAllocator involves finding the first clear bit in the bit array (indicating a free block), marking it as used, and returning the block’s address. Internally, the allocator calculates the memory address as an offset in its pool using simple pointer arithmetic. Because block size is fixed, the index of a bit corresponds directly to a memory offset. This makes allocation and deallocation operations predictable and fast (iterating over bits or using bit-ops), ideal for gameplay scenarios where many small objects are created and destroyed frequently.

- **Preconfigured Pools**:  
  The system can instantiate multiple FixedSizeAllocator pools for different block sizes. For example, the project sets up pools for sizes ~16 bytes, 32 bytes, 96 bytes, 256 bytes, and 1024 bytes (with various block counts). Each pool occupies a portion of the overall memory buffer. This segregated pool approach means small objects are handled in their own arenas, which **prevents fragmentation** between small and large allocations and takes advantage of size-specific optimizations.

---

## HeapManager

The **HeapManager** is a general-purpose allocator that manages a large memory region for variable-size allocations. It functions like a simplified heap within the provided buffer, using a **linked-list of memory blocks** to track free and allocated regions. Important aspects of the HeapManager include:

### Key Aspects

- **Memory Block Metadata**:  
  The HeapManager represents each free or allocated chunk with a `MemoryBlock` structure containing its size, a pointer to the block’s usable memory, and pointers linking to the next block in the list. These `MemoryBlock` descriptors are stored **in-place** in the managed memory itself. In other words, when a block is allocated, the system carves out space for a `MemoryBlock` header at the start of that block’s region, rather than allocating metadata elsewhere. This in-place metadata design avoids extra allocations and ensures the entire memory system operates within the single provided buffer.

- **Allocation with Alignment**:  
   The allocator honors alignment requirements for allocated blocks.When fulfilling an allocation request, it calculates any necessary padding to satisfy the requested alignment. The HeapManager is designed to utilize any existing alignment gap in a free block before splitting it, thereby maximizing usable memory.Internally, it computes an adjustment offset so that the returned pointer meets the alignment (using a formula based on the address modulo alignment). If a free block is larger than needed, the HeapManager will **split or shrink** the block: it allocates the portion needed (with an aligned address) and leaves the remainder as a smaller free block in the list. The allocated block’s metadata stores how much alignment adjustment was applied, so that this padding can be recovered later if the block is freed.

- **Free List & Coalescing**:  
  All free memory blocks are maintained in a singly-linked list, sorted by their memory address. When a block is freed, it’s inserted back into the free list in address order, which prepares the system for efficient coalescing. The **Collect** operation (garbage collection) traverses the free list and **merges adjacent free blocks** into one larger block. For example, if a freed block lies immediately next to another free block in memory, they are combined into a single continuous free region. This coalescing reduces fragmentation over time and maximizes the size of contiguous free memory available for big allocations. The allocator may call `Collect()` automatically if an allocation request fails due to fragmentation, and then retry the request to see if memory has been liberated by merging small frees into larger ones.

- **First-Fit Strategy**:  
  The HeapManager’s allocation strategy scans the free list for the first block that can satisfy the request (considering size + alignment). This **first-fit approach** tends to be fast and works well when free list is maintained in order. The code returns the first suitable free block found. This strategy, combined with coalescing, helps keep fragmentation low without heavy computation. There’s also support for querying the heap state (functions like `GetLargestFreeBlockSize()` and `ShowFreeBlocks()`) so developers can monitor memory usage and fragmentation at runtime.

- **Descriptor Pool (Optional)**:  
  The initializer takes a `numDescriptors` parameter which could be used to pre-reserve a pool of `MemoryBlock` descriptors. In the current implementation, the HeapManager doesn’t explicitly allocate a descriptor array; instead, it creates `MemoryBlock` structs on the fly in free memory as needed.  The comment in `main.cpp` suggests a descriptor pool might be an optional optimization (to avoid using the managed heap for metadata at all)[GitHub](https://github.com/FutureWayne/Memory-Allocator/blob/399d2b229e25a78442369a9c962a2b9ef9f954b0/main.cpp), but the current design effectively inlines metadata into the free space.

---

## Code Highlights

### Noteworthy Techniques

- **Multiple Allocator Strategy**:  
  The integration of fixed-size allocators with a general heap is implemented by overriding the global `malloc/free` functions.  The custom `malloc` checks each FixedSizeAllocator and uses the first one large enough to handle the request; if none qualify, it delegates to the HeapManager. Similarly, `free` determines which allocator owns the pointer and frees it in the appropriate pool or heap. This design cleanly abstracts the two-tier allocator to the user – a caller can simply use `malloc` or `new` and not worry whether the memory came from a pool or the heap. It’s a good example of customizing global operators in C++ to plug in a custom system transparently

- **In-Place Memory Management**:  
  The allocator avoids external dependencies by constructing all its control structures inside the provided memory buffer. For instance, when splitting a free block, the `createNewBlock` function uses pointer arithmetic to place a new `MemoryBlock` struct at the start of the newly allocated portion. The `MemoryBlock::pBaseAddress` then points to the usable memory right after that header. This in-place approach means the allocator has zero reliance on the default heap for its bookkeeping – all metadata lives in the same memory region it manages. It’s an efficient low-level technique that ensures the custom allocator is self-contained.

- **Pointer Arithmetic and Alignment**:  
  The code makes heavy use of pointer arithmetic for efficiency and alignment. Utility inline functions like `PointerAdd` and `PointerSub` abstract the casting and arithmetic on `void*` pointers.  Alignment calculations use bitwise operations – for example, computing the required padding so that `(baseAddress + padding)` satisfies the alignment constraint (using the formula `(alignment - (addr mod alignment)) mod alignment`). This demonstrates a solid understanding of how to work with memory addresses in C++ and ensure aligned allocations for performance (critical when dealing with SIMD, cache lines, or platform requirements in games).

- **BitArray for Fast Lookup**:  
  The custom `BitArray` class in the Utilities module provides methods to set, clear, and find bits efficiently. The FixedSizeAllocator leverages this to quickly find free blocks. In the code, a simple loop checks each bit (which is already fast for small block counts), but the BitArray also has methods like `FindFirstClearBit` that could be used to further optimize this. Using a bit array instead of, say, a linked list of free nodes, minimizes memory overhead and can potentially utilize bit-level parallelism (some operations might be optimized with CPU bit scan instructions). This is an idiomatic low-level optimization often used in game engines for allocating from fixed pools.

- **Memory Safety Checks**:  
  Beyond guardbands, the code is fortified with runtime assertions (`assert`) in debug mode to catch errors and invalid usage. For example, it asserts that pointers passed to Free belong to the heap before attempting to free,  and that allocations requested are not zero-size, etc. The guardband feature itself checks that the known pattern is intact on both ends of a block during deallocation, which helps detect buffer overruns early. These defensive programming practices are important in systems programming and demonstrate attention to reliability – a crucial trait for engine-level code that gameplay programmers might write or maintain.

- **Debugging and Leak Detection**:  
  The project integrates with the MSVC debug heap to detect memory leaks – for instance, the `main.cpp` enables `_CRTDBG_MAP_ALLOC` and calls `_CrtDumpMemoryLeaks()` at the end of execution. Additionally, because global `new` and `delete` are overridden, any usage of `new` in the code (such as the test at the end of `MemorySystem_UnitTest`) will route through the custom allocator. This means the entire program’s dynamic memory usage is under the purview of the custom system, making it easier to track and verify that no leaks occur (since the custom allocator can report if all memory was freed). The inclusion of unit tests (for the memory system, bit array, and fixed allocator) in `main.cpp` is also a highlight – it shows thorough verification of the allocator’s correctness under various scenarios (random allocations, frees, coalescing checks, etc.)

---

## Usage Instructions

### Build the Project

- Open `Memory Allocator.sln` in Visual Studio (or compile with a C++17-compatible compiler on Windows).
- Debug mode enables guardbands and leak checks.
- Executable runs unit tests on launch.

### Initialize Memory System

```cpp
InitializeMemorySystem(void* pHeapMemory, size_t heapSize, unsigned numDescriptors);
```

- Allocate a memory block (e.g. via `HeapAlloc`).
- Set up pools and heap manager inside the block.

### Allocate Memory

- Use `malloc`, `new`, or the allocator's own `Alloc()` functions.
- Allocations are automatically routed through the system.
- Global pointers: `g_pHeapManager`, `g_pFixedSizeAllocators`.

### Free Memory

- Use `free` or `delete`.
- Freed memory returns to correct pool/heap.
- Optional: call `Collect()` to merge free blocks.

### Shutdown

```cpp
DestroyMemorySystem();
```

- Cleans up allocators.
- Releases base memory block (e.g., via `HeapFree`).

### Run & Test

- Unit tests validate allocations, frees, and garbage collection.
- Ready to embed into a game engine or project.

---

## Potential Extensions

- **Game Engine Integration**:  
  Integrate with Unreal/Unity systems and profilers.

- **Memory Tracking & Profiling**:  
  Track allocation stats, group by gameplay system, expose via debug UI.

- **Thread-Safety**:  
  Add locks or thread-local allocators. Explore lock-free free list algorithms.

- **Additional Allocator Types**:  
  Stack Allocator (LIFO), Buddy Allocator, etc.

- **Dynamic Resize/Customization**:  
  Allow runtime heap growth or runtime-configured pool sizes.

- **Garbage Collection Strategies**:  
  Periodic or background `Collect()` execution to improve large block availability.

---

