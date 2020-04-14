---
layout: post
title: "AI retrocomputing projects: Eliza [1]"
date: 2020-04-14
---

WORK IN PROGRESS

This page will describe my implementation of the Eliza system - the first artificial intelligence chatbot, as published in the 1966 issue of the *Communications of the Association for Computing Machinery*.

Watch this space.

TODO: write background matter introducing `MAL` and `JKL`.

# Implementation considerations

The PAIP source is implememented using a dialect of Common Lisp circa 1990. `JKL` is a tiny language compared with Common Lisp, and is more directly influenced by the design of Clojure than the original Lisp. This has had three consequences:

* I've sometimes had to implement some Common Lisp functions (e.g. `char`) in `JKL` for direct compatability with the PAIP source

* I've sometimes had to use a different solution to that used by PAIP because `JKL` has a mechanism that provides equivalent or close-enough semantics (e.g. `hashmaps` rather than Common Lisp association lists

* In a few cases, Common Lisp functions used by PAIP have completely different semantics in Clojure and thus in `JKL`. I've decided to prefer the Clojure semantics in these cases to retain consistency with `MAL` and, specifically, the `MAL` regression test suite. Accordingly, I've sometimes had to develop a `JKL` function that achieves the same effect. Notably `atomic?` as the `JKL` implementation of Common Lisp's `atom` function, because `atom` in `JKL` is actually a Clojure-like mutable value

