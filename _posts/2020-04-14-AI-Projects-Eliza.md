---
layout: post
title: "AI retrocomputing projects: Eliza [1]"
date: 2020-04-14
---

# Background

At the beginning of 2019 [I decided to get back into coding](https://www.non-kinetic-effects.co.uk/blog/2019/01/01/MAL-1) by building an interpreter for the Lisp programming language. The successful [result](https://www.non-kinetic-effects.co.uk/blog/2019/04/28/MAL-5) was `JKL`, my own version of Lisp, and which is based on the [MAL guidelines](https://github.com/kanaka/mal).

A year later, I decided to pick programming up again, this time using `JKL` to re-implement some of the classic Artificial Intelligence (AI) systems pioneered in the 1960s and 1970s, as described by Peter Norvig in his excellent book Paradigms of Artificial Intelligence Programming (hereafter '*Paradigms*').

This page is a work-in-progress, and I'll update it as and when I make progress on this task.

# Going under the bonnet

`JKL` is a tiny programming language whose decevlopment was intended to teaching interpreter-writing techniques, and it lacks some of the functionality needed to build serious applications. I was therefore going to have to continue to develop the language itself, adding new functions and fixing bugs that emerged from using it in anger.

A [separate post](https://www.non-kinetic-effects.co.uk/blog/2020/04/03/Journey-continues) describes this 'under the bonnet' work.

# Eliza - the first chatbot

As a starting point, I decided to try and recreate the Eliza system, which in modern terms is a chatbot that can fake a dialogue with a human. It was originally described in a 1966 issue of the *Communications of the Association for Computing Machinery*. Although primitive by today's standards, it was ground-breaking at the time, and an early indicator of the potential (and limitations) of AI. I picked Eliza from the various systems described in *Paradigms* because it's basic function - chat with a person - is so easy to describe to people, but at the same time an excellent test of a useful AI mechanism: pattern matching.

... TO BE CONTINUED



