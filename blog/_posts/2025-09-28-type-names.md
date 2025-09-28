---
layout: post
title: Obtaining type name strings
tags: C++ templates
toc: true
---

## TL;DR:

Getting the type names as strings can be useful: for debugging purposes, for registering classes in factories, etc.

There are two methods of obtaining those names:

1. [At runtime, using `typeid(T).name()` and `__cxa_demangle()`.](#at-runtime-using-typeidtname)
2. [At compile-time, using `std::source_location::current().function_name()`.](#at-compile-time-using-function-names)

Those are described in more detail below.

Both produce compiler-dependent names. (1) gives the same names on GCC and Clang, but MSVC uses a different format (this includes Clang when in MSVC-compatible mode). (2) gives prettier, but even more compiler-dependent names.

To get names that are more consistent across compilers, you can use my library [cppdecl](https://github.com/MeshInspector/cppdecl), which applies some heuristics to normalize those names as much as possible.

## At runtime, using `typeid(T).name()`

This is the first thing that people see when googing the topic, and in line with C++ traditions it does the wrong thing by default:

```cpp
#include <iostream>
#include <typeinfo>

int main()
{
    std::cout << typeid(int).name() << '\n';
}
```
This prints:

* `"i'` on GCC and Clang.
* `"int"` on MSVC (and on Clang when in MSVC-compatible mode; everywhere in this section about `typeid`, when I say "MSVC", I imply also Clang in MSVC-compat mode).

Is `i` the first letter of the type? No. Let's look at some other names:

* `MyClass` (assuming `class MyClass {};`) gives `"7MyClass"` on GCC/Clang, `"class MyClass"` on MSVC.
* `std::string` gives `"NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE"` on GCC/Clang and `"class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> >"` on MSVC.

What GCC and Clang print are the *mangled names*, which aren't intended to be human-readable, and are used for some internal purposes (such as for type comparisons for `dynamic_cast`s in some cases).

On MSVC, you can use the non-standard <code>typeid(T).<b>raw</b>_name()</code> to get mangled names, which use a different format. Here's one for `std::string`: `".?AV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@"`.

If you just need *some* strings e.g. as map keys (not necessarily human-readable), you might be tempted to use mangled names, but in those cases [`std::type_index`](https://en.cppreference.com/w/cpp/types/type_index.html) is usually a better option.

### Demangling the names

How do you get human-readable names on GCC and Clang? You *demangle* them.

There are command-line utilities for this:

* Clang's `llvm-cxxfilt`.
* `c++filt` from binutils.

Running `llvm-cxxfilt -t "NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE"` (or `c++filt -t ...`) prints `std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >` (`std::string` is a `typedef` for this type).

But calling those from your program just to demangle the names isn't the best idea.

GCC/Clang come with a demangling library. Include [`<cxxabi.h>`](https://gcc.gnu.org/onlinedocs/libstdc++/libstdc++-html-USERS-4.3/a01696.html) and call `abi::__cxa_demangle()`.

Here is a little wrapper class for it:
```cpp
class Demangler
{
    #ifndef _MSC_VER
    // We keep those here so that `abi::__cxa_demangle()` can reuse its buffer between calls.
    char *buf_ptr = nullptr;
    std::size_t buf_size = 0;
    #endif

  public:
    constexpr Demangler() {}

    #ifndef _MSC_VER
    Demangler(Demangler &&other) noexcept
        : buf_ptr(other.buf_ptr), buf_size(other.buf_size)
    {
        other.buf_ptr = nullptr;
        other.buf_size = 0;
    }

    Demangler &operator=(Demangler other) noexcept
    {
        // Using the copy&swap idiom.
        std::swap(buf_ptr, other.buf_ptr);
        std::swap(buf_size, other.buf_size);
        return *this;
    }

    ~Demangler()
    {
        #ifndef _MSC_VER
        std::free(buf_ptr); // Does nothing if `buf_ptr` is null.
        #endif
    }
    #endif

    // Demangles the `typeid(...).name()` if necessary, or returns `name` as is if this platform doesn't mangle the type names.
    // The resulting pointer is either owned by this `Demangler` instance (and is reused on the next call), or is just `name` as is.
    // So for portable use ensure that both the `Demangler` instance and the `name` stay alive as long as you use the resulting pointer.
    // Returns null on demangling failure, but I've never seen that happen.
    [[nodiscard]] const char *operator()(const char *name)
    {
        #ifdef _MSC_VER
        return name;
        #else
        int status = -4; // Some custom error code, in case `abi::__cxa_demangle()` doesn't modify it at all for some reason.
        buf_ptr = abi::__cxa_demangle(name, buf_ptr, &buf_size, &status);
        if (status != 0) // -1 = out of memory, -2 = invalid string, -3 = invalid usage
            return nullptr; // Unable to demangle.
        return buf_ptr;
        #endif
    }
};
```
And now you can do this:
```cpp
Demangler d;
std::cout << d(typeid(int).name()) << '\n'; // Prints `int`.
```
This class is written to return the string unchanged on MSVC, so you can use it unconditionally on all compilers.

### Demangled names on different compilers

The names are somewhat portable across compilers, but there are differences.

First of all, MSVC likes to add `struct`/`class`/`union`/`enum` prefixes. For example, for `class MyClass {};` you get:

* `"MyClass"` on GCC/Clang.
* `"class MyClass"` on MSVC.

You can easily remove those:
```cpp
// Replace `a` with `b` in `source`, return the new string.
[[nodiscard]] std::string Replace(std::string_view source, std::string_view a, std::string_view b)
{
    std::string ret;

    std::size_t cur_pos;
    std::size_t last_pos = 0;

    while ((cur_pos = source.find(a, last_pos)) != std::string::npos)
    {
        ret.append(source, last_pos, cur_pos - last_pos);
        ret += b;
        last_pos = cur_pos + a.size();
    }

    ret.append(source, last_pos);
    return ret;
}

// On MSVC, removes the prefixes `struct`/`class`/`union`/`enum` from the string. Elsewhere returns it unchanged.
[[nodiscard]] std::string AdjustType(std::string_view type)
{
    #ifdef _MSC_VER
    std::string ret = Replace(type, "class ", "");
    ret = Replace(ret, "struct ", "");
    ret = Replace(ret, "union ", "");
    ret = Replace(ret, "enum ", "");
    return ret;
    #else
    // This should be just `std::string(type)`, but Clang is bugged: https://github.com/llvm/llvm-project/issues/161071
    return std::string(type.data(), type.data() + type.size());
    #endif
}
```

And now finally, `std::cout << AdjustType(Demangler{}(typeid(MyClass).name())) << '\n';` prints `MyClass` on all compilers.

This is good enough to give portable names for simple classes, but there are still differences in more complex cases, e.g. templates. For example, `std::string` gives:

* On GCC/Clang: `"std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >"`.
* On MSVC: `"std::basic_string<char,std::char_traits<char>,std::allocator<char> >"`

As you can see, there are still some differences: `__cxx11`, and spaces after `,`s. And it's a bit sad that this doesn't just print `"std::string"`.

## At compile-time, using function names

There's a second, completely different approach to getting the names:

```cpp
#include <iostream>
#include <source_location>
#include <string_view>

template <typename T>
const char *RawTypeName()
{
    return std::source_location::current().function_name();
}

int main()
{
    std::cout << RawTypeName<int>() << '\n';
}
```
`std::source_location::current().function_name()` returns the current function name. Turns out that when used in templates, it happens to return their stringified template arguments as well, which we can extract the types from.

The above prints:

* On MSVC: `const char *__cdecl RawTypeName<int>(void)`
* On GCC: `const char* RawTypeName() [with T = int]`
* On Clang: `const char *RawTypeName() [T = int]` (notably it gives almost the same string when in MSVC-compatible mode too, which is unlike `typeid(T).name()`, where it imitates MSVC)

As you can see, `int` is included in all those strings, but in different places.

Note that `std::source_location` is a relatively new addition to the language (added in C++20). Before we had that, we used to use non-standard compiler extensions, like this:
```cpp
template <typename T>
const char *RawTypeName()
{
    #ifdef _MSC_VER
    return __FUNCSIG__;
    #else
    return __PRETTY_FUNCTION__;
    #endif
}
```
This being non-standard isn't actually a big deal, because strictly speaking `.function_name()` isn't guaranteed to include the template argument names either. For both of those, if it works in practice (it does), it's good enough.

### Extracting type names from the strings

How do we extract `"int"` from `"const char *RawTypeName() [T = int]"`?

Some libraries hardcode the amount of characters that need to be removed from those those strings (from both sides) to get just the name, but this is error-prone. I've seen at least one library that did this on GCC, Clang, and MSVC, but forgot to test Clang's behavior in MSVC-compatible mode.

A better solution is to determine the amount of characters to remove automatically.

This is simple: Take a type with a known name, such as `int`. Find this name in `RawTypeName<int>()`, remember its position. Then use this position to find the names in other strings returned by `RawTypeName<...>()`. Like this:
```cpp
class MyClass {};

int main()
{
    std::string_view int_name = RawTypeName<int>();
    std::size_t prefix_len = int_name.rfind("int");
    std::size_t suffix_len = int_name.size() - 3 - prefix_len;

    std::string_view myclass_name = RawTypeName<MyClass>();
    std::cout << myclass_name << '\n'; // Prints `const char* RawTypeName() [with T = MyClass]` in Clang.

    myclass_name.remove_prefix(prefix_len);
    myclass_name.remove_suffix(suffix_len);
    std::cout << myclass_name << '\n'; // Prints just `MyClass` or `class MyClass` in MSVC.
}
```

### Variance between compilers

The type names reported by this method can vary wildly between compilers, and even on the same compiler they can depend on which C++ standard library implementation is used.

For `int` and `class MyClass {};`, it behaves the same as the `typeid`-based approach, meaning that on MSVC only, the latter reports `"class MyClass"` (notably Clang in MSVC-compat mode omits `class`, unlike when using `typeid`).

But if we take `std::string`, more differences show up:

* GCC reports `"std::__cxx11::basic_string<char>"`. It has removed the redundant template arguments (which have their default values anyway).

* Clang normally reports `"std::basic_string<char>"`, so it has further removed the redundant namespace `__cxx11` (because it is `inline`).

  But it can also report just `"std::string"` if you're using libc++ (the Clang's own implementation of the C++ standard library, because it has special annotations on `std::string`, namely [`__attribute__((__preferred_name__(...)))`](https://stackoverflow.com/q/78922348/2752075)).

* MSVC reports `"class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> >"`, which is the exact same string as it returns from `typeid(std::string).name()`.

  So it's recommended to call `AdjustType()` from earlier on those strings.

### Extracting the type names at compile-time

The [code snippet above](#extracting-type-names-from-the-strings) can be evaluated at compile-time, and will probably be optimized by the compiler to e.g. compute `prefix_len` and `suffix_len` at compile-time. (You can even make them `constexpr`, if you also do so to `int_name` and `RawTypeName()`.)

The problem is that unless you go our of your way, the original long strings like `"const char *RawTypeName() [T = int]"` will still show up in the compiled binary. Which probably isn't that big of a deal, but it is *uncool*.

With some `constexpr` magic, it's possible to avoid this entirely:

```cpp
// This reuses `RawTypeName()`, `Replace()`, and `AdjustType()` from earlier.
// All of them need to be marked `constexpr`.

struct RawTypeNameFormat
{
    std::size_t prefix_len = 0;
    std::size_t suffix_len = 0;
};

constexpr RawTypeNameFormat raw_type_name_format = []{
    std::string_view int_name = RawTypeName<int>();
    RawTypeNameFormat ret;
    ret.prefix_len = int_name.rfind("int");
    ret.suffix_len = int_name.size() - 3 - ret.prefix_len;
    return ret;
}();

template <typename T>
constexpr std::string TypeNameLow()
{
    std::string_view name = RawTypeName<T>();
    name.remove_prefix(raw_type_name_format.prefix_len);
    name.remove_suffix(raw_type_name_format.suffix_len);
    return AdjustTypeConst(name);
}

template <typename T>
constexpr auto type_name_array = []{
    constexpr std::size_t n = TypeNameLow<T>().size();
    std::string raw = TypeNameLow<T>(); // The function must be called twice, for obscure C++ reasons, at least at the time of writing this.
    std::array<char, n + 1> ret{};
    for (std::size_t i = 0; i < n; i++)
        ret[i] = raw[i];
    return ret;
}();

template <typename T>
[[nodiscard]] constexpr std::string_view TypeName()
{
    const auto &ret = type_name_array<T>;
    return {ret.data(), ret.data() + ret.size() - 1};
}

// ---

class MyClass {};

int main()
{
    std::cout << TypeName<MyClass>() << '\n';
}
```

This prints `MyClass` on all three compilers. I'm sure there is some room to optimize the compilation speed here.


If you look at the assembly output on [godbolt][1], you'll see that indeed, the only string in the binary is `"MyClass"`, there's no `"const char *RawTypeName() [T = int]"`.

## The cppdecl library

As mentioned at the beginning of the post, I've made a library called [cppdecl](https://github.com/MeshInspector/cppdecl) to simplify obtaining type names.

It wraps either `typeid(T).name()` or the `.function_name()`-based approach, and then applies a large collection of rewrite rules to the result, to make the names as consistent as possible across compilers. Like in the example above, all those transformations are performed at compile-time.


&nbsp;

Thanks for reading!

[1]: https://gcc.godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGIAKykrgAyeAyYAHI%2BAEaYxCAA7AmkAA6oCoRODB7evgGp6ZkCoeFRLLHxSbaY9o4CQgRMxAQ5Pn6BdpgOWQ1NBCWRMXGJyQqNza15HeP9YYPlw0kAlLaoXsTI7BzmAMxhyN5YANQmO25NxEwAnqfYJhoAgrv7h5gnZ8hj6FhUt/dPZj2DAOXmOpzcTjGxEwrD%2Bj2ewNe7zcCjWG0wAH1aKhRHUGHCAUCQWCzlCwsAMQA3PCYADuBIRxLe4LJRgZ8P8Vk5DAweAUomI6BM/gAIsKRUc0AwxphVCliEcviAQKzgEcAEqYFIGTYQJUqgjEclUmm0xVozakRUEdDK1UmulHJhW/X26mO6JLf4mBJWR5HAPW20Go1GI7QginP1Pf2B114ABemIIkvWGLSCij/0DQbtieTRwMY3T6XeEo0WfhsYDtIQdDeUGQaYzZfN602ADoaAx0BBnYWmMWM0slkcwGBThLXYbycqGMPs4GfdGczmIx2mCkUq49RbMFaiwQSworU3iMejgBaAdD9JenYr1cBiMnSyTo7RSsPJ8Bw8X99nv%2BlhOh2GRJhA96PgGPpilW36Buum7bj2u7tvuN5HguD6Ls%2BmAEOsDDhnhX4wd6HJchYPL4PyTRCqK4qSgIMpygq06hmqDzoNoXhjAAKlc256jadozkYDpmgQAmYF68K%2Bjhr57FQPxHBiACyQhuBiABq2DqvJbHkkRKbvpq2pMLqknbla5hmAcg4KK%2BZjWWYZg2ZB8kviZWo6pgEARs5ZhQl4DiOQFblfmueGtqZPl%2BXhAVeAwWShY54XYdWRnRd55m%2Bf5jmMD4KU2WlUFGQRRkRUugI1Eo8kAPR1UcvF1g5CgIGstDoB%2BbzcWMJwAGwaAZRgQJZ0kmINVrRF4KZuAYYZ8h%2BXjAMAmC2kcCAEAQKQKCADXAIQCBeNEHZoCwdW0LQlLnZd12XvKqDaF0BB1XyCheJgCh1Vw/VcBoCRcB5eHlcNwCjVJHboEwjQQVaY2Q9DTAQa%2BFhHPDYG%2BSOlXQdVPZ4L8smwTGDwEJgLBmaTyJjcwbBNQSUrMfKjHSimyAIE0RwAFTqkwtL8duESsJj3pyRlEYg8JKp7liOLQ1kypntCggQV2iXdAIGI08L6VPAkRP/EFIU83zUmC2wABiJAsNDIulfGSYYim8qYDQqhYowrYVjrOb2wW71UK77uEe%2BXvRqROv/AzpMsRqvP85EQuW8Q1sppctKO1JmtCxi/DJ9DrbClyYqi/BAag%2BJRxhEeWvRXHptC%2BCVe3BB2OxybAuJ1b%2BcRq367O4H9DBzsEpV1nbAdsQ3a9jZTcue5Yt4aBXgB3gbuD62o9a6B%2BbI9eOxXkZHb96vQe98DxCET3Ed6y3EePKT5MGJT4LU0LdM7HcjxR7KTOg019dsGCKgWkLdZJ20lm6U0Rwa4mTrh3Ng4JeLN3nqXaBQsJ5k1QJSTEx9VB%2BV5hnbcY9MS5xTkfaEA9GAoJzFvaELAsGYn9q7fB6cxrEJzl3AgS8V5ryoWffCF8jicV6gQeOEAtYoNInBB%2BFNmRnFfrTJBH9I5MWjkzJgM1UBo0zlrDEFxrgF05OKZc8lv4x19o7aBrZ45m0wEA%2BkZwlHYBVhjW%2B4DgyqnDLzaxAC7HAMQcgqMRwGpNQQG8Kgas8RHBYDxFMsRJRiHoF1AgtI8CWiOLnI4qBoj8nWG8Nw1hrBEUHExK0%2Bd6CDhTPnAgYS0Z4FpqgKgRxaRGkcGGGpfIOz6Ulvom4Hx2ZkCsUUwGH8KpyT1q3TJQkPH5ksXgT2QT5ngmgYswplgZKoMipGTkeAGLvjToXXZ9FvYIXPpfYi18RRuLIiTMmsiqZSRrk470RjuS8hooKYxopmaM1YhA0SFJ3RmhsULUButSpRydJo18/VPLD20UQ3RvSAnKJObhARwdfTrihjDFYh8cVI1HEU9crjRzXkBhMy5NyQmXlpTcuyCgHKqSuHNeyJxxlijvg8Ku0SmBhDBSYjK%2Bo0AzWRMskFCCzjMtZYy5BYqzhjm2P4NwDAwDbEuRwFYtBOD%2BF4H4DgWhSCoE4AUt8wFURoQUjwUgBBNCapWAAawCBofQnBJB6rtUazgvBdouttQazVpA4CwCQGdFI9YyAUAgGGiNKB5rADMFwP6fA6Ck2ILtCA0RPXRDCE0K4nBrU5uYMQK4AB5aIT0HAFt4GdNgghS0MFoPmgNpAsDTWAOcS6u1uC8CwNbIw4gW34GhN0bB3bDWyi6DNLY1qq41E9bQPA0RLglo8FgT1M4WDVtINg4g0R0iYBFGTQwwBF1GDtSsKgBhgAKC0qaUtyFt38EECIMQ7ApAyEEIoFQ6gW26C4PoE9KA1k2EXTk%2BAKxUApDxN2y8XxJymDNRYMw/UrwAHVEnodlIaJgV5txQ0EGky8cRiAkAULwBhxAjRYF2pAFYnR1Z%2BAgK4SYfgAMhDmGUCoegMxFGyJ4NoPHCh4gGFx4YAGGN4l6BMATeQJM1Gej0GYomhjxAkzMVjegxh9BUwsNT9G9wSC1Tqj1LbjUcCOKoAAHP1S8/VJCSnjUcRNHYuAdg0EcCAuBCAkCtUsXg/qtAjlIGEpgWB4iwydf4F12qODutIPqw15mfUgD9Re4zHAzCmaS96m16Wd1xAyM4SQQA
