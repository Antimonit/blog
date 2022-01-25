---
layout: post
title:  "Trailing commas"
date:   2022-01-25 09:10:00 +0900
tags:   programming
---

A colleague recently commented on one of my pull requests that I had left unnecessary trailing commas scattered around code after refactoring. She was not aware of their intention, hence this article exists. These commas were left in the code deliberately and by design.

What once used to be a syntax error is now a feature. Check out [What's new in Kotlin 1.4](https://kotlinlang.org/docs/whatsnew14.html#trailing-comma).

These commas do not change the program's behaviour in any way—they are just syntactic sugar. The program compiles to the same bytecode with or without the trailing comma. But this seemingly insignificant feature comes with a few not so obvious quality-of-life improvements.

## Cut and paste

A minuscule benefit of trailing comma manifests when we cut and paste parameters, typically when reordering parameters. When the list of parameters does not end with a trailing comma, we have to spend an extra moment to add/remove a comma when we add/remove a parameter at the end of the list.

<p align="center">
  <img src="/assets/images/trailing_comma/cut_and_paste.png" width="60%"/>
</p>

## Code reviews

Trailing comma particularly excels in code reviews. Utilizing trailing commas leads to more concise code changes.
As a result, it becomes faster to scan the code and realize that there was only a single line added and a single line removed:

<p align="center">
  <img src="/assets/images/trailing_comma/changes_with.png" />
</p>

rather than having to mentally filter out unnecessary comma juggling in unrelated lines:

<p align="center">
  <img src="/assets/images/trailing_comma/changes_without.png" />
</p>

## Merge conflicts

In my opinion, the greatest benefit of using a trailing comma is that it prevents a very common type of merge conflict.
To give a trivial example, let's have a look at what may happen when we try to merge two branches (or rebase one on top of another) with a single commit each:
<!-- To give a trivial example of such situation, imagine we try to merge two branches (or rebase on top of another) with a single commit each: -->
<!-- To give a trivial example, imagine two commits in two independent branches that -->

1. Add `middleName` in the middle of the list of parameters.
2. Remove `isActive` from the end of the list of parameters.

<!-- Now, when we try to merge these branches or rebase one on top of another, -->
With a trailing comma, we are presented with a clean set of addition and deletions that do not interfere with each other.

<p align="center">
  <img src="/assets/images/trailing_comma/conflict_with.png" />
</p>

But without a trailing comma, we are presented with a dreaded conflict that we have to resolve manually.

<p align="center">
  <img src="/assets/images/trailing_comma/conflict_without.png" />
</p>

This conflict occurred because we removed a comma from the `lastName` parameter although all we intended was just to remove the `isActive` parameter instead. Sure enough, we can quickly resolve this trivial example using the magic wand button in the gutter, but it is not hard to imagine a situation where the changes are intertwined with other commits, in which case the magic wand becomes no longer available.
