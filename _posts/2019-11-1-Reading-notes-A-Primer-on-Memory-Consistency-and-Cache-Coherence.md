---
layout: post
title: Reading notes - A Primer on Memory Consistency and Cache Coherence
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

