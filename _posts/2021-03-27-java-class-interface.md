---
title: "JAVA - abstract class vs interface"
date: 2021-03-26
categories:
  - JAVA
tags:
  - interface
  - abstract class
---

## Common:
- Form a contract that any subclasses which inherits from them **have to implement their abstract methods** (i.e., methods w/o body). 
- Cannot be instantiated as itself

## Difference:
##### Abstract Class: \
- *extends*
- (usually) Contains at least one abstract method
- Possible to have no methods (reason: to prevent it from being instantiated on its own)
- When to use? To define superclass with some common methods (general methods) for all subclasses 
while giving some flexibilities of some methods' (abstrace methods) body to subclasses  

##### Interface: \
- *implements*
- Contains only abstract methods
- When to use? When we have multiple subclasses having methods with common interface (name, parameter) yet unique body only.

##### References
https://www.youtube.com/watch?v=2aQ9Y7bumts

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
