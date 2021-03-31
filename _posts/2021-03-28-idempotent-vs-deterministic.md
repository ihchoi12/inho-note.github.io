---
title: "Ideompotent vs Deterministic"
date: 2021-03-26
categories:
  - Distributed Systems
tags:
  - Deterministic
  - Ideompotent
---

## Idempotent
- Internal state only changes as a result of the first message (even if duplicates exist)
- f(x) = f(f(x)) (where f is the application of the algorithm for a given input and x is the initial state)

## Deterministic
- Internal state changes is only a function of the sequence of inputs
- Algorithm == f(x) (where x is the initial state)



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
