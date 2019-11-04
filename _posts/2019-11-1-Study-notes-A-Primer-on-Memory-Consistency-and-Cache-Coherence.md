---
layout: post
title: Study notes - A Primer on Memory Consistency and Cache Coherence
---

## 1. Introduction to Consistency and Coherence

> _"We and many others find it useful to separate shared memory correctness into two sub-issues: consistency and coherence."_  
> _"It is the job of consistency (memory consistency, memory consistency model, or memory model) to define shared memory correctness."_  
> _"The correctness criterion for a single processor core partitions behavior between one correct result and many incorrect alternatives. This is because the processor’s architecture mandates that the execution of a thread transforms a given input state into a single well-defined output state, even on an out-of-order core. Shared memory consistency models, however, concern the loads and stores of multiple threads and usually allow many correct executions while disallowing many (more) incorrect ones. "_  

According to the program order of memory loads and stores, memory consistency model defines several actual orders which are all correct.  
Different memory consistency models define different actual orders.  
The execution of a single thread can only have one correct result, even if the memory operator order can be more then one, considering the situation of out-of-order core.  
The exectution of multiple threads, which share memory, can have more then one correct result.  

> _"Unlike consistency, coherence (or cache coherence) is neither visible to software nor required."_  
> _"Coherence seeks to make the caches of
a shared-memory system as functionally invisible as the caches in a single-core system."_  
> _"Consistency models define correct shared memory behavior in terms of loads and stores (memory
reads and writes), without reference to caches or coherence. "_  
> _"Access to stale data (incoherence) is prevented using a coherence protocol, which is a set of rules implemented by the distributed set of actors within a system."_  

## 2. Coherence Basics

> _"To prevent incoherence, the system must implement a cache coherence protocol to regulate the actions of the cores such that Core 2 cannot observe the old value of 42 at the same time that Core 1 observes the value 43."_  
> _"The basis of our preferred definition of coherence is the single-writer–multiple-reader (SWMR)
invariant. "_  
> _"Another way to view this definition is to consider, for each memory location, that the memory location’s lifetime is divided up into epochs. In each epoch, either a single core has read–write access or some number of cores (possibly zero) have read-only access. "_  
> _"In addition to the SWMR invariant, coherence requires that the value of a given memory location is propagated correctly."_  
> _"Thus, the definition of coherence must augment the SWMR invariant with a data value invariant that pertains to how values are propagated from one epoch to the next. This invariant states that the value of a memory location at the start of an epoch is the same as the value of the memory location at the end of its last read–write epoch. "_  

"the value of a memory location" means the value stored on a memory location.

> _"A core can perform loads and stores at various granularities, often ranging from 1 to 64 bytes. In theory, coherence could be performed at the finest load/store granularity"_  
> _"Coherence pertains to all storage structures that hold blocks from the shared address space. These structures include the L1 data cache, L2 cache, shared last-level cache (LLC), and main memory. These structures also include the L1 instruction cache and translation lookaside buffers (TLBs)."_  
> _"Coherence does not pertain to the architecture (i.e., coherence is not architecturally visible). Strictly speaking, a system could be incoherent and still be correct if it adhered to the specified memory consistency model. "_  
