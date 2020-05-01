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

# Implementing `loop` / `recur` in `JKL`

## Design planning

At this point, instead of starting to hack together a quick-and-dirty solution, I decided to spend some time thinking through the overall implementation approach - since `loop` / `recur` seemed like the most complex change I'd yet made to `JKL` 1.0. I wrote this section over several days - in effect roughing out a high-level design of the proposed solution, although I hadn't planned that when I started writing it. My first step was to fully understand the context for designing the new functionality, which meant re-acquainting myself with the basics of evaluation in `JKL` itself.

## The starting point - `EVAL` in `JKL` 1.0

The `EVAL` function is essentially a large C# switch statement whose cases correspond to the special forms in `JKL`, i.e. `fn*`, `do`, `def!`, `let*`, etc (the switch default is normal function application). When a special form is evaluated, it processes its arguments and / or executes the statements in its body, and then does one of the following:
* Directly returns a `JKL` value
* Returns the value calculated by a recursive call to `EVAL`
* Where Tail Call Optimisation is possible rather than recursion, loops back to the beginning of `EVAL`, helping to avoid stack overflow in the underlying C#

The evaluation environment (i.e. the set of symbols that are bound to data values, function arguments, etc) is held in `Env` objects, which are basically a C# dictionary that maps symbols to values (without itself recording where they came from). New `Env`s are created:
* By the original REPL, before any builtin functions and other startup values are loaded
* By `fn*`, to bind argument names and their values. These `env` objects are stored with the functions themselves, providing `JKL`'s closure mechanism
* By `let*`, to hold the `let` variables
* By `catch`, to store the values of `throw` statements
* By `EVAL`, when executing functions created by `fn*`

Each `Env` stores a pointer to the `Env` within which it was created (or `null` for the outermost REPL), thus providing a lexical scoping mechanism for symbols that have the same name. `EVAL` takes an `Env` as an argument, and can thus pass it's parent environment (or one it has itself derived, e.g. by `let*`) to recursive calls. Calls to `def!` add new symbols and their values to the current (innermost) `Env`.

## A conceptual model for `loop` and `recur` in `JKL`

Conceptually, `recur` returns control to a previous point in the evaluation - specifically the corresponding `loop` or containing function - passing back new values for the `loop` variables or the function arguments. Once the new bindings are established, `JKL` reevaluates the forms in the body of the `loop` or function. Evaluation continues until the flow of control reaches the end of the `loop` (or function) without invoking `recur`, as in the `factorial` example above, when the loop counter reaches 0. Furthermore, if the `recur` implementation can use TCO, then an infinite loop (e.g. `(loop (recur))`) shouldn't in itself cause stack overflow in the underlying C#.

Unfortunately, there is no precedent for this process in `JKL` (or MAL): none of the existing special forms explicitly return control to a previous point. Furthermore, with the exception of the `Env` mechanism, there is no explicit tracking of the evaluation stack or the state of the computation as it progresses. So some new mechanisms are going to be needed

## A solution outline

After a few days of thinking, I felt that I understood the overall solution. The key here was realising that the existing `env` mechanism could provide the basis for the evaluation history required. More precisely:

* Create a new `loop` special form, modelled closely on `let`
* Annotate `env` objects to record their purpose, namely (as per the cases above) REPL, `fn*`, `let*`, `catch`, `EVAL` and now `loop`
* allow `env` creators to store in their new `env` the names of the binding symbols and (unevaluated) body forms - these will be re-used by `recur`
* Create a `recur` special form that takes arguments as per its Clojure equivalent. When evaluated, `recur` asks the current `env` to find the closest enclosing `loop` or `recur` env (if there is none `recur` throws an error). `recur` retrieves the bindings and body forms stored by the enclosing `env`, throwing an error if the number of bindings doesn't match it's own arg count. `recur` then retrieves the enclosing `env`'s parent `env`, rebinds the arguments, sets the `EVAL` AST to the stored body, and then continues execution by TCO

The last step 'goes one `env` up' from the enclosing `env` (by using *its* parent) so that it doesn't cause an error by rebinding symbols that already have a value (this is a basic error in MAL).

I think this is the answer. Now to try it out!

Incidentally, these changes also provide the basis for a useful tool - a stack trace of the 'env's that can be accessed when a thrown error isn't caught, to provide better context information for debugging. I might even implement this first, since I suspect it will come in useful during the development I'm about to do.


