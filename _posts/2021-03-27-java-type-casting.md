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
- Then, why do upcasting? Advantage: **Polymorphism** 


##### Polymorphism
- Use of a single symbol (e.g., Class, Object, interface, etc.) to represent multiple different types
- Advantage: can handle multiple different types by a single type 
```
List<Animal> animals = new ArrayList<>();
animals.add(new Cat());
animals.add(new Dog());
new AnimalFeeder().feed(animals);
```


##### Downcasting
- Casting from a superclass to a subclass
- (unlike upcasting) Done using cast operator
- When to use? to gain access to members **specific to subclass**
```
((Cat) animal).meow();
```

##### Overriding
- Providing a specific implementation of a method in a subclass provided by its superclass
- Although a subclass var is assigned to its superclass var, calling methods **invokes on the real object** (i.e., subclass) 


## Related Operators
- *instanceof*: check the specific type of an object
```
if (animal instanceof Cat) {
    ((Cat) animal).meow();
}
```
- *cast() & isInstance()*: methods of object **Class** (useful when we handle generic types since we can treat them as Class<T>)

##### References
https://www.baeldung.com/java-type-casting

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
