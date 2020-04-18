---
layout: post
title: "AI retrocomputing projects: Eliza [1]"
date: 2020-04-14
---

# Background

At the beginning of 2019 [I decided to get back into coding](https://www.non-kinetic-effects.co.uk/blog/2019/01/01/MAL-1) by building an interpreter for the Lisp programming language, following the excellent [MAL guidelines](https://github.com/kanaka/mal). This successful [exercise](https://www.non-kinetic-effects.co.uk/blog/2019/04/28/MAL-5) yielded `JKL`, my own version of Lisp.

A year later, I decided to use `JKL` to re-implement one of the classic Artificial Intelligence (AI) systems pioneered in the 1960s and 1970s, as described by Peter Norvig in his excellent book Paradigms of Artificial Intelligence Programming (hereafter '*Paradigms*'). My primary goal was to conduct an interesting exercise in retrocomputing. This evolving post charts my progress on this task.

I also wanted to give `JKL` a serious work-out, given that it is a tiny programming language that lacks some of the functionality needed to build serious applications. A [separate post](https://www.non-kinetic-effects.co.uk/blog/2020/04/03/Journey-continues) describes the enhancements (and fixes) that I made as a result of using `JKL` for application development. Some of these enhancements turned into deep-dives in their own right, notably [adding a loop capability](https://www.non-kinetic-effects.co.uk/blog/2020/04/18/looping-deep-dive).

# Eliza - the first chatbot

As a starting point for this retrocomputing challenge, I decided to try and recreate the Eliza system, which in modern terms is a chatbot that can fake a keyboard-based dialogue with a human. It was originally described in a 1966 issue of the *Communications of the Association for Computing Machinery*. Although primitive by today's standards, it was ground-breaking at the time, and an early indicator of the potential (and limitations) of AI. I picked Eliza from the various systems described in *Paradigms* because it's basic function - chat with a person - is easy to understand, but at the same time an excellent test of a useful AI mechanism: pattern matching.

... TO BE CONTINUED



