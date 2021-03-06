---
layout: post
title: "Make a Lisp [8] Deep dive: environments and stack tracing"
date: 2020-05-03
---

# Environments and stack tracing

*TBD - write intro para*

Stack tracing - *errors with line numbers and/or stack traces* - is one of the recommended *Next Steps* in the [MAL guide](https://github.com/kanaka/mal/blob/master/process/guide.md). I decided I needed stack tracing to support my implementation of `loop` (see the separate [deep dive](https://www.non-kinetic-effects.co.uk/blog/2020/04/18/looping-deep-dive)) after realising that the tracing and looping mechanisms have some common requirements and mechanisms. 

The intent is that when an exception is thrown, `JKL` should print something like a stack trace of the preceeding function calls, to help debugging. In `JKL`, the closest thing to a function stack is the set of nested `env` objects that hold the environmental context (symbols and their values) for evaluation. My initial thinking was that when an exception is raised, the current `env` should be passed to the exception handler so that the context for the exception can be written to the error log (currently the console).

This post describes the changes required to `JKL` 1.0 to enable stack tracing, starting with a quick recap of the environment objects that support my planned approach.

# Background

## Environments in `JKL` 1.0

The `EVAL` method takes an `env` object as an argument - each of which contains a C# dictionary that maps symbols to values. Symbols and their values are added to an `env` by the `def!` special form. When a symbol is encountered during evaluation, `JKL` searches for the symbol's value using the `Find` method of the local `env`.  

The first `env` is created by the top-level REPL, before any builtin functions and other startup values are loaded. During evaluation, additional `env` objects are created:
* By `fn*`, to bind argument names and their values. These `env` objects are stored with the functions themselves, providing `JKL`'s closure mechanism
* By `let*`, to hold the `let` variables
* By `catch`, to store the values of `throw` statements
* By `EVAL`, when executing functions created by `fn*`

Each `env` stores a pointer to the parent `env` within which it was created (or `null` for the outermost REPL), thus providing a nested scoping mechanism for symbols that have the same name. If `Find` spots the symbol in the local `env`, it returns it. Otherwise it recursively examines the parent `env`.

# Stack tracing using `env`

## Changes required

`env` objects clearly hold some of the basic information required for stack tracing. However, there are some limitations:
* `env` objects do not record the necessary contextual information, such as the reason they were created (e.g. because of a `let*`) or the origin of their symbols (such as the line number of the file, if any, that introduced the symbol)
* `env objects` are passed to successive `EVAL` calls and are thus directly accessible within `EVAL` itself. However, the `env` is not passed to built-in functions or the methods associated with `JKL` types (such as arithmetic operations or sequence access), and thus cannot in turn be passed to exception handlers when errors are detected in these places

The rest of this section describes how I addressed these limitations.

## Annotating `env` objects

To add annotations to `env` objects I derived subclasses of `env` class to represent the different reasons for which an environment is created (REPL, `fn*`, `let*`, etc) and gave each subclass a different label for printing (initially I used an enum for the same purpose before realising that subclasses would be more convenient). I then allowed the subclass for functions (`EnvFnCall`) to store the function associated with the `env` so that this could be printed out in the stack trace.

## Making `env` objects accessible

Making the current `env` accessible to exception handlers was more challenging. There seemed to be two broad approaches. The first is to try to pass the current `env` object (e.g. as an argument) to every place where errors could be thrown. This would require a massive number of changes (literally 100s) across the entire C# code base. 

The second approach is to establish a single record of the current `env`, and to update this when a new evaluation context is established, and again at the beginning of every `EVAL`. This should at least track exceptions raised during reading and evaluation, which are the vast majority of cases.

## First try

Here's the result of using the new stack trace mechanism. The error occurs in a function `f` that takes a single argument, binds the arg to a local `let*` variable, and then tries to numerically add the let variable value to a string:

```
*** Welcome to JK's Lisp ***
JKL> (def! f (fn* (fnArg) (let* (n fnArg) (+ n "wtf"))))
<fn (fnArg) (let* (n fnArg) (+ n "wtf"))>
JKL> (f 1)
Eval error: Plus - expected a number but got: '"wtf"'
In let*
In <fn (fnArg) (let* (n fnArg) (+ n "wtf"))>
In REPL
```

This is looking good! But what happens if the error occurs in a nested function call?

```
...
JKL> (def! g (fn* (n) (f n)))
<fn (n) (f n)>
JKL> (g 1)
Eval error: Plus - expected a number but got: '"wtf"'
In let*
In <fn (fnArg) (let* (n fnArg) (+ n "wtf"))>
In REPL
```

`JKL` is correctly printing the immediate context for the error (the `let*` in `f`) but the fact that `f` was called by `g` isn't shown. Here's another more complex example, involving `f` as before, but now called from within a closure (a dynamically defined function) itself declared in a `let` in a separate REPL-level function `g`:
```
(def! f (fn* (fArg)
           (let* (n fArg)
		(+ n "wtf"))))
(def! g (fn* (gArg)
	   (let* (closureFn (fn* (clArg)
                     (prn "In closure") (f clArg)))
		(closureFn gArg))))
```
When `g` is invoked, we get...
```
...
JKL> (g 42)
"In closure"
Eval error: Plus - expected a number but got: '"wtf"'
In let*
In <fn (fArg) (let* (n fArg) (+ n "wtf"))>
In REPL
```
We aren't seeing a full call stack because the `env` objects (as currently implemented) were not in fact designed to record a sequence of function calls. Instead, their purpose is to record the context (hence 'environment') in which a function was defined, and to recover this context when the body of the function is evaluated. Specifically, when `EVAL` has to evaluate a non-core function (one created by `fn*` as opposed to say an arithmetic operation) it:
* Creates a new evalution `env` that consists of the one that was in force when the function was defined (which is the REPL for both `f` AND `g`) and which is stored with the function
* Adds to this `env` new symbols created by binding the function's arguments to the values being passed to the function
* Executes the body of the function in the new `env` (via TCO)

So, at this point, I don't have a full stack trace mechanism. However, the current functionality should be sufficient to support the required by `loop` and `recur` (the successful implementation of which proved this point. 

Additional changes required to support `loop` and `recur` were as follows:
* Storage of binding symbol names so that these could be rebound
* Storage of body forms so that these can be re-evaluated once a `recur` has set bindings up
* A `rebind` function that allows an existing symbol in an `env` to be rebound
