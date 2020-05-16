---
layout: post
title: "AI retrocomputing projects: Eliza [1]"
date: 2020-04-14
---

# Background

At the beginning of 2019 [I decided to get back into coding](https://www.non-kinetic-effects.co.uk/blog/2019/01/01/MAL-1) by building an interpreter for the Lisp programming language, following the excellent [MAL guidelines](https://github.com/kanaka/mal). This successful [exercise](https://www.non-kinetic-effects.co.uk/blog/2019/04/28/MAL-5) yielded `JKL` 1.0, my own version of Lisp. A year later, I decided to give `JKL` 1.0 a serious workout by re-implementing one of the classic Artificial Intelligence (AI) systems pioneered in the 1960s and 1970s, as described by Peter Norvig in his excellent book Paradigms of Artificial Intelligence Programming (hereafter '*Paradigms*'). This evolving post charts my progress on this task.

As a starting point for this retrocomputing challenge, I decided to try and recreate the Eliza system, which in modern terms is a chatbot that can fake a keyboard-based dialogue with a human. It was originally described in a 1966 issue of the *Communications of the Association for Computing Machinery*. Although primitive by today's standards, Eliza was ground-breaking at the time, and an early indicator of the potential (and limitations) of AI. I picked Eliza from the various systems described in *Paradigms* because it's basic function - chat with a person - is easy to understand, but at the same time an excellent test of a useful AI mechanism: pattern matching.

Some aspects of Eliza implementation needed functions that weren't already present in `JKL` 1.0, which was after all based on a minimalist teaching language (MAL). These changes, which went hand-in-hand with the early stages of Eliza development, involved:

* [General enhancements and fixes](https://www.non-kinetic-effects.co.uk/blog/2020/04/03/Journey-continues)
* Adding a [loop capability](https://www.non-kinetic-effects.co.uk/blog/2020/04/18/looping-deep-dive)
* Improving the representation of [environments](https://www.non-kinetic-effects.co.uk/blog/2020/05/03/environments) - the mechanism by which `JKL` associates symbols and their values
 
# Eliza - the first chatbot

The top level of Eliza is a sort of READ-EVAL-PRINT loop:
```
(def! eliza (fn* ()
   (loop ()
      (prn (readline "eliza> "))
      (recur))))
```

... TO BE CONTINUED



