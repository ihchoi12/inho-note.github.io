---
title: "JAVA - type casting"
date: 2021-03-26
categories:
  - JAVA
tags:
  - type
  - casting
---

## Two (different) kinds of type in JAVA: Primitive & Reference
##### Primitive variable
- Contains value
- **Primitive Conversion** menas irreversible changes of its value
```
double myDouble = 1.1;
int myInt = (int) myDouble;
        
assertNotEquals(myDouble, myInt);
```
##### Reference variable
- Only refers to an object (NOT contain the object itself)
- **reference variable casting** does NOT touch the object it refers to, but only labels this object in another way
- It may expands (downcasting) or narrows (upcasting) opportunities to work with it
> A reference is like a remote control to an object. The remote control has more or fewer buttons depending on its type, and the object itself is stored in a heap. When we do casting, we change the type of the remote control but don’t change the object itself.

## Reference Variable Casting
##### Upcasting
- implicitly performed by the compiler 
```
Cat cat = new Cat();
// Supppose Cat extends Animal, following two lines are same
Animal animal = cat;
animal = (Animal) cat;
// The cat instance itself does NOT change
// But, we cannot invoke methods in Cat class only from the animal variable
```
- Then, why do upcasting? Advantage: **Polymorphism** //HERE



##### Downcasting

##### References
https://www.baeldung.com/java-type-casting

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
