---
layout: post
title: "PSA: Enable <code>-fvisibility-inlines-hidden</code> in your shared libraries to avoid subtle bugs"
tags: C++ visibility
toc: true
---

Me and my colleague have just resolved a subtle bug that has been bothering us for months, that was caused by not using this flag. This post summarizes what we found.

I'm suggesting this flag in addition to `-fvisibility=hidden`, not to replace it.

Obligatory disclaimer that I don't consider myself an expert on the topic, so if I got something wrong, please tell me [here](https://github.com/HolyBlackCat/holyblackcat.github.io/issues) or elsewhere.

<details><summary>TL;DR</summary>

If a public class in your library is marked <code>__attribute__((__visibility__("default")))</code> as it should (for <code>typeid</code> and <code>dynamic_cast</code> to work correctly across shared library boundaries), and if it has <code>inline</code> functions, then those <code>inline</code> functions will be exported and merged across shared libraries. This happens regardless of <code>-fvisibility=hidden</code>.

This means the behavior of your compiled library can change depending on what other libraries are doing (different compiler optimizations, different defines, etc).

Use <code>-fvisibility-inlines-hidden</code> to override this behavior, but beware that the addresses of inline functions will compare unequal across shared library boundaries (this can be explicitly reverted for individual functions using the same visibility attribute).

</details>

## How shared library exports are annotated in the source code

