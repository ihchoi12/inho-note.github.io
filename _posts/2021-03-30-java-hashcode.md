---
title: "JAVA - hashcode()"
date: 2021-03-26
categories:
  - JAVA
tags:
  - hashcode()
---

## Intro: when hash should be useful?
- Sometimes, naive operations are inefficient (example below)
```
List<String> words = Arrays.asList("A", "B", "C");
if (words.contains("C")) {
    System.out.println("C is in the list");
}
```
- Some data structure (e.g., Map) handles these issues using **hash tables**
- **hashcode()** calculate the hash value for a given **key** in hash tables (values can be accessed efficiently!)





## Commonality:
- Form a contract that any subclasses which inherits from them **have to implement their abstract methods** (i.e., methods w/o implementation). 
- Cannot be instantiated as itself
- Can be instantiated using its subclass (e.g., InterfaceA varA = new SubClassA() or AbstractClassA varA = new SubClassA())

## Difference:
##### Abstract Class: 
- *extends*
- (usually) Contains at least one abstract method
- Possible to have no methods (reason: to prevent it from being instantiated on its own)
- NOT allow multiple inheritance
- Can define member variables
- When to use? To define superclass with some common methods (concrete methods) for all subclasses
(taking advantage of code reusability and reduced implementation cost here) 
while giving some flexibilities of some methods' (abstrace methods) implementation to subclasses  

##### Interface: 
- *implements*
- Contains only abstract methods
- Allow multiple inheritance
- ALL member variables are (implicitly) public, static and final
- When to use? When we have multiple subclasses having methods with common interface (name, parameter) yet unique implementation only.

##### References
https://www.baeldung.com/java-hashcode

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
