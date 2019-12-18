---
layout: post
title: Learn - A Primer on Memory Consistency and Cache Coherence
---

## 1. Introduction to Consistency and Coherence

> _We and many others find it useful to separate shared memory correctness into two sub-issues: consistency and coherence._  
>
> _It is the job of consistency (memory consistency, memory consistency model, or memory model) to define shared memory correctness._  
>
> _The correctness criterion for a single processor core partitions behavior between one correct result and many incorrect alternatives. This is because the processor’s architecture mandates that the execution of a thread transforms a given input state into a single well-defined output state, even on an out-of-order core. Shared memory consistency models, however, concern the loads and stores of multiple threads and usually allow many correct executions while disallowing many (more) incorrect ones._  

According to the program order of memory loads and stores, memory consistency model defines several actual orders which are all correct.  

Different memory consistency models define different actual orders.  

The execution of a single thread can only have one correct result, even if the memory operator order can be more then one, considering the situation of out-of-order core.  

The exectution of multiple threads, which share memory, can have more then one correct result.  

> _Unlike consistency, coherence (or cache coherence) is neither visible to software nor required._  
>
> _Coherence seeks to make the caches of
a shared-memory system as functionally invisible as the caches in a single-core system._  
>
> _Consistency models define correct shared memory behavior in terms of loads and stores (memory
reads and writes), without reference to caches or coherence._  
>
> _Access to stale data (incoherence) is prevented using a coherence protocol, which is a set of rules implemented by the distributed set of actors within a system._  

## 2. Coherence Basics

> _To prevent incoherence, the system must implement a cache coherence protocol to regulate the actions of the cores such that Core 2 cannot observe the old value of 42 at the same time that Core 1 observes the value 43._  
>
> _The basis of our preferred definition of coherence is the single-writer–multiple-reader (SWMR)
invariant._  
>
> _Another way to view this definition is to consider, for each memory location, that the memory location’s lifetime is divided up into epochs. In each epoch, either a single core has read–write access or some number of cores (possibly zero) have read-only access._  
>
> _In addition to the SWMR invariant, coherence requires that the value of a given memory location is propagated correctly._  
>
> _Thus, the definition of coherence must augment the SWMR invariant with a data value invariant that pertains to how values are propagated from one epoch to the next. This invariant states that the value of a memory location at the start of an epoch is the same as the value of the memory location at the end of its last read–write epoch._  

"the value of a memory location" means the value stored on a memory location.

> _A core can perform loads and stores at various granularities, often ranging from 1 to 64 bytes. In theory, coherence could be performed at the finest load/store granularity_  
>
> _Coherence pertains to all storage structures that hold blocks from the shared address space. These structures include the L1 data cache, L2 cache, shared last-level cache (LLC), and main memory. These structures also include the L1 instruction cache and translation lookaside buffers (TLBs)._  
>
> _Coherence does not pertain to the architecture (i.e., coherence is not architecturally visible). Strictly speaking, a system could be incoherent and still be correct if it adhered to the specified memory consistency model._  

## 3. Memory Consistency Motivation and Sequential Consistency

> _Sequential consistency was first formalized by Lamport. Lamport first called __a single processor (core) sequential__ if “the result of an execution is the same as if the operations had been executed in the order specified by the program.” He then called __a multiprocessor sequentially consistent__ if “the result of any execution is the same as if the operations of all processors (cores) were executed in some sequential order, and the operations of each individual processor (core) appear in this sequence in the order specified by its program.” This total order of operations is called __memory order__. In SC, memory order respects each core’s program order, but other consistency models may permit memory orders that do not always respect the program orders._  
>
> _Coherence applies on a per-block basis, whereas consistency is defined across all blocks._  
>
> _We adopt the formalism of Weaver and Germond with the following notation: L(a) and S(a) represent a load and a store, respectively, to address a. Orders <p and <m define program and global memory order, respectively. Program order <p is a per-core total order that captures the order in which each core logically (sequentially) executes memory operations. Global memory order <m is a total order on the memory operations of all cores._  

The preconditon of global memory order is the assumption that the operations of all processors (cores) were executed in some sequential order.  

