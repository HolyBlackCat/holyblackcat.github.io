---
layout: post
title: Unforgettable factory revisited
tags: C++ templates
toc: true
---

## TL;DR:

This post presents some improvements to the original [Unforgettable Factory Registration](https://www.nirfriedman.com/2018/04/29/unforgettable-factory/) by Nir Friedman.

Reading that post is not a requirement (I explain everything from scratch), but if you're already familiar with it, you can skip to the [improvements](#instantiating-the-variable-by-passing-it-as-a-template-argument) part. (In short, I'm explaining how to remove the dependency on the default constructor instantiation.)

## The problem statement

It's often useful to know every class derived from a specific base, either to construct all of them, or to construct a specific derived class (typically by its name).

For example:
```cpp
struct Animal
{
    virtual ~Animal() = default;
    virtual void Say() = 0;
};

struct Cat : Animal
{
    void Say() override {std::cout << "Meow!\n";}
};

struct Dog : Animal
{
    void Say() override {std::cout << "Bark!\n";}
};

int main()
{
    std::string animal_type = "Cat"; // Imagine this comes from the user at runtime.
    std::unique_ptr<Animal> ptr = SomeMagic(animal_type); // We want this to do `std::make_unique<Cat>()`.
    ptr->Say(); // Meow!
}
```

This is typically achieved with a map of class names to lambdas that create objects of those types:
```cpp
int main()
{
    std::map<std::string, std::unique_ptr<Animal> (*)()> m = {
        {"Cat", []() -> std::unique_ptr<Animal> {return std::make_unique<Cat>();}},
        {"Dog", []() -> std::unique_ptr<Animal> {return std::make_unique<Dog>();}},
    };

    std::unique_ptr<Animal> ptr = m.at("Cat")(); // Calls `std::make_unique<Cat>()`.
    ptr->Say(); // Meow!
}
```

The question is how do we fill the map `m`. Doing it manually is an option, but this way it's easy to forget some types. Additionally, this can lead to poor incremental compilation times, because the source file creating the map needs to be rebuilt every time a class is added or removed, and it needs to `#include` headers for every derived class under the sun.

So it is desirable for each derived class to register itself independently. How do we do this without listing all the classes in one place?

## Initialializers of global variables

