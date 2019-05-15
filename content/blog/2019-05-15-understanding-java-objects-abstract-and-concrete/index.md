---
title: Understanding Java Objects, Abstract and Concrete
date: "2019-05-15T16:25:53.595Z"
description: Trying to understand Java's sprawl, one constructor at a time. 
---

I've been working a lot in Java lately, but only when my project required me to adopt some of the famous object-oriented design patterns did I realise how totally unsure I was of what was going on at a fundamental level, so I found it a real challenge to keep up with all the terminology. I felt like I had hit a brick wall.

I decided to [invest some time](https://docs.oracle.com/javase/tutorial/java/TOC.html) in unpacking some of those building blocks - so let's blow the doors off some of this Java jargon. I'm hoping that if you've ever felt in a similar position that these notes might be able to help further your own understanding of Java's sprawl.

### Java Jargon #1: Creating Objects

In Java, almost everything is an object - such as `String`, `ArrayList` and `HashMap`. Objects are created from classes. Think of your class code as an instruction manual for Java to produce an object.

Creating an object requires creating an instance of a class - so the phrase *instantiating a class* is often used by people who want to sound especially smart. But you can use both terms interchangeably, depending on who you're talking to or if you're trying to win a game of Scrabble.

To formally create an object/instantiate a class, which is to say what we have to do in order to stop the compiler from going full on Mr Resetti at us, we follow a three-step process: declaration, instantiation and initialisation.

**Declaration** is achieved by specifying the `type` and the `name` of  a variable, so we could tell Java `Dragon viserion;`, which would get everything prepped and primed to accept a `Dragon` type. But at this point our object is currently closer to my checking account than a fearsome dragon: it's empty. Nothing in it. Zip, nada, nill or, specifically, `null`.

Always watch out for `null`. If I had kids, I wouldn't like to see them hanging around with `null`.

For the record: Java also has both *primitive* and *reference* variables, which we can wave our hands at now but can potentially create some additional bother every now and then, generally when it comes to assignment and comparison. But, hey, we're programming in Java so it's probably safe to assume we're all kind of into a spot of bother every now and then.

Anyway, we're ready for steps 2 and 3 - instantiation and initialisation, which like to rock up as a pair:

```java
viserion = new Dragon();
```

The `new` operator **instantiates** the class, allocating the required memory behind the scenes and then returning a reference (the address) of that memory so our `viserion` variable knows where to send anyone who comes knocking. Calling `new` also requires a call to a constructor, which is where the parenthesis enter the scene. The `()` part handles **initialisation**, which goes sniffing out a class constructor that matches the signature and finally gets our object fully setup and ready to come out.

Constructors in a Java class have the same name as the class and no return type. If we take a look at our `Dragon` class we can see that it has two constructor signatures: one which can respond to a zero-argument constructor, which will in turn call our two-argument constructor that requires a `Stomach` and `Mouth` argument with defaults.

```java
public class Dragon {
  private Stomach stomach;
  private Mouth mouth;

  public Dragon() {
    this(new BigStomach(), new BigMouth());
  }

  public Dragon(Stomach stomach, Mouth mouth) {
    this.stomach = stomach;
    this.mouth = mouth;
  }

  public eat(Food food) {
    mouth.consume(food);
    stomach.digest(food);
  }

  public dracarys(Target target) {
    return mouth.breatheFire(target);
  }
}
```

Constructors are mandatory for classes, but if one isn't explicitly defined then the compiler will treat the class to a zero-argument [default constructor](https://en.wikipedia.org/wiki/Default_constructor) totally free of charge.

We've also split declaration and instantiation/initialisation of `viserion` across two lines, which isn't necessary - we can also declare all of our assignment on a single line if we'd like. We don't even need to assign our objects to variables, either, and can instead use them directly in an expression. Let's bring all of that together and create another `Dragon`:

```java
Dragon drogon = new Dragon(new GiantStomach(), new GiantMouth());
```

We're declaring a reference variable `drogon`, instantiating our Dragon class with `new` and then initialising it with our constructor which responds to the `Stomach` and `Mouth` signature. We're also instantiating two new objects, `GiantStomach` and `GiantMouth`, directly in our constructor call.

Now, how about a real example? Look at how `ArrayList` features three constructor signatures in the Java source code:

```java
// Constructs an empty list with the specified initial capacity.
public ArrayList(int initialCapacity) {
  if (initialCapacity > 0) {
    this.elementData = new Object[initialCapacity];
  } else if (initialCapacity == 0) {
    this.elementData = EMPTY_ELEMENTDATA;
  } else {
    throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
  }
}

// Constructs an empty list with an initial capacity of ten.
public ArrayList() {
  this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// Constructs a list containing the elements of the specified collection, in the order they are returned by the collection's iterator.
public ArrayList(Collection<? extends E> c) {
  elementData = c.toArray();
  if ((size = elementData.length) != 0) {
    if (elementData.getClass() != Object[].class)
      elementData = Arrays.copyOf(elementData, size, Object[].class);
  } else {
    this.elementData = EMPTY_ELEMENTDATA;
  }
}
```

Yikes, that was quite a lot to take in. Consider it work on the foundations. If you're anything like me, you often *know* this stuff without actually, you know, *knowing* it, so it's nice to have a refresher.

### Java Jargon #2: Concrete Classes

This word pops up all over the place and I've found you're supposed to know what it means by default. For the record, I didn't really know what it meant for **months**.

To summarise it in a sentence, a concrete class is any class you can create (instantiate) with the `new` keyword. Concrete classes have *all* of their methods implemented, regardless of however many interfaces they `implement` or classes they `extend`.

There's no specific language keywords needed when it comes to making something concrete, but you'll probably see the word pop-up a lot in UML diagrams and people shooting the breeze about object-oriented programming, so it's nice to know what's going on so you can take part in the conversation.

Our `Dragon` class is bona fide, 100% certified concrete, which is totally easy to do when you're not using anything abstract. I find it easier to think of classes as concrete by default and only really in terms of concrete and abstract when using either inheritance (via `extends`) and interfaces (via `implements`).

### Java Jargon #3: Interfaces

If a concrete class is an instruction manual, an interface is a blueprint - it's a list of unimplemented method signatures that, by stating we will `implement` the interface, our class is pledging to support. When a class opts to `implement` an interface, this is often said to be a **contract** - because the class that's implementing is essentially promising to include these methods.

Not fulfilling an Interface contract will also have the compiler visit you in the dead of night, provided you're compiling your code at night. It won't be happy about it.

A Java class can implement many interfaces, as you can see with `ArrayList`:

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
```

You'll spot a lot of the '-able' interfaces when you work in Java - here we can see `Cloneable` and `Serializable`, and you'll see these pop up time and time again, but for now we'll focus on `List`. Notice how `List` uses the `interface` keyword instead of `class`:

```java
public interface List<E> {
    int size();
    boolean isEmpty();
    //... there's plenty more empty methods
```

Neither `size()` not `isEmpty()` are implemented in this interface. We couldn't create a `List` object from this interface with the `new` keyword, which is why it's not a concrete class. But, by implementing `List`, `ArrayList` pledges to follow the contract and ensure it knows how to respond when sent requests for any of the interface method signatures.

Interfaces are like a little bow tie for some of your classes - they add a neat degree of formality, which the type checker respects. You can use interface names as the type for your reference variables, too, which means any class implementing that interface will be allowed to be assigned to it.

### Java Jargon #4: Abstract Classes

Abstract classes can have both implemented and unimplemented methods, but they're not concrete because they can't create an object with the `new` keyword. However, subclasses can use the `extends` keyword to inherit their functionality.

Many of Java's List collections inherit from the abstract class `AbstractList`:

```java
// AbstractList.java
public abstract class AbstractList<E> {
  //...
  public boolean add(E e) {
    add(size(), e);
    return true;
  }

  public abstract E get(int index);
```

Notice the `abstract` keyword used to denote both an abstract class and an abstract method: use of the latter requires declaration of the former. Also notice how the abstract method `get`  lacks braces and features a semicolon, similar to the way we declared our interface methods earlier.

`ArrayList` is a subclass of `AbstractList`, and so has to implement a `get` method but will inherit the `add` method:

```java
// ArrayList.java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
  // ...
  public E get(int index) {
    Objects.checkIndex(index, size);
    return elementData(index);
  }
```

If `ArrayList` did not implement its own `get` method while extending `AbstractList` it would have to declare its class `abstract` in order to satiate the compiler, which would mean it could not create objects using the `new` keyword and would also be unable to reach that much-desired concrete status.

### Wrapping Up

We've taken a look at what makes a concrete and abstract class in Java, which meant familiarising ourself with how classes are instantiated. We moved from a simple example `Dragon` class to looking at some slices of source code from JDK 12, taking a look at how `ArrayList` extends `AbstractList` and implements `ArrayList`.

Understanding these building blocks creates a massively useful foundation for getting to grips with the strict object-oriented nature some of Java's most prominent design patterns.
