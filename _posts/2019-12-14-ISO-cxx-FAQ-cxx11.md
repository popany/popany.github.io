---
layout: post
title: ISO C++ FAQ - C++11
---

## [C++11 Overview](https://isocpp.org/wiki/faq/cpp11)

## [C++11 Language Extensions — General Features](https://isocpp.org/wiki/faq/cpp11-language)

### auto

### decltype

### Range-for statement

### Initializer lists

### Uniform initialization syntax and semantics

### Rvalue references and move semantics

### Lambdas

### noexcept to prevent exception propagation

If a function cannot throw an exception or if the program isn’t written to handle exceptions thrown by a function, that function can be declared `noexcept`. For example:

    extern "C" double sqrt(double) noexcept;    // will never throw

    vector<double> my_computation(const vector<double>& v) noexcept // I'm not prepared to handle memory exhaustion
    {
        vector<double> res(v.size());   // might throw
        for(int i; i<v.size(); ++i) res[i] = sqrt(v[i]);
        return res;
    }

If a function declared `noexcept` throws (so that the exception tries to escape the `noexcept` function) the program is terminated by a call to `std::terminate()`. The call of `terminate()` cannot rely on objects being in well-defined states; that is, there is no guarantee that destructors have been invoked, no guaranteed stack unwinding, and no possibility for resuming the program as if no problem had been encountered. This is deliberate and makes `noexcept` a simple, crude, and very efficient mechanism – much more efficient than the old dynamic `throw()` exception specification mechanism.

It is possible to make a function conditionally `noexcept`. For example, an algorithm can be specified to be `noexcept` if (and only if) the operations it uses on a template argument are `noexcept`:

    template<class T>
    void do_f(vector<T>& v) noexcept(noexcept(f(v.at(0)))) // can throw if f(v.at(0)) can
    {
        for(int i; i<v.size(); ++i)
            v.at(i) = f(v.at(i));
    }

Here, we first use noexcept as an operator: `noexcept(f(v.at(0)))` is true if `f(v.at(0))` can’t throw, that is if the `f()` and `at()` used are `noexcept`.

The `noexcept()` operator is a constant expression and does not evaluate its operand.

The general form of a `noexcept` declaration is `noexcept(expression)` and “plain `noexcept`” is simply a shorthand for `noexcept(true)`. All declarations of a function must have compatible `noexcept` specifications.

A destructor shouldn’t throw; a generated destructor is implicitly `noexcept` (**independently of what code is in its body**) if all of the members of its class have `noexcept` destructors (which, ahem, they too will have by default).

It is typically a bad idea to have a move operation throw, so declare those `noexcept` wherever possible. A generated copy or move operation is implicitly `noexcept` if all of the copy or move operations it uses on members of its class have `noexcept` destructors.

`noexcept` is widely and systematically used in the standard library to improve performance and clarify requirements.

> [noexcept specifier](https://en.cppreference.com/w/cpp/language/noexcept_spec)
>
> Specifies whether a function could throw exceptions.  
>
> Every function in C++ is either non-throwing or potentially throwing
>
> * potentially-throwing functions are:
>   * functions declared with noexcept specifier whose expression evaluates to false
>   * functions declared without noexcept specifier except for
>     * destructors unless the destructor of any potentially-constructed **base** or **member** is potentially-throwing
>     * default constructors, copy constructors, move constructors that are implicitly-declared or defaulted on their first declaration unless
>       * a constructor for a base or member that the implicit definition of the constructor would call is potentially-throwing
>       * a subexpression of such an initialization, such as a default argument expression, is potentially-throwing 
>       * a default member initializer (for default constructor only) is potentially-throwing
>     * copy assignment operators, move assignment operators that are implicitly-declared or defaulted on their first declaration unless the invocation of any assignment operator in the implicit definition is potentially-throwing 
>     * deallocation functions
> * non-throwing functions are all others (those with noexcept specifier whose expression evaluates to true as well as destructors, defaulted special member functions, and deallocation functions)
>
> If a virtual function is non-throwing, all declarations, including the definition, of every overrider must be non-throwing as well, unless the overrider is defined as deleted:
>
>     struct B {
>         virtual void f() noexcept;
>         virtual void g();
>         virtual void h() noexcept = delete;
>     };
>     struct D: B {
>         void f();              // ill-formed: D::f is potentially-throwing, B::f is non-throwing
>         void g() noexcept;     // OK
>         void h() = delete;     // OK
>     };
>
> Non-throwing functions are permitted to call potentially-throwing functions. Whenever an exception is thrown and the search for a handler encounters the outermost block of a non-throwing function, the function std::terminate is called:
>
>     extern void f();  // potentially-throwing
>     void g() noexcept {
>         f();      // valid, even if f throws
>         throw 42; // valid, effectively a call to std::terminate
>     }
>

> [noexcept operator](https://en.cppreference.com/w/cpp/language/noexcept)
>
> The noexcept operator performs a compile-time check that returns true if an expression is declared to not throw any exceptions.
>
> It can be used within a function template's noexcept specifier to declare that the function will throw exceptions for some types but not others.
>
> The noexcept operator does not evaluate expression.

### constexpr

### nullptr – a null pointer literal

### Copying and rethrowing exceptions

### Inline namespaces

### User-defined literals

## [C++11 Language Extensions — Classes](https://isocpp.org/wiki/faq/cpp11-language-classes)

## [C++11 Language Extensions — Other Types](https://isocpp.org/wiki/faq/cpp11-language-types)

## [C++11 Language Extensions — Templates](https://isocpp.org/wiki/faq/cpp11-language-templates)

## [C++11 Language Extensions — Concurrency](https://isocpp.org/wiki/faq/cpp11-language-concurrency)

## [C++11 Language Extensions — Miscellaneous Language Features](https://isocpp.org/wiki/faq/cpp11-language-misc)

## [C++11 Library Extensions — General Libraries](https://isocpp.org/wiki/faq/cpp11-library)

## [C++11 Library Extensions — Containers and Algorithms](https://isocpp.org/wiki/faq/cpp11-library-stl)

## [C++11 Library Extensions — Concurrency](https://isocpp.org/wiki/faq/cpp11-library-concurrency)
