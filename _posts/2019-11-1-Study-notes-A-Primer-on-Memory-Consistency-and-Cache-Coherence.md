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

## 3. Memory Consistency Motivation and Sequential Consistency

> _"Sequential consistency was first formalized by Lamport. Lamport first called __a single processor (core) sequential__ if “the result of an execution is the same as if the operations had been executed in the order specified by the program.” He then called __a multiprocessor sequentially consistent__ if “the result of any execution is the same as if the operations of all processors (cores) were executed in some sequential order, and the operations of each individual processor (core) appear in this sequence in the order specified by its program.” This total order of operations is called __memory order__. In SC, memory order respects each core’s program order, but other consistency models may permit memory orders that do not always respect the program orders."_  
> _"Coherence applies on a per-block basis, whereas consistency is defined across all blocks."_  
> _"We adopt the formalism of Weaver and Germond with the following notation: L(a) and S(a) represent a load and a store, respectively, to address a. Orders <p and <m define program and global memory order, respectively. Program order <p is a per-core total order that captures the order in which each core logically (sequentially) executes memory operations. Global memory order <m is a total order on the memory operations of all cores."_  

The preconditon of global memory order is the assumption that the operations of all processors (cores) were executed in some sequential order.  

> _"An __SC execution__ requires:_
>  
> 1. _All cores insert their loads and stores into the order <m respecting their program order, regardless of whether they are to the same or different addresses (i.e., a=b or a≠b). There are four cases:_  
>
>    * _If L(a) <p L(b) ⇒ L(a) <m L(b) /`*` Load→Load `*`/_  
>    * _If L(a) <p S(b) ⇒ L(a) <m S(b) /`*` Load→Store `*`/_  
>    * _If S(a) <p S(b) ⇒ S(a) <m S(b) /`*` Store→Store `*`/_  
>    * _If S(a) <p L(b) ⇒ S(a) <m L(b) /`*` Store→Load `*`/_  
>
> 2. _Every load gets its value from the last store before it (in global memory order) to the same address:_  
> _Value of L(a) = Value of $MAX_{<m} \{S(a) | S(a) <m L(a)\}$, where $MAX_{<m}$ denotes “latest in memory order.”_  
>
> _"Each execution of a test-and-set instruction, for example, requires that the load for the test and the store for the set logically appear consecutively in the memory order (i.e., no other memory operations for __the same or different__ addresses interpose between them)."_  
> _"An __SC implementation__ permits only __SC executions__."_  
> _"At minimum, an SC implementation should permit __at least one__ SC execution for every program."_  
> _"To write multithreaded code, a programmer needs to be able to synchronize the threads, and such synchronization often involves atomically performing __pairs of__ operations. This functionality is provided by instructions that atomically perform a “read–modify–write” (RMW), such as the wellknown “test-and-set,” “fetch-and-increment,” and “compare-and-swap.”"_  

## 4. Total Store Order and the x86 Memory Model
