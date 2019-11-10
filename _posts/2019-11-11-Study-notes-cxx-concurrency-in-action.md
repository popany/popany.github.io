---
layout: post
title: Study notes - C++ Concurrency in Action
---

## 1.  x

## 2.  x

## 3.  x

## 4.  x

## 5. The C++ memory model and operations on atomic types

> _Whatever its type, an object is stored in one or more memory locations. Each __memory location__ is either an object (or sub-object) of a scalar type such as unsigned short or my_class* or a sequence of adjacent bit fields. If you use bit fields, this is an important point to note: though adjacent bit fields are distinct objects, they’re still counted as the same memory location. (Zero-length bit field rates memory locations, but doesn't have a memory location itself.)
)_  
>
> _There are four important things to take away from this:_
>
> * _Every variable is an object, including those that are members of other objects._  
> * _Every object occupies at least one memory location._  
> * _Variables of fundamental types such as int or char occupy exactly one memory location, whatever their size, even if they’re adjacent or part of an array._  
> * _Adjacent bit fields are part of the same memory location._  