[Skip if you already know this.](#the-bug)

### On MSVC

On MSVC, functions in shared libraries must be marked `__declspec(dllexport)` when building the library, and `__declspec(dllimport)` when consuming it, for example:

```cpp
// `a.cpp` compiled into `a.dll`.
#include <iostream>

__declspec(dllexport) void foo()
{
    std::cout << "Hello!\n";
}
```
```cpp
// `b.cpp` compiled into an executable, linked against `a.dll`.
__declspec(dllimport) int foo();

int main()
{
    foo();
}
```
This is usually done with a macro:
```cpp
// `a.h`
#pragma once

#ifdef _WIN32
#  ifdef A_BUILD
#    define A_API __declspec(dllexport)
#  else
#    define A_API __declspec(dllimport)
#  endif
#else
#  define A_API
#endif

A_API void foo();
```
```cpp
// `a.cpp`
#define A_BUILD
#include "a.h"
#include <iostream>

void foo()
{
    std::cout << "Hello!\n";
}
```


### On Linux

On Linux, everything is imported/exported by default.

But there's the `-fvisibility=hidden` flag that makes export opt-in, making the behavior similar to that of MSVC. Most projects seem to be using it (see the [GCC Wiki](https://gcc.gnu.org/wiki/Visibility) for why this is a good idea).

When using `-fvisibility=hidden`, exported functions have to be marked `__attribute__((__visibility__("default")))`.

We can add it to the macro from earlier:

```cpp
// `a.h`
#pragma once

#ifdef _WIN32
#  ifdef A_BUILD
#    define A_API __declspec(dllexport)
#  else
#    define A_API __declspec(dllimport)
#  endif
#else
#  define A_API __attribute__((__visibility__("default")))
#endif

A_API void foo();
```
```cpp
// `a.cpp`
#define A_BUILD
#include "a.h"
#include <iostream>

void foo()
{
    std::cout << "Hello!\n";
}
```

Interestingly, this attribute seems to only be necessary on definitions, but it's easier to keep it on declarations for consistency with MSVC.

### On MinGW

MinGW also exports/imports all functions by default, and this can similarly be disabled with `-fvisibility=hidden`. It understands both annotation styles, so the macro above works with both `#ifdef _WIN32` (which includes MinGW) and `#ifdef _MSC_VER` (which excludes MinGW).

## Classes

When exporting classes from shared libraries, it's possible to either export the individual member functions:

```cpp
struct MyClass
{
    A_API void foo();
    A_API void bar();
};
```

Or the entire class:

```cpp
struct A_API MyClass
{
    void foo();
    void bar();
};
```

On Windows (both MSVC and MinGW), the two approaches seem to be equivalent. (Except that MSVC with `/W2` warns on the class-wide annotation if you use member variables of class types, and those classes themselves don't have the same class-wide annotation. This seems to be purely a matter of programmer intent, since there's otherwise no difference in behavior.)

On Linux, they are **not** equivalent. The attribute on the entire class is needed to export some additional metadata, such as the vtable, type information for `typeid`, etc. (That is, when `-fvisibility=hidden` is used.) **Forgetting the attribute on the whole class can lead to issues across shared library boundaries**, such as `dynamic_cast` failing incorrectly, or the result of `typeid(T)` comparing unequal incorrectly (when one shared library creates a class instance or uses `typeid`, and another tries to cast the instance, or compare the `std::type_info`; Clang's libc++ is especially prone to this, requiring the attribute even on `enum`s if `typeid` is applied to them).

Therefore, properly written libraries, even header-only ones, will have `__attribute__((__visibility__("default)))` on every public `struct`/`class`/`union`/`enum` that they define, in case the user tries to use it across shared library boundaries. (This is on Linux. Exporting whole classes isn't needed on MSVC, as explained above.)

## The bug

Now our bug.

We have shared library `A` with following contents.
```cpp
// a.h
#pragma once

// Not thinking about Windows for now.
#define A_API   __attribute__((__visibility__("default")))
#define A_CLASS __attribute__((__visibility__("default")))

A_API void Foo();

struct A_CLASS Vec3
{
    float x, y, z;

    float LengthSq() const
    {
        return x*x + y*y + z*z;
    }
};
```
```cpp
// a.cpp
#include "a.h"
#include <print>

void Foo()
{
    std::println("{}", Vec3{-0.02146666, 0.0014069901, 0.0014069926}.LengthSq());
}
```

Since `Vec3` is entirely header-only, it could be a part of a separate header-only library or a part of A, it doesn't change anything.

Since this is a well-behaved library, `struct Vec3` is exported (see previous section).

Library `A` is compiled with GCC with `-fvisibility=hidden`.

This is already bugged. Can you spot the bug?

### The symptoms

Once the library above is compiled, `Foo()` can print different values depending on who calls it.

On Arm 64, if used from an executable (or another library) compiled with the same GCC, it prints `0.0004647767`. But if used from an executable (or library) compiled with Clang, it instead prints `0.00046477673`.

Is this a `std::println()` quirk? No, `std::bit_cast<std::uint32_t>(...)` also returns different values: `0x39f3ad46` vs `0x39f3ad47`.

A floating-point quirk? Yes, adding `-ffp-contract=off` everywhere to enfore strict math makes this consistent.

But still, how do we see different results without recompiling the library that implements `Foo`?

You can reproduce this yourself on less obscure platforms by adding an `#ifdef`:
```cpp
// ...

float LengthSq() const
{
    #ifdef FOO
    return 42;
    #else
    return x*x + y*y + z*z;
    #endif
}
```
Then define the macro in the executable but not the library, or vice versa, and observe the library behavior changing to match the define used in the exectuable.

### The answer

Turns out that `__attribute__((__visibility__("default")))` on `Vec3` overrides `-fvisibility=hidden` for all members of the class, even for inline functions.

Since the function is `inline`, it doesn't error on duplicate definitions resulting from this, and instead **merges the definitions, choosing one of them**.

The same thing can happen on MSVC if the entire class or `LengthSq` is imported/exported (but then the library owning `struct Vec3` can't be header-only, since you need something to export the class).

### How to fix this?

On Linux **`-fvisibility-inlines-hidden`** solves this. (Add it to `-fvisibility=hidden` you should already have.) [GCC Wiki](https://gcc.gnu.org/wiki/Visibility) says that it reduces the binary size as well.

The downside is that it makes the addressed of those `inline` member functions compare unequal across shared library boundaries. You can counteract this for individual `inline` functions by marking them `__attribute__((__visibility__("default")))`.

Or instead of this flag, you can manually hide individual functions with <code>__attribute__((__visibility__(<b>"hidden"</b>)))</code>. But the flag seems more useful, since comparing addresses of inline functions should be rare.

Further, as a library implementer, you must assume that your users use this flag, so functions that need stable addresses need to be explicitly exported *anyway*, and at that point you might just use the same flag yourself.

MSVC [doesn't have an equivalent flag](https://developercommunity.visualstudio.com/t/implement-zcdllexportinlines-aka-fvisibility-inlin/374754), and seemingly doesn't allow "unexporting" functions if the entire class is already exported, so the solution there is to never export classes with `inline` functions, and instead export their individual member functions. There's no reason to export entire classes there, only exporting functions is enough.

I must also mention [`-fvisibility-ms-compat`](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-fvisibility-ms-compat), which *in theory* emulates MSVC behavior on GCC/Clang (so you can skip exporting classes and only export their member functions), which is less error-prone (you can never forget to export a class, silently resulting in wrong behavior of `dynamic_cast` or `typeid`). It seems to work in Clang, but is [**bugged in GCC**](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=55958), so sadly unusable. Also, as a library implementer, forcing your users to use this flag to get the correct behavior is sketchy.

## How to write shared libraries?

So, knowing all this, how do you write shared libraries "properly", in a way that avoids those issues?

Make two macros, one for exporting functions and another for classes:

```cpp
#ifdef _WIN32
#  ifdef MYLIB_BUILD
#    define MYLIB_API __declspec(dllexport)
#  else
#    define MYLIB_API __declspec(dllimport)
#  endif
#  define MYLIB_CLASS
#else
#  define MYLIB_API   __attribute__((__visibility__("default")))
#  define MYLIB_CLASS __attribute__((__visibility__("default")))
#endif
```

Use `MYLIB_API` to export non-`inline` free functions and non-`inline` member functions, and those `inline` functions that need stable addresses across library boundaries.

Use `MYLIB_CLASS` on every `class`/`struct`/`union`/`enum` that the user can interact with. Forgetting to do this, even if it doesn't immediately error, can lead to silently failing `dynamic_cast` and `typeid` comparing unequal for the same type, at least on libc++.

&nbsp;

Thanks for reading!
