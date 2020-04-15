---
layout: post
title: "Make a Lisp [6] The journey continues..."
date: 2020-04-03
---

# `JKL` - the journey continues

As described [in a separate post](https://www.non-kinetic-effects.co.uk/blog/2020/04/14/AI-Projects-Eliza), I've decided to continue the programming project I started in 2019 by using my Lisp implementation - `JKL` - for some application development. Specifically I decided to re-implement one of the classic AI systems described in the book *Paradigms of Articial Intelligence Programming*: Eliza, the first chatbot.

Although `JKL` is a Lisp-like language, it is not directly based on Common Lisp. Instead, it is based on the minimalist teaching language [MAL](https://github.com/kanaka/mal/blob/master/process/guide.md) which itself shares some features with the Clojure variant of Lisp. As a result, some Common Lisp features used in *Paradigms* are not available in `JKL`. In a few cases, Common Lisp and Clojure have different semantics, meaning that a direct implementation of the *Paradigms* code would result in unexpected (and probably wrong) behaviour.

This post - which will continue to evolve during this project - describes the 'under-the-bonnet' changes that I had to make to `JKL` to provide the missing functionality required by Eliza, or to handle semantic differences between Common Lisp and `JKL`.

# Handling semantic variations

TODO - describe `atomic?` as the `JKL` implementation of Common Lisp's `atom` function, because `atom` in `JKL` is actually a Clojure-like mutable value

# New `JKL` functions

## Miscellaneous new builtin `JKL` functions added for Common Lisp compatability

* `char` - extracts a single element of a string (since `nth` only works on sequences). It throws an error if the index is negative, but returns nil if the index exceeds the string length. `char` actually returns a substring of length 1 rather than a character because `JKL` currently lacks a character class. This may get added in due course
* `symbol-name` - returns a string which is the print-name of a symbol, or throws an error if given a non-symbol

## Added `and`

Added the `and` macro using the existing `or` macro as a model. The semantics of `and` in `JKL` match that of Clojure, namely that `(and)` returns true but otherwise evaluates all of its arguments until any one returns `false` or `nil`. The value of the last non-false argument is returned. The resultant code is
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
Notice that, as per Common Lisp, `(atom)` returns `true`.

## Improved `map` implementation

As per the `MAL` guide, the `map` function in `JKL` takes a function `f` and a single sequence `s`, and then applies `f` to the successive elements of `s`. However, looking at some exercises in Lisp textbooks, I realised that Common Lisp's `mapcar` and Clojure's `map` function can handle multiple sequences. I decided to have a go at extending `map` to have similar semantics.

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

# Other updates and fixes

Revised `let*` to allow multiple body forms, the last of which is the return val. Previously `let*` only had a single body form, and a `do` was required if multiple forms were wanted. However, the new implementation is inconsistent with the `(do...)` equivalent, so should I added a TODO to consider refactoring the two.

Refactored `try*`...`catch*`... so that syntax error checking of the `catch*` clause is now performed *before* the `try` is evaluated. The intent is to help avoid error-in-error-handler situations that could slip through user testing that doesn't explicitly check the thrown exception branch. The cost is to make correct code marginally less efficient.

Checked that `JKL` can handle exceptions inside a `catch*` clause (to one level deep).

Noticed that `let*` bindings can include symbols such as `+` so that `(let* (+ 1) +)` returns `1` and the following
```
; use the let binding to redefine + as -
(let* (+ (fn* (a b) (- a b)))
    (+ 1 2))
```
returns `-1`. Added a TODO to consider getting rid of these.

Changed the error handling that results when non-numbers are used as arguments for numeric functions (e.g. `(+ 1 "a")`). I'd previously treated this as an internal error without considering that hosted code might introduce errors. Now `JKL` prints a more informative evaluation error message rather than just terminating.

Started to rationalise the format of the error messages printed by built in functions.

Redid error messages in `Env` to give better info when binding fails because the number of bindings doesn't match the number of expressions. The need for this fix became apparent when working on the improved implementation of map as described below.

Changed the definitions of built-in macros (`or`, `not`, etc) so that their args have meaningful names (e.g. `or-args` rather than `xs` as per MAL). This helps debugging by making it easier to identify places where incorrect numbers of arguments are supplied to the macros.


