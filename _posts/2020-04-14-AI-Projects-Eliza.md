---
layout: post
title: "Make a Lisp [6] The journey continues..."
date: 2020-04-03
---



# !!!!! NOW UPLOADED - EDIT ON GITHUB!!!!!!!

# Back into lisp
So, after a long lapse, I decided to get back into Lisp development. My goals were to:

* Actually develop one or more actual applications
* Continue to develop  `JKL` itself by adding new functions needed by the applications, fixing bugs or addressing open TODOs

I'll describe the application development in a separate post. The remainder of this post will list the various changes that I've made to `JKL`.

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

# New features

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
(Incidentally, I redid the error messages in `Env` - where the binding occurs - to make the problem clearer)
TODO - this fix is ongoing at the moment. Watch this space.  


# !!!!! NOW UPLOADED - EDIT ON GITHUB!!!!!!!