> _An __SC execution__ requires:_
>  
> 1. _All cores insert their loads and stores into the order <m respecting their program order, regardless of whether they are to the same or different addresses (i.e., a=b or a≠b). There are four cases:_  
>
>    * _If L(a) <p L(b) ⇒ L(a) <m L(b) /`*` Load→Load `*`/_  
>    * _If L(a) <p S(b) ⇒ L(a) <m S(b) /`*` Load→Store `*`/_  
>    * _If S(a) <p S(b) ⇒ S(a) <m S(b) /`*` Store→Store `*`/_  
>    * _If S(a) <p L(b) ⇒ S(a) <m L(b) /`*` Store→Load `*`/_  
>
> 2. _Every load gets its value from the last store before it (in global memory order) to the same address:_  
> _Value of L(a) = Value of $MAX_{<m} \{S(a) | S(a) <m\ L(a)\}$, where $MAX_{<m}$ denotes “latest in memory order.”_  
>
> _Each execution of a test-and-set instruction, for example, requires that the load for the test and the store for the set logically appear consecutively in the memory order (i.e., no other memory operations for __the same or different__ addresses interpose between them)._  
>
> _An __SC implementation__ permits only __SC executions__._  
>
> _At minimum, an SC implementation should permit __at least one__ SC execution for every program._  
>
> _To write multithreaded code, a programmer needs to be able to synchronize the threads, and such synchronization often involves atomically performing __pairs of__ operations. This functionality is provided by instructions that atomically perform a “read–modify–write” (RMW), such as the wellknown “test-and-set,” “fetch-and-increment,” and “compare-and-swap.”_  

## 4. Total Store Order and the x86 Memory Model

> _A widely implemented memory consistency model is total store order (TSO). TSO is used in SPARC implementations and, more importantly, appears to match the memory consistency model of the widely used x86 architecture._  
>
> _Processor cores have long used write (store) buffers to hold committed (retired) stores until the rest of the memory system could process the stores._  
>
> _For a single-core processor, a write buffer can be made __architecturally invisible__ by ensuring that a load to address A returns the value of the most recent store to A even if one or more stores to A are in the write buffer. This is typically done by either __bypassing__ the value of the most recent store to A to the load from A, where “most recent” is determined by __program order__, or by stalling a load of A if a store to A is in the write buffer._  
>
> _In fact, to preserve single-thread sequential semantics, each core must see the effect of its __own store__ in __program order__, even though the store is not yet observed by other cores._  
>
> _An __TSO execution__ requires:_
>  
> 1. _Program oder v.s. memory order:_  
>
>    * _If L(a) <p L(b) ⇒ L(a) <m L(b) /`*` Load→Load `*`/_  
>    * _If L(a) <p S(b) ⇒ L(a) <m S(b) /`*` Load→Store `*`/_  
>    * _If S(a) <p S(b) ⇒ S(a) <m S(b) /`*` Store→Store `*`/_  
>    * _If S(a) <p FENCE ⇒ S(a) <m FENCE /`*` Store → FENCE `*`/_  
>    * _If FENCE <p L(a) ⇒ FENCE <m L(a) /`*` FENCE → Load  `*`/_  
>
> 2. _Load value:_  
> _Value of L(a) = Value of $MAX_{<m} \{S(a) | S(a) <m\  L(a) \ or\  S(a) <p\ L(a)\}$_  
>
> _The implementation story for TSO/x86 is similar to SC with the addition of per-core FIFO write buffers._  
>
> _The semantics of the FENCE specify that all instructions before the FENCE in program order must be ordered before any instructions after the FENCE in program order._  

## 5. Relaxed Memory Consistency

