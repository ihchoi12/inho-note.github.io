---
title: "C++ & JAVA - type casting"
date: 2021-03-26
categories:
  - C++
  - JAVA
tags:
  - type casting
---

## Basic: variable
- Definition: a location of computer's memory (size depends on the variable type) 
- Two components: 1) (starting) address; 2) value;

## Three (different) kinds of type : Primitive, Reference, Pointer
##### Primitive variable (C++, JAVA)
- Contains value
- **Primitive Conversion** menas irreversible changes of its value
```java
double myDouble = 1.1;
int myInt = (int) myDouble;
        
assertNotEquals(myDouble, myInt);
```

##### Reference variable (only JAVA)
- ALL object variables in JAVA are references (the value is the address, it may not be the actual memory address though [3, 4]) 
- Only refers to an object (NOT contain the object itself)
- **reference variable casting** does NOT touch the object it refers to, but only labels this object in another way
- It may expands (downcasting) or narrows (upcasting) opportunities to work with it
> A reference is like a remote control to an object. The remote control has more or fewer buttons depending on its type, and the object itself is stored in a heap. When we do casting, we change the type of the remote control but don’t change the object itself.

##### Reference variable (only C++)
- Another name to an existing variable (cannot reference to null)
- Implicit referencing and dereferencing (unlike pointers)
- Must be initialized once declared (cannot change the referencing target after then)
- Reference target can be a function
```
int foo(double i) {
  return 2;
}

// syntax: RETURN_TYPE (&REFERENCE_VAR_NAME) (PARAMETER_LIST) = REFERENCED_FUNCTION;
int(&ref)(double) = foo;
ref(2.0);
```

##### Pointer variable (only C++)
- Take the address of the other variable as its value
- * is used to access the value of the variable that the pointer is pointing
- We can use *pointer arithmetic*: moving the pointer location (i.e., its value, that is, address of the target variable)
- Target can be a function
```
int foo(double i) {
  return 2;
}

// syntax: RETURN_TYPE (*POINTER_NAME) (PARAMETER_LIST) = REFERENCED_FUNCTION;
int(*func_foo)(double) = foo;
func_foo(2.0);
```


## Reference or Pointer Variable Casting
##### Upcasting
- Implicitly performed by the compiler 
```
/* Supppose Cat extends Animal */

// [JAVA]
Cat cat = new Cat();
// following two lines are same
Animal animal = cat;
animal = (Animal) cat;

// [C++]
Animal* animal;
Cat cat();
animal = &cat;

/* The cat instance itself does NOT change */
/* But, we cannot invoke methods in Cat class only from the animal variable */
```
- Narrow down opportunities to work with it. 
- Then, why do upcasting? Advantage: **Polymorphism** 

##### Polymorphism
- Use of a single symbol (e.g., Class, Object, interface, etc.) to represent multiple different types
- Advantage1: can handle multiple different types by a single type (i.e., can use generic functions for multiple derived classes)
```
List<Animal> animals = new ArrayList<>();
animals.add(new Cat());
animals.add(new Dog());
new AnimalFeeder().feed(animals);
```
- C++: since the memory size of a pointer is decided by the type, base class pointer cannot access the memory of derived class (i.e., members of the derived class)

![image](https://user-images.githubusercontent.com/37887404/113405871-1b4cb700-93dd-11eb-9aef-8ca0332e21e4.png)

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
[1] References in C++: https://www.tutorialcup.com/cplusplus/references.htm
[2] https://www.baeldung.com/java-type-casting
[3] Memory Address of Objects in Java: https://www.baeldung.com/java-object-memory-address
[4] https://stackoverflow.com/questions/18396927/how-to-print-the-address-of-an-object-if-you-have-redefined-tostring-method

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
