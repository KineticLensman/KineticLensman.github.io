---
layout: post
title: "Make a Lisp [7] Deep dive: looping"
date: 2020-04-18
---

Please see [this post](https://www.non-kinetic-effects.co.uk/blog/2020/04/14/AI-Projects-Eliza) for the background to this content.

# The need for a loop capability in `JKL`

The top level of Eliza in *Paradigms* uses the Common Lisp `loop` macro to repeatedly read lines of input from the keyboard before passing them to the Eliza rules processor:
```
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
   (print 'eliza>)
   (write (flatten (use-eliza-rules (read))) :pretty t)))
```
Unfortunately, I can't implement this directly in `JKL` because it lacks a `loop`-like iteration mechansism (like MAL, `JKL` relies on recursion). My options were therefore to
* Use recursion instead of iteration. This would be the simplest option, but I doscounted it because it wouldn't offer any opportunities to enhance `JKL` - one of my project goals. So I discounted it
* Implement something like Common Lisp's `loop` macro (which is also used in Clojure)
* Extend `JKL`'s recursion mechanism with a Clojure-like `recur` function

# Deciding how to implement `loop` in `JKL`

As evidenced by Clojure, a Lisp implementation can include both `recur` and `loop`, so there isn't an irrevocable choice between them. I therefore decided to explore both before picking one to let me proceed with the Eliza project.

## Looping in Common Lisp

I decided to start by finding out how is `loop` implemented in Common Lisp, which meant searching for an implementation where the source code is in the public domain. One such implementation is [Steel Bank Common Lisp](https://github.com/sbcl/sbcl), which has the following [`loop` macro](https://github.com/sbcl/sbcl/blob/master/src/code/loop.lisp)

```
(defun loop-standard-expansion (keywords-and-forms environment universe)
  (if (and keywords-and-forms (symbolp (car keywords-and-forms)))
      (loop-translate (make-loop keywords-and-forms environment universe))
      (let ((tag (gensym)))
        `(block nil (tagbody ,tag (progn ,@keywords-and-forms) (go ,tag))))))

(sb-xc:defmacro loop (&environment env &rest keywords-and-forms)
  (loop-standard-expansion keywords-and-forms
                           env
                           *loop-ansi-universe*))
```
The line
```
`(block nil (tagbody ,tag (progn ,@keywords-and-forms) (go ,tag)))
```
shows that the core of `loop` uses the `go` special form (the Lisp equivalent of the much maligned goto statement) within a `tagbody`. This isn't at first glance the best route for `JKL` - I'd need to implement some sort of goto-like mechanism first, which isn't something I'd considered as a priority. I'd also need to implement some of Common Lisp's loop syntax, which is actually an idiosyncratic domain-specific langauge, and not particularly lisp-like.

## Looping in Clojure

Clojure's looping mechanism is very different to Common Lisp's. Specifically, Clojure has two special forms: `loop` and `recur`, as described in the [Clojure reference manual](https://clojure.org/reference/special_forms). `loop` is like `let`, in that it has a set of bindings followed by a body of forms. It is usually used in conjunction with `recur`, which rebinds the `loop` variables before control jumps back to the beginning of `loop`. If `recur` is used outside a loop, control jumps back to the start of the function in which it occurs. `recur` must be used in the so-called tail position of a function or a loop (its use in other places is an error). 

By way of example, here is an example from the reference manual of `loop` and `recur`:
```
(def factorial
  (fn [n]
    (loop [cnt n acc 1]
       (if (zero? cnt)
            acc
          (recur (dec cnt) (* acc cnt))))))
```

Because `JKL` is conceptually closer to Clojure than Common Lisp, I decided to aim for a `JKL` equivalent of Clojure's `loop` - `recur` approach.

# Implementing `loop` / `recur`

## Some background

At this point, instead of starting to hack together a quick-and-dirty solution, I decided to spend some time thinking through the issues involved - since `loop` / `recur` seemed like it would be the most complex change I'd yet made to the baseline `JKL` as described by the MAL guidelines. I spread this thinking out over several days, and the rest of this section captures my conclusions. In retrospect, this thinking became a high-level design of the proposed solution, although I hadn't planned that when I started this part of the post (which up to now had been a sort of stream of consciousness).

First, some background. In `JKL`, the `EVAL` function is essentially a large C# switch statement whose cases correspond to the various special forms (`fn`, `do`, `def!`, `let*`, etc) in `JKL`. Each special form:
* Processes its arguments and executes the statements in its body, if any, and then either
* Directly returns a `JKL` value, OR
* Returns the value calculated by a recursive call to `EVAL`, OR
* Where Tail Call Optimisation is possible, loops back to the beginning of `EVAL` rather than making a recursive call

The evaluation environment (i.e. the set of symbols that are bound to data values, function arguments, etc) is held in `Env` object, which is basically a C# dictionary that maps symbols to values (without caring where they came from). New `Env`s are created:
* by the original REPL
* when functions are created (to hold the argument values)
* when `let` special forms are evaluated (to hold the `let` variables)
* TBD any others?????

Other symbols and their values are added to an `Env` by `def!`. Finally, each `Env` stores a pointer to the `Env` within which it was created (or `null` for the outermost REPL), thus providing a lexical scoping mechanism for symbols that have the same name. `EVAL` takes an `Env` as an argument, and can thus pass it's own environment (or a derived one) to recursive calls.

## `loop` and `recur` 

Conceptually, `recur` works by returning control to a previous point in the evaluation - the corresponding `loop` or containing function, passing new values for the `loop` variables or the function arguments. Once the new bindings have been established, the forms in the body of the `loop` or function are re-evaluated. Evaluation continues until the flow of control reaches the end of the `loop` (or function) without invoking `recur`, or until some resource runs out.

Unfortunately, there is no precedent for this process in `JKL` (or MAL): none of the existing special forms explicitly return control to a previous point. Furthermore, with the exception of the `Env` mechanism, there is no explicit tracking of the evaluation stack or the state of the computation as it progresses.

So some new mechanisms are going to be needed


TBD - I'm now exploring this idea. Watch this space


