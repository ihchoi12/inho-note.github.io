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

## How hashCode() works?
- returns an integer value
- Equal Objects (by *equals()*) must return the same hash code
- Collisions can happen, but ensuring distinct results for unequal objects improves the performance of hash tables
- hashCode() defined by **class** Object returns distinct integers for distinct objects
- During a single execution of Java app, hashCode() invoked on the same object returns the same value (but the value can be inconsistent across executions)

## How to implement efficient hashCode()?
- Several standard options: IntelliJ, Eclipse, Lombok, etc.
- Those utilize number 31: prime and bitwise multication friendly
```
31 * i == (i << 5) - i
```

## How to handle collisions?
- Various methodologies with their own pros and cons
- ex. *HashMap* uses the *separate chaining method*: each bucket is maintained as a linked-list
- Java 8 improve it: if a bucket size over the certain threshold, replace it with tree map, i.e., O(n) to O(log n)


##### References
https://www.baeldung.com/java-hashcode

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