The solutions typically involve global variables (or `static` ones inside classes; the right term is "variables with static storage duration"), as the initializers for those run at program startup, before entering `main()`. (See [caveat](#behavior-in-static-libraries).)

For example:

```cpp
// animal.h
struct Animal
{
    virtual ~Animal() = default;
    virtual void Say() = 0;
};
using AnimalMap = std::map<std::string, std::unique_ptr<Animal> (*)()>;
AnimalMap &GetAnimalMap();

// animal.cpp
AnimalMap &GetAnimalMap()
{
    static AnimalMap ret;
    return ret;
}

// cat.h
struct Cat : Animal
{
    void Say() override {std::cout << "Meow!\n";}
};

// cat.cpp
static const int register_animal = []{
    GetAnimalMap().try_emplace(
        "Cat",
        []() -> std::unique_ptr<Animal> {return std::make_unique<Cat>();}
    );
    return 0;
}();

// dog.h
struct Dog : Animal
{
    void Say() override {std::cout << "Bark!\n";}
};

// dog.cpp
static const int register_animal = []{
    GetAnimalMap().try_emplace(
        "Dog",
        []() -> std::unique_ptr<Animal> {return std::make_unique<Dog>();}
    );
    return 0;
}();

// main.cpp
int main()
{
    std::unique_ptr<Animal> ptr = GetAnimalMap().at("Cat")();
    ptr->Say(); // Meow!
}
```
Several things to note:

* Notice how each class can use
  ```cpp
  static const int register_animal = []{
      GetAnimalMap().try_emplace(
          "X",
          []() -> std::unique_ptr<Animal> {return std::make_unique<X>();}
      );
      return 0;
  }();
  ```
  to register itself in the map. Since this is a global variable, it's constructed at program startup, before the `main()` function is entered, so that's when the lambda runs.

  This way we don't need a single big source file that knows all derived classes, and we can even load new classes at runtime from shared libraries.

* Secondly, notice how the map is now a `static` variable in a function:
  ```cpp
  using AnimalMap = std::map<std::string, std::unique_ptr<Animal> (*)()>;
  AnimalMap &GetAnimalMap()
  {
      static AnimalMap ret;
      return ret;
  }
  ```
  This is important, because if it was a global variable, you'd run into the risk of it being constructed *after* the `register_animal` variables, so they would be accessing the map before it's created.

  But if we move it inside of a function, it'll be constructed on the first use instead, no matter when it happens.

  This function *can* be `inline` in a header, but only as long as shared libraries aren't involved. If they are, then you run the risk of getting a separate map in every shared library, which seems to be impossible to avoid on Windows without moving the function to a `.cpp` file. (On Linux this can happen as well if `-fvisibility=hidden` is used, but can then be fixed by marking the function with `__attribute__((__visibility__("default")))` as usual.)

* The derived classes don't necessarily need headers anymore, and can be defined in `.cpp` files (as long as other translation units access them via the base class only).

* Or if you want headers, the `register_animal` variables *can* also be `inline` in headers, but then if a header isn't included in any TU (translation unit), the class won't get registered. This problem is shared by all other header-only registration methods. (You'd also need to come up with unique names for the variables, which is best solved by making them full specializations of a variable template. I'm not going to go into more detail here, since we have better approaches to class registration, explained below.)

## Wrapping the global variable in a macro

Most often the `static const int register_animal = ...;` gets wrapped in a macro for the ease of use:

```cpp
// animal.h
#define REGISTER_ANIMAL(T) \
    static const int CAT(register_animal_, __COUNTER__) = []{ \
        GetAnimalMap().try_emplace(#T, []() -> std::unique_ptr<Animal> {return std::make_unique<T>();}); \
        return 0; \
    }();

#define CAT(x, y) CAT_(x, y)
#define CAT_(x, y) x##y
```

Several things to note here:

* Firstly, notice how the class name is obtained via `#T`. This makes it sensitive to how the user spells the class name (`Cat`, `::Cat`, `SomeNamespace::Cat`, etc). The alternatives are discussed later.

* Secondly, notice that the variable name is now generated using `CAT(register_animal_, __COUNTER__)`, instead of being the fixed `int register_animal`.

  This gives you a unique variable name for each use of the macro, which lets you to use `REGISTER_ANIMAL` multiple times per `.cpp` file.

  `__COUNTER__` is a macro that expands to the next integer every time it's used: `0`, `1`, `2`, etc, so the names end up being `register_animal_0`, `register_animal_1`, etc. (The counter is per translation unit, i.e. per individual `.cpp` file, but this isn't a problem for us, since the variables are `static`.)

  This macro is non-standard, but all three big compilers (Clang, GCC, MSVC) support it nontheless. There are several alternatives (e.g. `__LINE__`; or the aforementioned variable templates, which make those usable in headers), but they aren't worth discussing, because we have better registration approaches than those macros.

## Registration via CRTP

*CRTP* stands for the *Curiously Recurring Template Pattern*. It refers to the technique of inheriting a class from a template that uses the same class as its template argument, i.e. `struct A : B<A> {...};`.

This happens to enable many useful template tricks, such as a more convenient registration syntax in our scenario.

We want to end up with this:
```cpp
struct Cat : MakeAnimal<Cat> // `MakeAnimal<...>` in turn inherits from `Animal`.
{
    void Say() override {std::cout << "Meow!\n";}
};
```
And have this register the class with no further intervention.

How do we implement `MakeAnimal`? If you think about it for a bit, you'll arrive at the correct conclusion that it has to do with `static` variables in classes:
```cpp
template <typename T>
struct MakeAnimal : Animal
{
    static int Register()
    {
        GetAnimalMap().try_emplace("??", []() -> std::unique_ptr<Animal> {return std::make_unique<T>();});
        return 0;
    }
    inline static int register_animal = Register();
};
```
This immediately presents a challenge: How do we get the name of `T` as a string? Since this is a template rather than a macro, we can no longer use `#`. This is a topic for a [whole separate post](/blog/2025/09/28/type-names.html). I'm going to assume that you've implemented one of the techniques from that post, and here I'm going to use `std::string(cppdecl::TypeName<T>())` from my [library](https://github.com/MeshInspector/cppdecl) mentioned there.

Also notice that we've replaced the lambda with a `static` function, because MSVC chokes on the lambda otherwise (which I assume is a compiler bug, but it didn't investigate this further).

Next, if you run this, you'll quickly notice that it doesn't work. The map stays empty. (Except on MSVC it apparently does work, but this is MSVC doing weird MSVC things as usual.)

This is because `int register_animal` doesn't get instantiated. In general, members of class templates are instantiated lazily, only if they are used. And indeed, if you add `(void)Cat::register_animal;` in `main()`, you'll notice that `Cat` does get registered, but `Dog` still doesn't.

## Forcing the variable to be instantiated

How can we instantiate `int register_animal` without using it outside of the template?

Naive attempts like
```cpp
inline static int register_animal = Register();

void blah()
{
    (void)register_animal;
}
```
...all fail, because here `blah` itself doesn't get instantiated.

There are several ways around this:

