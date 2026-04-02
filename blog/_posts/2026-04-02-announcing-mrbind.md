---
layout: post
title: Announcing mrbind, an automatic binding generator for C++ to C, C#, and Python
tags: C++ bindings mrbind
toc: false
---

## Intro

After about 1-1.5 years of solo full-time development, I'm proud to announce [**mrbind**](https://github.com/MeshInspector/mrbind).

Mrbind parses C++ headers using [Clang libraries](https://clang.llvm.org/docs/LibTooling.html), and automatically generates bindings for them for other languages, currently **C**, **C#**, and **Python**.

It's currently used in production for [MeshLib](https://github.com/MeshInspector/MeshLib), which is a relatively large C++ library with 28k+ lines of headers that get parsed.

## Features

The main features are:

* Parsing C++ headers directly. There are no interface files (compared to e.g. SWIG).

  Any annotations are done directly in the input headers (such as excluding individual functions from the bindings, or listing arguments for templates).

* Proper support for standard library classes.

  We have custom bindings for standard containers, for `std::function` (accepting lambdas from other languages), and so on.

* Proper support for templates.

  We try to agressively instantiate all encountered templates (with all template arguments they're seen being used with), including recursively instantiating members of instantiated class templates.

  Anything that's not instantiated automatically can be [instantiated manually](https://github.com/MeshInspector/mrbind/blob/master/docs/adding_template_specializations.md).

## The future of the project

At least one more target language is planned: JS via Embind.

The priorities at work have shifted, so I'll have to work on JS support part-time, but it should get done eventually.

Then a refactoring of the Python backend is in order, but I'm not sure when I'll have time for that.

## Why not existing tools?

### SWIG

I considered using [SWIG](https://www.swig.org/), but decided to make a new tool, because:

* We really wanted a C backend, and SWIG doesn't have one.

* We didn't want to manually write interface files, which SWIG requires.

  Now, it's possible that we could *generate* the interface files from the data parsed by libclang, but I haven't investigated that.

### CppSharp

I briefly tested [CppSharp](https://github.com/mono/cppsharp), but had some issues with it:

* Certain C++ constructs caused it to crash or misbehave ([1](https://github.com/mono/CppSharp/issues/1934), [2](https://github.com/mono/CppSharp/issues/1935)), and the bug report didn't get much interest. I'd either have to actively fix bugs in CppSharp to get it to work on my code, or simplity the code, neither of which I wanted to do.

* I didn't like the interface that it produced. For example, raw pointers got exposed directly as `unsafe` C# pointers, as opposed to exposing them as optional references (emulated, since C# doesn't have optional references) (we assume raw pointers to semantically mean optional references).

* It's written in C# as opposed to C++, which makes patching it more difficult as a C++ developer.

And since it only supports one output language, anything that supports more than one language automatically gets an edge over it, by producing consistent output across languages.
