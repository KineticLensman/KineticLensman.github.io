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
I can't implement this directly in `JKL` because, in common with MAL, `JKL` lacks `loop`, relying on recursion instead. My options were therefore to
* Directly use `JKL`'s existing recursion capability. This would be the simplest option, but wouldn't offer any opportunities to enhance `JKL` - one of my project goals. So I discounted it
* Extend `JKL`'s recursion mechanism with a Clojure-like `recur` function
* Implement something like Common Lisp's `loop` macro (which is also used in Clojure)

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
shows that the core of `loop` uses the `go` special form (the Lisp equivalent of the much maligned goto statement) within a `tagbody`. This isn't at first glance the best route for `JKL` - I'd need to implement some sort of goto-like mechanism first, which isn't something I'd considered as a priority. I'd also need to implement some of the mechanisms the Common Lisp provides for controlling loop-based iteration, and these are a complex subject in their own right.

## Looping in Clojure

Clojure's looping mechanism is very different to Common Lisp's. Specifically, Clojure has two special forms: `loop` and `recur`, as described in the [Clojure reference manual](https://clojure.org/reference/special_forms). `loop` is like `let`, except that it sets up a recursion point containing a body of forms. `recur` rebinds the `loop` variables and control jumps back to the beginning of the `loop`. If `recur` is used outside a loop, control jumps back to the start of the function in which it occurs. `recur` must be used in the so-called tail position of a function or a loop (its use in other places is an error). 

By way of example, here is an example from the reference manual of `loop` and `recur`:
```
(def factorial
  (fn [n]
    (loop [cnt n acc 1]
       (if (zero? cnt)
            acc
          (recur (dec cnt) (* acc cnt))))))
```

Given that `JKL` is very loosely based on Clojure rather than Common Lisp, my gut instinct is to write something analagous to Clojure's `loop` - `recur` mechanism. Furthermore, because `loop` is similar to `let`, I think I can get a lot of the necessary functionality by copying and modifying the existing `let` C# code. Which, incidentally, is as follows:
```
case "let*":
  if (ast.Count() < 3)
  {
    throw new JKLEvalError("'let*' should have two arguments (bindings-sequence and result), but instead had " + (ast.Count() - 1));
  }

  // Extract the first parameter - the bindings list. E.g. (p (+ 2 3) q (+ 2 p))
  a1 = ast[1];
  if (!(a1 is JKLSequence a1Seq))
  {
    throw new JKLEvalError("'let*' should be followed by a non-empty bindings-sequence and a result form, but got '" + a1 + "'");
  }

  // Create a new Env to hold the symbols defined by the let*. It's discarded at the end of the let.
  Env TempLetEnv = new Env(env);

  // Process each pair of entries in the bindings list.
  for (int i = 0; i < a1Seq.Count(); i += 2)
  {
    // The first element should be a 'key' symbol. E.g. 'p'.
    if (!(a1Seq[i] is JKLSym bindingKey))
    {
      throw new JKLEvalError("'let' expected symbol but got: '" + a1Seq[i].ToString(true) + "'");
    }
    // The second element (e.g. (+ 2 3)) is the value of the key symbol in Let's environment
    JKLVal val = EVAL(a1Seq[i + 1], TempLetEnv);
    // Store the new value in the environment.
    TempLetEnv.Set(bindingKey, val);
  }
  // The let body is evaluated in the temporary env we've just created.
  env = TempLetEnv;

  // EVAL all but the last result form.
  for( int iBody = 2; iBody < ast.Count()-1; iBody++)
  {
    EVAL(ast[iBody], env);
  }

  // Finally, using TCO, EVAL the last body form to get the Let's result.
  OrigAst = ast[ast.Count() - 1];
  break;

```

