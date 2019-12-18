---
layout: post
title: Learn C#
---

## Syntax

### Dispose & Finalize

#### [IDisposable.Dispose Method](https://docs.microsoft.com/en-us/dotnet/api/system.idisposable.dispose?view=netframework-4.8)

Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.

    public void Dispose ();

Use this method to close or release unmanaged resources such as files, streams, and handles held by an instance of the class that implements this interface. By convention, this method is used for all tasks associated with freeing resources held by an object, or preparing an object for reuse.

If an object's Dispose method is called more than once, the object must ignore all calls after the first one. The object must not throw an exception if its Dispose method is called multiple times. Instance methods other than Dispose can throw an ObjectDisposedException when resources are already disposed.

Because the Dispose method must be called explicitly, there is always a danger that the unmanaged resources will not be released, because the consumer of an object fails to call its Dispose method. There are two ways to avoid this:

* Wrap the managed resource in an object derived from `System.Runtime.InteropServices.SafeHandle`. Your `Dispose` implementation then calls the `Dispose` method of the `System.Runtime.InteropServices.SafeHandle` instances.

* Implement a finalizer to free resources when `Dispose` is not called. By default, the garbage collector automatically calls an object's finalizer before reclaiming its memory. However, if the `Dispose` method has been called, it is typically unnecessary for the garbage collector to call the disposed object's finalizer. To prevent automatic finalization, `Dispose` implementations can call the `GC.SuppressFinalize` method.

When you use an object that accesses unmanaged resources, such as a `StreamWriter`, a good practice is to create the instance with a `using` statement. The using statement automatically closes the stream and calls `Dispose` on the object when the code that is using it has completed.

#### [Object.Finalize Method](https://docs.microsoft.com/en-us/dotnet/api/system.object.finalize?view=netframework-4.8)

Allows an object to try to free resources and perform other cleanup operations before it is reclaimed by garbage collection.

    ~Object ();

The Finalize method is used to perform cleanup operations on unmanaged resources held by the current object before the object is destroyed. The method is protected and therefore is accessible only through this class or through a derived class.

##### How finalization works

The `Object` class provides no implementation for the `Finalize` method, and the garbage collector does not mark types derived from `Object` for finalization unless they override the `Finalize` method.

If a type does override the `Finalize` method, the garbage collector adds an entry for each instance of the type to an internal structure called the `finalization queue`. The `finalization queue` contains entries for all the objects in the managed heap whose **finalization code** must run before the garbage collector can reclaim their memory. The garbage collector then calls the `Finalize` method automatically under the following conditions:

* After the garbage collector has discovered that an object is inaccessible, unless the object has been exempted from finalization by a call to the `GC.SuppressFinalize` method.

* On `.NET Framework` only, during **shutdown** of an application domain, unless the object is exempt from finalization. During **shutdown**, even objects that are still accessible are finalized.

`Finalize` is automatically called only once on a given instance, unless the object is re-registered by using a mechanism such as `GC.ReRegisterForFinalize` and the `GC.SuppressFinalize` method has not been subsequently called.

`Finalize` operations have the following limitations:

* The **exact time** when the finalizer executes is undefined. To ensure deterministic release of resources for instances of your class, implement a `Close` method or provide a `IDisposable.Dispose` implementation.

* The finalizers of two objects are not guaranteed to run in any **specific order**, even if one object refers to the other.

* The `thread` on which the finalizer runs is **unspecified**.

The `Finalize` method might not run to completion or might not run at all under the following **exceptional circumstances**:

* If another finalizer **blocks** indefinitely (goes into an infinite loop, tries to obtain a lock it can never obtain, and so on). Because the runtime tries to run finalizers to completion, other finalizers **might not be called** if a finalizer blocks indefinitely.

* If the process **terminates** without giving the runtime a chance to clean up. In this case, the runtime's first notification of process termination is a `DLL_PROCESS_DETACH` notification.

The runtime continues to finalize objects during shutdown only while the number of finalizable objects continues to decrease.

If `Finalize` or an override of `Finalize` throws an exception, and the runtime is not hosted by an application that overrides the default policy, the runtime terminates the process and no active `try/finally` blocks or finalizers are executed. This behavior ensures process integrity if the finalizer cannot free or destroy resources.

##### Overriding the Finalize method

You should override `Finalize` for a class that uses unmanaged resources, such as file handles or database connections that must be released when the managed object that uses them is discarded during garbage collection. You **shouldn't** implement a `Finalize` method for **managed objects** because the garbage collector releases managed resources automatically.

If a `SafeHandle` object is available that wraps your unmanaged resource, the recommended alternative is to implement the dispose pattern with a safe handle and not override `Finalize`.

The `Object.Finalize` method does nothing by default, but you should override `Finalize` **only if necessary**, and **only to release unmanaged resources**. Reclaiming memory tends to take much longer if a finalization operation runs, because it requires at least two garbage collections. In addition, you should override the `Finalize` method **for reference types only**. The common language runtime only finalizes reference types. It ignores finalizers on value types.

Every implementation of `Finalize` in a derived type must call its base type's implementation of `Finalize`. This is the **only case** in which application code is allowed to call `Finalize`. An object's `Finalize` method **shouldn't call** a method on any objects other than that of its **base class**. This is because the other objects being called could be collected at the same time as the calling object, such as in the case of a common language runtime shutdown.

The `C#` compiler does not allow you to override the `Finalize` method. Instead, you provide a finalizer by implementing a `destructor` for your class. A `C#` `destructor` **automatically calls** the destructor of its base class.

Because garbage collection is non-deterministic, you do not know precisely when the garbage collector performs finalization. To release resources immediately, you can also choose to implement the [dispose pattern](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose?view=netframework-4.8) and the `IDisposable` interface. The `IDisposable.Dispose` implementation can be called by consumers of your class to free unmanaged resources, and you can use the `Finalize` method to free unmanaged resources in the event that the `Dispose` method is not called.

`Finalize` can take almost any action, including `resurrecting` an object (that is, making the object accessible again) after it has been cleaned up during garbage collection. However, the object can only be resurrected once; Finalize cannot be called on resurrected objects during garbage collection.

##### The SafeHandle alternative

Creating reliable finalizers is often difficult, because you cannot make assumptions about the state of your application, and because unhandled system exceptions such as OutOfMemoryException and StackOverflowException terminate the finalizer. Instead of implementing a finalizer for your class to release unmanaged resources, you can use an object that is derived from the System.Runtime.InteropServices.SafeHandle class to wrap your unmanaged resources, and then implement the dispose pattern without a finalizer.

#### [using statement (C# Reference)](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/using-statement)

The `using` statement ensures that `Dispose` is called even if an exception occurs within the using block. You can achieve the same result by putting the object inside a try block and then calling Dispose in a `finally` block; in fact, this is how the using statement is translated by the compiler.

## Debug

[How to debug corruption in the managed heap](https://stackoverflow.com/questions/7064966/how-to-debug-corruption-in-the-managed-heap)

[Debugging managed code memory leak with memory dump using windbg](https://blogs.msdn.microsoft.com/tess/2006/02/09/net-crash-managed-heap-corruption-calling-unmanaged-code/)
