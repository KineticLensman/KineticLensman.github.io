---
layout: post
title: "Make a Lisp [7] Deep dive: looping"
date: 2020-04-18
---

# Looping deep dive

The top level of Eliza in *Paradigms* uses the Common Lisp `loop` macro to repeatedly read lines of input from the keyboard before passing them to the Eliza rules processor:
```
(defun eliza ()
  "Respond to user input using pattern matching rules."
  (loop
   (print 'eliza>)
   (write (flatten (use-eliza-rules (read))) :pretty t)))
```
However, in common with MAL, `JKL` lacks `loop`, relying on efficient recursion instead. My options were therefore to
* Directly use `JKL`'s existing recursion capability. This would be the simplest option, but wouldn't offer any opportunities to enhance `JKL` - one of my project goals. So I discounted it
* Extend `JKL`'s recursion mechanism with a Clojure-like `recur` function
* implement an interation mechanism in `JKL` based on the `loop` macro used in Common Lisp and Clojure

As evidenced by Clojure, a Lisp implemnentation can include both `recur` and `loop`, so there isn't an irrevocable choice between them. I therefore decided to explore both before picking one to let me proceed with the Eliza project.

How is `loop` implemented in Common Lisp? To find out, I looked at the [source code for `loop`](https://github.com/sbcl/sbcl/blob/master/src/code/loop.lisp) used in the open source [Steel Bank Common Lisp](https://github.com/sbcl/sbcl), where I found the following:

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
This shows that SBCL uses the Lisp equivalent of a `goto` statement. 

