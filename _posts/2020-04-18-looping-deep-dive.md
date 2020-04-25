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
Unfortunately, I can't implement this directly in `JKL` because, in common with MAL, `JKL` lacks `loop`. relying on recursion instead. My options were therefore to
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

# Loop implementation options

Having made this decision, the next challenge was figuring out how to implement it. `loop` and `recur` are special forms, analogous to other `JKL` constructs such as `def!`, `let*`, `do`, etc, all of which sit within an `EVAL` function coded in C#. Logically, `loop` and `recur` would be added there as well. However, one big difference is none of the existing special forms ever explicitly return to a previous execution state in the way that `recur` needs to do. Instead, they typically process their arguments and return or recursively invoke `EVAL` to continue execution.

I spent some time wondering if I needed to add additional information to EVAL or somehow loop within it so that `recur` can send control back to `loop`. I then realised that there is perhaps an existing mechanism that I can repurpose. Specifically, `EVAL` builds `env` objects when it assigns expressions to bindings. At present, the bindings list isn't itself persisted. But, I wondered, if the `env` objects kept a note of the bindings, `recur` could perhaps access and re-use these. 

TBD - I'm now exploring this idea. Watch this space


