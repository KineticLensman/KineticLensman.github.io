---
layout: post
title: "Make a Lisp [6] The journey continues..."
date: 2020-04-03
---

# `JKL` - the journey continues

As described [in a separate post](https://www.non-kinetic-effects.co.uk/blog/2020/04/14/AI-Projects-Eliza), I've decided to continue the programming project I started in 2019 by using my Lisp implementation - `JKL` - for some application development. Specifically I decided to re-implement one of the classic AI systems described in the book *Paradigms of Articial Intelligence Programming*: Eliza, the first chatbot.

Part of the challenge is that I can't directly code in `JKL` the Eliza algorithms provided in *Paradigms*. Specifically, *Paradigms* decribes Eliza using [Common Lisp](https://common-lisp.net/). However, although `JKL` is a Lisp-like language, it is actually based on the teaching language [MAL](https://github.com/kanaka/mal/blob/master/process/guide.md), which itself is derived from [Clojure](https://clojure.org/index) - a variant of Lisp developed after *Paradigms* was written. Because MAL is a minimal language intended to showcase Lisp implementation techniques, not to support serious application development, it lacks a 'standard library' of functions. Furthermore, data structures in `JKL`, like those in Clojure and MAL, are by default persistent and immutable, compared with those in Common Lisp whose variables can  be modified or redefined once created.

For these reasons, I have had to 
* Add functionality to `JKL` where it is obviously mising (e.g. for list and string processing). Incidentally, this gives me an excuse to continue to play around with the Visual Studio C# environment within which `JKL` was developed
* Develop `JKL` alternatives to the *Paradigms* approach that better retain the immutable, functional spirit of the original Clojure semantics

Where there was a choice between these two approaches, I opted for 'maintain-Clojure-consistency' as my overarching principle. 

This post - which will continue to evolve during this project - describes these changes. 

# Handling semantic variations

## Atoms

As already stated, data structures in `JKL` are immutable and persistent. The exceptions are referred to as atoms - values that are created using the `atom` function and updated using `swap`. In contrast, in Common Lisp, `atom` is a function that returns `T` for anything that isn't a cons (a list cell), and where `(atom)` returns `T`.

Because I'm prioritizing Clojure compliance, I had to write an `atomic?` function that has the same effect as Common Lisp's `atom`. The resultant `JKL` code is as follows:
```
(def! atomic? (fn* (x)
	(if (sequential? x)
		(if (empty? x)
			true
			false)
		true)))
```

# New `JKL` functionality

## Looping

I've moved this part to a [separate post]() because it became quite a complex story in its own right.



## The `and` macro

I added an `and` macro using `JKL`'s existing `or` macro as a model. The semantics of `and` in `JKL` match that of Clojure, namely that `(and)` returns true but otherwise evaluates all of its arguments until any one returns `false` or `nil`. The value of the last non-false argument is returned. The resultant code is
```
(defmacro! and (fn* (& and-args)
	(if (empty? and-args)
		true
		(if (= 1 (count and-args))
			(if (first and-args)
				(first and-args)
				false)
			(let* (and-var (gensym))
				`(let* (~and-var ~(first and-args))
					(if ~and-var
						(and ~@(rest and-args))
						~and-var)))))))
```
## Improved `map` implementation

As per the `MAL` guide, the `map` function in `JKL` takes a function `f` and a single sequence `s`, and then applies `f` to the successive elements of `s`. However, looking at some exercises in Lisp textbooks, I realised that Common Lisp's `mapcar` and Clojure's `map` function can handle multiple sequences. I therefore decided to extend `map` to have similar semantics.

Internally, the `JKL` implementation of `map` makes a new sequence to hold the result, and then iterates over the `s` applying `f` to each element in turn. One consequence, which I hadn't thought  through before, is that `f` must take a single argument. Consider the following extract from the REPL....

```
JKL> (def! f (fn* (a b) (+ a b)))
<fn (a b) (+ a b)>
JKL> (f 1 2)
3
JKL> (map f (1 2 3))
Eval error: More parameters (bindings) than supplied values (expressions): (a b) ... (1)
```
TODO - this fix is ongoing at the moment. Watch this space.

## Miscellaneous new builtin `JKL` functions added for Common Lisp compatability

I also added several Common Lisp / Clojure functions to `JKL` as built-in functions implemented using C# (the language in which `JKL` itself is implemented). The new functions are as follows:  

* `char` - extracts a single element of a string (since `nth` only works on sequences). It throws an error if the index is negative, but returns nil if the index exceeds the string length. `char` actually returns a substring of length 1 rather than a character because `JKL` currently lacks a character class. Such a class might get added in due course
* `symbol-name` - returns a string which is the print-name of a symbol, or throws an error if given a non-symbol
* `trace` - toggles a new tracing mechanism which is currently used to record `env` updates. Will add trace-levels or similar in due course

# Other updates and fixes

I made several other enhancements based on experience using `JKL`. Some of these were made before I started the current Eliza project, but I've only just got around to writing them up now. In approximately chronological order, I:

* Revised `let*` to allow multiple body forms, the last of which is the return val. Previously `let*` only had a single body form, and a `do` was required if multiple forms were wanted. However, the new implementation is inconsistent with the `(do...)` equivalent, so  I added a TODO to consider refactoring the two

* Refactored `try*`...`catch*`... so that syntax error checking of the `catch*` clause is now performed *before* the `try` is evaluated. The intent is to help avoid error-in-error-handler situations that could slip through user testing that doesn't explicitly check the thrown exception branch. The cost is to make correct code marginally less efficient

* Checked that `JKL` can handle exceptions inside a `catch*` clause (to one level deep)

* Noticed that `let*` bindings can include symbols such as `+` so that `(let* (+ 1) +)` returns `1` and the following returns `-1`. I added a TODO to consider getting rid of these
```
; use the let binding to redefine + as -
(let* (+ (fn* (a b) (- a b)))
    (+ 1 2))
```

* Changed the error handling that results when non-numbers are used as arguments for numeric functions (e.g. `(+ 1 "a")`). I'd previously treated these as internal errors without considering that these cases might be bugs in code written by `JKL` users. Now `JKL` prints a more informative evaluation error message rather than just terminating

* Started to rationalise the format of the error messages printed by built in functions

* Redid error messages in `Env` to give better info when binding fails because the number of bindings doesn't match the number of expressions. The need for this fix became apparent when working on the improved implementation of map as described above

* Changed the definitions of built-in macros (`or`, `not`, etc) so that their args have meaningful names (e.g. `or-args` rather than `xs` as per MAL). This helps debugging by making it easier to identify places where incorrect numbers of arguments are supplied to the macros


