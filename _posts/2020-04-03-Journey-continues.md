---
layout: post
title: "Make a Lisp [6] The journey continues..."
date: 2020-04-03
---

# Back into lisp
So, after a long lapse, I decided to get back into Lisp development, picking up `JKL` where I left it last year. My goals were to:

* Use `JKL` to build a more significant application than the little test apps I've implemented to date 
* Continue to develop `JKL` itself by adding new functions needed by the applications, fixing bugs and addressing open TODOs

I'll describe the application development in a separate post. The remainder of this post will list the various changes that I've made to `JKL`. I started these a couple of months ago, but its only now that I've got round to writing things up.

# Updates and fixes

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

# New features

## String handling

Added a `char` function to extract a single element of a string (since `nth` only works on sequences). `char` actually returns a substring of length 1 rather than a characters because `JKL` currently lacks a character class. This may get added in due course.

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

## Added `symbol-name`

Added a `symbol-name` built-in function with the same semantics as CL - it returns the print-name of a symbol, or throws an error if given a non-symbol.

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