> _For teaching purposes, this section introduces an eXample relaxed Consistency model (XC) that captures the basic idea and some implementation potential of relaxed memory consistency models. XC assumes that a __global memory order exists__, as is true for the strong models of SC and TSO, as well as the largely defunct relaxed models for Alpha and SPARC Relaxed Memory Order (RMO)._  
>
> XC provides a FENCE instruction so that programmers can indicate when order is needed; otherwise, by default, loads and stores are unordered.  
>
> A FENCE instruction does not specify an address. Two FENCEs by the same core also stay ordered. However, a FENCE __does not affect the order of memory operations at other cores__ (which is why “fence” may be a better name than “barrier”).  
> _XC’s memory order is guaranteed to respect (preserve) program order for:_  
>
> * _Load → FENCE_  
> * _Store → FENCE_  
> * _FENCE → FENCE_  
> * _FENCE → Load_  
> * _FENCE → Store_  
>
> _XC maintains TSO rules for ordering two accesses to the same address only:_  
>
> * _Load → Load to same address_  
> * _Load → Store to the same address_  
> * _Store → Store to the same address_  
>
> _These rules enforce the __sequential processor model__ (i.e., sequential core semantics) and prohibit behaviors that might astonish programmers._  
>
> _XC ensures that loads immediately see updates due to their own stores (like TSO's write buffer bypassing). This rule preserves the sequentiality of single threads, also to avoid programmer astonishment._  
>
> _Here, we formalize XC in a manner consistent with the previous two chapters’ notation and approach. Once again, let L(a) and S(a) represent a load and a store, respectively, to address a. Orders <p and <m define per-processor program order and global memory order, respectively. Program order <p is a per-processor total order that captures the order in which each core logically (sequentially) executes memory operations. Global memory order <m is a total order on the memory of all cores._  
>
> _More formally, an __XC execution__ requires the following:_
>
> 1. _All cores insert their loads, stores, and FENCEs into the order <m respecting:_  
>
>     * _If L(a) <p FENCE ⇒ L(a) <m FENCE /`*` Load → FENCE `*`/_  
>     * _If S(a) <p FENCE ⇒ S(a) <m FENCE /`*` Store → FENCE `*`/_  
>     * _If FENCE <p FENCE ⇒ FENCE <m FENCE /`*` FENCE → FENCE `*`/_  
>     * _If FENCE <p L(a) ⇒ FENCE <m L(a) /`*` FENCE → Load `*`/_  
>     * _If FENCE <p S(a) ⇒ FENCE <m S(a) /`*` FENCE → Store `*`/_  
>
> 2. _All cores insert their loads and stores to the same address into the order <m respecting:_  
>
>    * _If L(a) <p L'(a) ⇒ L(a) <m L' (a) /`*` Load → Load to same address `*`/_  
>    * _If L(a) <p S(a) ⇒ L(a) <m S(a) /`*` Load → Store to same address  `*`/_  
>    * _If S(a) <p S'(a) ⇒ S(a) <m S' (a) /`*` Store → Store to same address  `*`/_  
>
> 3. _Every load gets its value from the last store before it to the same address:_  
> _Value of L(a) = Value of $MAX_{<m} \{S(a) | S(a) <m\  L(a) \ or\  S(a) <p\ L(a)\}$_  
>
> _With TSO, the atomic RMW is used to attempt to acquire the lock, and a store is used to release the lock. With XC, the situation is more complicated. For the acquire, XC does not, by default, constrain the RMW from being reordered with respect to the operations in the critical section. To avoid this situation, a __lock acquire must be followed by a FENCE__. Similarly, the lock release is not, by default, constrained from being reordered with respect to the operations before it in the critical section. To avoid this situation, a __lock release must be preceded by a FENCE__._  
>
> _SC for DRF programs asks programmers to ensure programs are DRF under SC and asks implementors to ensure that all executions of DRF programs on the relaxed model are also SC executions. XC and, as far as we know, all commercial relaxed memory models support SC for DRF programs._  
>
> _A more concrete understanding of “SC for DRF” requires some definitions:_  
>
> * _Some memory operations are tagged as synchronization (“synchronization operations”), while the rest are tagged data by default (“data operations”). Synchronization operations include lock acquires and releases._
> * _Two data operations Di and Dj conflict if they are from different cores (threads) (i.e., __not ordered by program order__), access the same memory location, and at least one is a store._  
> * _Two synchronization operations Si and Sj conflict if they are from different cores (threads), access the same memory location (e.g., the same lock), and the two synchronization operations are not compatible (e.g., acquire and release of a spinlock are not compatible, whereas two read_locks on a reader–writer lock are compatible)._  
> * _Two synchronization operations Si and Sj transitively conflict if either Si and Sj conflict or if Si conflicts with some synchronization operation Sk, Sk <p Sk' (i.e., Sk is earlier than Sk' in a core K’s program order), and Sk' transitively conflicts with Sj._  
> * _Two data operations Di and Dj race if they conflict and they appear in the global memory order without an intervening pair of transitively conflicting synchronization operations by the same cores (threads) i and j. In other words, a pair of conflicting data operations Di <m Dj are not a data race if and only if there exists a pair of transitively conflicting synchronization operations Si and Sj such that Di <m Si <m Sj <m Dj._  
> * _An SC execution is __data-race-free (DRF)__ if no data operations race._  
> * _A program is DRF if all its SC executions are DRF._  
> * _A memory consistency model supports “__SC for DRF__ programs” if all executions of all DRF programs are __SC executions__. This support usually requires some special actions for synchronization operations._  
>
> _With FENCEs around synchronization operations, XC supports __SC for DRF__ programs._  
>
> _Supporting SC for DRF programs allows many programmers to reason about their programs with SC and not the more complex rules of XC and, at the same time, benefit from any hardware performance improvements or simplifications XC enables over SC._  
>
> _To allow synchronization races but not data races, programmers must tag variables as synchronization when it is possible they can race, using keywords such as volatile and atomic, or create synchronization locks implicitly with Java’s monitor-like synchronized methods. In all cases, implementations are free to reorder references as long as data-race-free programs obey SC._  
>
> _Thus, HLLs, such as Java and C++, adopt the relaxed memory model approach of “SC for DRF.” When these HLLs run on hardware, should the hardware’s memory model also be relaxed? On one hand, (a) relaxed hardware can give the most performance, and (b) compilers and runtime software need only translate the HLL’s synchronizations operations into the particular hardware’s low-level synchronization operations and FENCEs to provide the necessary ordering. On the other hand, (a) SC and TSO can give good performance, and (b) compilers and runtime software can generate more portable code without FENCEs from incomparable memory models. Although the debate is not settled, it is clear that relaxed __HLL models do not require relaxed hardware__._  
