---
layout: post
title: "Make a Lisp [8] Deep dive: environments and stack tracing"
date: 2020-05-03
---

# Stack tracing

*TBD - write intro para*

Stack tracing is one of the recommended *Next Steps* in the [MAL guide](https://github.com/kanaka/mal/blob/master/process/guide.md), more precisely, *Errors with line numbers and/or stack traces*. I added it to support my implementation of `loop` (see the separate [deep dive]()) after realising that the tracing and looping mechanisms shared some common requirements and mechanisms. 

The intent is that when an exception is thrown, `JKL` should print something like a stack trace of the preceeding function calls, to help debugging. In `JKL`, the closest thing to a stack trace of function calls is the set of nested `env` objects that hold the environmental context (symbols and their values) for evalution. My initial thinking was that when an exception is raised, the current `env` should be passed to the exception handler, its relevant details should be written to the console, and the process repeated for the parent `envs` until the outermost REPL environment is reached.

This post describes the changes required to `JKL` 1.0 to enable this functionality, starting with a quick recap of the environment objects and their purpose.

## Environments in `JKL` 1.0

The `env` objects contain a C# dictionary that maps symbols to values. A new `env` is created:
* By the original REPL, before any builtin functions and other startup values are loaded
* By `fn*`, to bind argument names and their values. These `env` objects are stored with the functions themselves, providing `JKL`'s closure mechanism
* By `let*`, to hold the `let` variables
* By `catch`, to store the values of `throw` statements
* By `EVAL`, when executing functions created by `fn*`

Each `env` stores a pointer to the `env` within which it was created (or `null` for the outermost REPL), thus providing a lexical scoping mechanism for symbols that have the same name. As stated, `EVAL` takes an `env` as an argument, and can thus pass it's parent environment (or one it has itself derived, e.g. by `let*`) to recursive calls. 

'env' objects clearly hold some of the basic information required for stack tracing. However, there are some limitations, namely that `env` objects:
* Do not record additional contextual information, such as the reason they were created or the origin of their symbols, which are added by the `env` creator or by subsequent calls to `def!`
* Are not passed by `EVAL` to `JKL` built-in functions, so cannot be directle accessed when exceptions are raised by these functions
