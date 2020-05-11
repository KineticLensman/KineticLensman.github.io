---
layout: post
title: "Make a Lisp [7] Deep dive: looping"
date: 2020-04-18
---

# The need for a loop capability in `JKL`

Please see [this post](https://www.non-kinetic-effects.co.uk/blog/2020/04/14/AI-Projects-Eliza) for the background to this content.

The top level of Eliza in *Paradigms* uses the Common Lisp `loop` macro to repeatedly read lines of input from the keyboard before passing them to the Eliza rules processor:
```
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
   (print 'eliza>)
   (write (flatten (use-eliza-rules (read))) :pretty t)))
```
Unfortunately, , `JKL` lacks a loop mechanism (as does MAL, on which it is based), so I can't implement this Eliza code directly. The rest of this post described how I overcame this limitation.

# Choosing a looping approach

## Looping options

At first sight, I seemed to have the following options:
* Use recursion instead of iteration. This would be the simplest option, but it wouldn't offer any opportunities to enhance `JKL` - one of my project goals. So I discounted it
* Implement something like Common Lisp's `loop` macro (which is also used in Clojure)
* Extend `JKL`'s recursion mechanism with a Clojure-like `recur` function

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
There is a lot going on here, but the key line is
```
`(block nil (tagbody ,tag (progn ,@keywords-and-forms) (go ,tag)))
```
which shows that the core of `loop` uses the `go` special form (the Lisp equivalent of the much maligned goto statement) within a `tagbody`. This isn't at first glance the best route for `JKL` - I'd need to implement some sort of goto-like mechanism first, which isn't something I'd considered as a priority. I'd also need to implement some of Common Lisp's loop syntax, which is actually an idiosyncratic domain-specific langauge, and not particularly lisp-like.

## Looping in Clojure

Clojure's looping mechanism is very different to Common Lisp's. Specifically, Clojure has two special forms: `loop` and `recur`, as described in the [Clojure reference manual](https://clojure.org/reference/special_forms). `loop` is like `let`, in that it has a set of bindings followed by a body of forms. It is usually used in conjunction with `recur`, which rebinds the `loop` variables before control jumps back to the beginning of `loop`. If `recur` is used outside a loop, control jumps back to the start of the function in which it occurs. `recur` must be used in the so-called tail position of a function or a loop (its use in other places is an error). 

By way of example, here is an example from the Clojure reference manual of `loop` and `recur`:
```
(def factorial
  (fn [n]
    (loop [cnt n acc 1]
       (if (zero? cnt)
            acc
          (recur (dec cnt) (* acc cnt))))))
```

Because `JKL` is conceptually closer to Clojure than Common Lisp, I decided to aim for a `JKL` equivalent of Clojure's `loop` - `recur` approach.

# Implementing `loop` / `recur` in `JKL`

## Design planning

At this point, instead of starting to hack together a quick-and-dirty solution, I decided to spend some time thinking through the overall implementation approach - since `loop` / `recur` seemed like the most complex change I'd yet made to `JKL` 1.0. (Incidentally, I made an unconscious decision to implemennt `loop` in the underlying C# rather than to create a `loop` macro in `JKL` itself. I only realised a macro would have been an alternative when I came across [this blog](https://8thlight.com/blog/patrick-gombert/2015/03/23/tail-recursion-in-clojure.html)).

I wrote this section over several days - in effect roughing out a high-level design of the proposed solution, although I hadn't planned that when I started writing it. My first step was to fully understand the context for designing the new functionality, which meant re-acquainting myself with the basics of evaluation in `JKL` itself.

## The starting point - `EVAL` in `JKL` 1.0

The `EVAL` function takes two arguments:
* A `JKL` expression represented as an Abstract Syntax Tree (AST)
* An environment `env` object that holds the context (symbols and their values) within which the AST should be evaluated. Environments are described [here] (https://www.non-kinetic-effects.co.uk/blog/2020/05/03/environments)

`EVAL` is essentially a large C# switch statement whose cases correspond to `JKL` special forms , e.g. `fn*`, `do`, `def!`, `let*`, etc (the switch default is normal function application). When the AST is a special form, `EVAL` evalates the statements in the AST body and finshes by either:
* Directly returning a `JKL` value
* Returning the value calculated by a recursive call to `EVAL`
* Where Tail Call Optimisation is possible rather than recursion, looping back to the beginning of `EVAL`, helping to avoid stack overflow in the underlying C#


## A conceptual model for `loop` and `recur` in `JKL`

Conceptually, `recur` returns control to a previous point in the evaluation - specifically the corresponding `loop` or containing function - passing back new values for the `loop` variables or the function arguments. Once the new bindings are established, `JKL` reevaluates the forms in the body of the `loop` or function. Evaluation continues until the flow of control reaches the end of the `loop` (or function) without invoking `recur`, as in the `factorial` example above, when the loop counter reaches 0. Furthermore, if the `recur` implementation can use TCO, then an infinite loop (e.g. `(loop () (recur))`) shouldn't in itself cause stack overflow in the underlying C#.

Unfortunately, there is no precedent for this process in `JKL` (or MAL): none of the existing special forms explicitly return control to a previous point. Furthermore, with the exception of the `Env` mechanism, there is no explicit tracking of the evaluation stack or the state of the computation as it progresses. So some new mechanisms are going to be needed.

## A solution outline

After a few days of thinking, I'd hashed out a possible approach. The key idea was that the `JKL` `env` mechanism (as extended in a [separate deep dive](https://www.non-kinetic-effects.co.uk/blog/2020/05/03/environments)) could provide the basis for the evaluation history required. More precisely, I'd need to:

* Create a new `loop` special form, modelled on `let`
* Allow `env` creators to store in their new `env` the names of the binding symbols and (unevaluated) body forms - to be re-used by `recur`
* Create a `recur` special form that takes arguments as per its Clojure equivalent. When evaluated, `recur` asks the current `env` to find the closest enclosing `loop` or `recur` env (if there is none `recur` throws an error). `recur` retrieves the bindings and body forms stored by the enclosing `env`, throwing an error if the number of bindings doesn't match it's own arg count. `recur` then rebinds the arguments, sets the `EVAL` AST to the stored body, and then continues execution by TCO

I think this is the answer. Now to try it out!

## Implementation stage 1: `recur` within `loop`

I implemented `loop` and `recur` incrementally, following the steps I've just listed. The implementation went hand in hand with supporting changes to the `env` mechanism - namely a rebind mechanism (called from `recur`) and storage of bindings and body forms for re-evaluation. I also refactored `EVAL` rather than duplicate the complex code in `let*` when writing `loop`, creating a helper function in the process. 

As per my usual approach, I concentrated on writing comprehensive error checking throughout rather than proceeding straight to a quick-and-dirty solution. For example, `JKL` tests that the number of values defined by `loop` is the same as the number of expressions defined by `recur`. Although this error testing slowed down progress somewhat, the first proper functional code worked first time. Here it is - a fibonacci function based on `loop` and `recur`:
```
(def! fib
   (fn* (n)
      (loop [count n accumulator 1]
         (if (= 0 count)
             accumulator
             (recur (- count 1) (* accumlator count))))))
```
And here is some test output:
```
JKL> (fib 1)
1
JKL> (fib 2)
2
JKL> (fib 3)
6
JKL> (fib 4)
24
JKL> (fib 10)
3628800
JKL> (fib 20)
2.43290200817664E+18
```
Pleasingly, this runs in constant memory (according to Visual Studio graphical profiler) and is 'instantaeous' to the naked eye.

Unfortunately, however, the `(loop () (recur))` infinite loop mentioned above triggered a stack overflow exception in the underlying C#, which meant that the `recur` mechanism wasn't correctly using TCO as I'd hoped it might. (Incidentally, if `fib` is called with a negative number (e.g. `(fib -1)`) then a similar stack overflow occurs. This specific error can be headed off by a simple check in `fib` itself, but the underlying problem remains. I added a TODO to think about stack overflows).

Fixing the `(loop () (recur))` stack overflow turned out to be very easy. I'd incorrectly passed the stashed body forms to the TCO loop (altogether as a single list, rather than individually like let). With this fixed, the loop hung (implying  it was executing rather than crashing) using constant memory (implying that TCO is operating correctly).

## Implementation stage 2: `recur` within `fn*`

Now that `recur` works inside a `loop` special form, I had to make `recur` work inside a `fn*`, which (like `loop`) means stashing the function's parameters and body forms in an appropriate `env` for use by `recur`. I'd initially assumed this would look very similar to the `loop` implementation but had forgotten that function environments are more complicated - specifically there is the environment that exists when the function is created, and the environment that exists when the function is called. After some head scratching and experimentation, I realised that the parameters and body should not be stashed in the *create* environment but in the *call* environment. Once I'd figured this out, the following (clumsy-looking) non-`loop` fibonacci function worked as expected:
```
(def! fibrecur
  (fn* (target acc)
       (if (= 0 target)
            acc
          (recur (- target 1) (* acc target)))))
```
The far simpler infinite loop `(def! g (fn* (n) (recur n)))` also worked, in that `JKL` hung without crashing due to a stack overflow. 


## Tail check

In Clojure, `recur` can only be used in the so-called tail position - i.e. the last form executed by the enclosing `fn*` or `loop`. I haven't coded a tail-check test, but when I tried `(loop () (recur) 'dummy)` `JKL` generated an `EVAL` exception. I'm not too concerned because `recur` is only supposed to work in the tail position, but (as I write this) I don't actually understand why it is in fact crashing.

TO BE CONTINUED
 