* Nir [in his article](https://www.nirfriedman.com/2018/04/29/unforgettable-factory/) suggests putting the instantiation in a constructor:
  ```cpp
  MakeAnimal()
  {
      (void)register_animal;
  }
  ```
  If you try it with my example above, this won't work as well. Something that Nir doesn't mention is that this relies on:

  * Either someone constructing `Cat` manually in at least one place (to instantiate the constructor; it's sufficient that this construction exists somewhere in the code, it doesn't need to actually happen at runtime).

  * Or the derived class having a user-provided constructor: `Cat() {}`, as this also instantiates the base constructor. Interestingly, turns out that `Cat() = default;` doesn't do that.

* You can put it in a virtual function:
  ```cpp
  // In `struct MakeAnimal`:
  virtual void unused()
  {
      (void)register_animal;
  }
  ```
  Since according to my tests, it turns out that virtual functions get instantiated unconditionally, even if unused. (The standard seems to be ambiguous on whether it's [unspecified](https://timsong-cpp.github.io/cppwp/n4950/temp.spec#temp.inst-11) or [required](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#8) for them to always be instantiated.)

  The problem with this approach is that we're wasting a slot in the vtable on an unused function (which probably doesn't matter in practice, but is uncool). You could try to put this into a virtual destructor to avoid wasting a slot, but turns out that `virtual` doesn't have its magic properties there for some reason, so this ends up having the same problems as instantiating it in the constructor.

And then there's the third solution:

### Instantiating the variable by passing it as a template argument

This is something I've been doing for a few years, but haven't seen anyone else do.

Turns out that you can force the variable to be instantied by passing it as a template argument. Like this:

```cpp
template <typename T>
struct MakeAnimal : Animal
{
    // I've replaced `int` with `nullptr_t` here because I like it more, but it doesn't affect the result in any way.
    static std::nullptr_t Register()
    {
        GetAnimalMap().try_emplace(std::string(cppdecl::TypeName<T>()), []() -> std::unique_ptr<Animal> {return std::make_unique<T>();});
        return nullptr;
    }
    inline static std::nullptr_t register_animal = Register();

    // Force `register_animal` to be instantiated:
    static constexpr std::integral_constant<const std::nullptr_t *, &register_animal>
        register_helper{};
};
```
Turns out that passing `&register_animal` as a template argument (to `integral_constant` in this case) is enough to instantiate it, even if it's otherwise unused.

This little trick is why I'm writing this blog post.

## Is this behavior guaranteed?

It seems to be. [`[temp.inst]/4`](https://timsong-cpp.github.io/cppwp/n4950/temp.spec#temp.inst-4) somewhat vaguely says that:

> ... a member of a templated class ... is implicitly instantiated when ... referenced in a context that requires the member definition to exist or if the existence of the definition of the member affects the semantics of the program; ...

And then passing `&register_animal` as a template argument ODR-uses this variable: [`[basic.def.odr]/5`](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#5)

> ... A variable ... that is named by a potentially-evaluated expression ... is odr-used by \[it] unless \[some conditions that don't apply here]

Template arguments appear to count as potentially-evaluated, since they aren't listed among the exceptions in [`[basic.def.odr]/3`](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#3).

And the variable being ODR-used requires it to be defined: [`[basic.def.odr]/12`](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#11)

> Every program shall contain at least one definition of every function or variable that is odr-used in that program ... \[plus some unrelated exceptions]

Which, according to `[temp.inst]/4`, means it will be instantiated.

## Possible issues with automatic registration

Here are some issues that this approach shares with other similar methods.

### The header must be included in at least one TU

Any header-only registration method can only work if the header is included in at least one translation unit.

This includes our CRTP trick if you use it in headers.

### Behavior in static libraries

This applies to any trick based on the constructors of global/static variables.

The standard contains this curious paragraph: [`[basic.start.dynamic]/5`](https://timsong-cpp.github.io/cppwp/n4950/basic.start#dynamic-5)

> It is implementation-defined whether the dynamic initialization of a non-block non-inline variable with static storage duration is sequenced before the first statement of main or is deferred. If it is deferred, it strongly happens before any non-initialization odr-use of any non-inline function or non-inline variable defined in the same translation unit as the variable to be initialized.

What this says is that the global/static variables in a TU can be constructed lazily at runtime, the first time anything from this TU gets used. So if nothing from it ever gets used, they may never be constructed.

But what happens in practice isn't as bad. In practice (on all implementations I've tested) this isn't an issue, as those variables are always constructed before `main()`, with one exception:

When static libraries are involved, any TU in a static library will get completely optimized out if unused, taking its global variables with it.

Linkers have flags that disable this behavior for individual static libraries:

* `-Wl,--whole-archive -lwhatever -Wl,--no-whole-archive` on GNU
* `/WHOLEARCHIVE:whatever.lib` on MSVC

&nbsp;

That's all I have, thanks for reading.
