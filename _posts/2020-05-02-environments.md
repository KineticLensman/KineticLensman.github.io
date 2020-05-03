---
layout: post
title: "Make a Lisp [8] Deep dive: environments and stack tracing"
date: 2020-05-03
---

# Environments and stack tracing

*TBD - write intro para*

Stack tracing is one of the recommended *Next Steps* in the [MAL guide](https://github.com/kanaka/mal/blob/master/process/guide.md), more precisely, *Errors with line numbers and/or stack traces*. I added it to support my implementation of `loop` (see the separate [deep dive](https://www.non-kinetic-effects.co.uk/blog/2020/04/18/looping-deep-dive)) after realising that the tracing and looping mechanisms shared some common requirements and mechanisms. 

The intent is that when an exception is thrown, `JKL` should print something like a stack trace of the preceeding function calls, to help debugging. In `JKL`, the closest thing to a stack trace of function calls is the set of nested `env` objects that hold the environmental context (symbols and their values) for evaluation. My initial thinking was that when an exception is raised, the current `env` should be passed to the exception handler, its relevant details should be written to the console, and the process repeated for the parent `envs`, stopping after the outermost REPL environment (whose parent `env` is `null`).

This post describes the changes required to `JKL` 1.0 to enable stack tracing, starting with a quick recap of the environment objects which are key to the implementation.

# Background

## Environments in `JKL` 1.0

The `EVAL` method takes an `env` object as an argument - these contain a C# dictionary that maps symbols to values. Symbols and their values are added to an `env` by the `def!` special form. When a symbol is encountered during evaluation, `JKL` searches for the symbol's value using the `Find` method of the local `env`.  

The first `env` is created by the top-level REPL, before any builtin functions and other startup values are loaded. During evaluation, additional `env` objects are created:
* By `fn*`, to bind argument names and their values. These `env` objects are stored with the functions themselves, providing `JKL`'s closure mechanism
* By `let*`, to hold the `let` variables
* By `catch`, to store the values of `throw` statements
* By `EVAL`, when executing functions created by `fn*`

Each `env` stores a pointer to the parent `env` within which it was created (or `null` for the outermost REPL), thus providing a lexical scoping mechanism for symbols that have the same name. If `Find` doesn't find a symbol locally, it recursively searches the parent `env`, returning the next innermost value of the symbol, or raising an exception if it reaches the root `env` without encountering the symbol.

# stack tracing using `env`

## Changes required

'env' objects clearly hold some of the basic information required for stack tracing. However, there are some limitations:
* do not record additional contextual information, such as the reason they were created (e.g. because of a `let*`) or the origin of their symbols (such as the line number of the file, if any, that introduced the symbol)
* `env` objects are not accessible in many of the places where exceptions are raised, notably the built-in functions and the methods associated with `JKL` types (such as arithmetic operations or sequence access)

The rest of this section describes how I addressed these limitations.

## Annotating `env` objects

To add annotations to `env` objects I started by creating a C# enumeration type to record the various reasons for which they could be created, and then ensured this was set whenever creation actually occured.

## Making `env` objects accessible

This fix was more challenging. There seemed to be two broad approaches. The first is to try to pass the current `env` object (e.g. as an argument) to every place where errors could be thrown. This would require a massive number of changes (literally 100s) across the entire C# code base. 

The second approach is to establish a single record of the current `env`, and to update this when a new evaluation context is established.

