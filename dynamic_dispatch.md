Dynamic Dispatch
================
Because of the Liskov substitution, all variable of reference type (e.g. all non-primitive type variable in Java / pointer or reference type in c++) have both static (compile-time) type and dynamic (run-time) type.

Now consider methods (in Java, or function members in C++) to be simple functions with a special implicit parameter, i.e. `this`. Then the program becomes "types (collection of primitive types)" + "functions that work on those types (use those types as parameters)".

A common practice in programming is to have some functions of the same name (identifier text). This is because essentially those functions conceptually perform the same computation, but might slightly differ in input.

Dynamic dispatch refers to the ability of the programming language to choose, among functions of the same name, the "right" function to execute, based on the DYNAMIC TYPE of the arguments supplied in the function invocation (among other factors like # of args). The fact that such choice is based on the dynamic type of arguments enforces that such choice must also happen at run time.

All OO programming langauge supports dynamic dispatch to a certain extent.

Single Dispatch
===============
Most OO programming languages, e.g. Java, C++, do such choice at run time only based on the first special parameter i.e. `this`.

```java
interface Speakable { void speak();}
class Dog implements Speakable { @Override public void speak() { System.out.println("woof woof"); }}
class Cat implements Speakable { @Override public void speak() { System.out.println("meow meow"); }}

// imagine those 2 overrides as functions of the same name "speak":
// void speak(Dog this);
// void speak(Cat this);

public class App {
    public static void main(String[] args) {
        Speakable s = null;
        if (/* initialize a dog or a cat based on user input*/) {
            s = new Dog();
        } else {
            s = new Cat();
        }
        s.speak(); // jvm dispatch/resolve this invocation to actual function implementation
    }
}
```

And they don't further inspect the dynamic type of other arguments (i.e. the 2nd, the 3rd ...):

```java
interface Speakable {
    void speak();
    void eat(Apple a);
    void eat(Banana b);
}
class Dog implements Speakable {
    @Override public void speak() { System.out.println("woof woof"); }
    @Override public void eat(Apple a) { System.out.println("woof eating apple"); }
    @Override public void eat(Banana b) { System.out.println("woof eating banana"); }
}
class Cat implements Speakable {
    @Override public void speak() { System.out.println("meow meow"); }
    @Override public void eat(Apple a) { System.out.println("meow eating apple"); }
    @Override public void eat(Banana b) { System.out.println("meow eating banana"); }
}

// imagine those overrides as:
// void eat(Dog this, Apple a)
// void eat(Dog this, Banana b)
// void eat(Cat this, Apple a)
// void eat(Cat this, Banana b)

abstract class Fruit {}
class Apple extends Fruit {}
class Banana extends Fruit {}

public class App {
    public static void main(String[] args) {
        Speakable s = null;
        if (/* initialize a dog or a cat based on user input*/) {
            s = new Dog();
        } else {
            s = new Cat();
        }
        s.speak(); // jvm dispatch/resolve this invocation to actual function implementation

        Fruit f = null;
        if (/* initialize an apple or a banana based on user input*/) {
            f = new Apple();
        } else {
            f = new Banana();
        }
        s.eat(f); // An compile-time error!
    }
}
```

Because those languages only "dispatch" based on the first argument, they are called "single dispatch" langauges.

Double Dispatch
===============
Double dispatch is the ability of a programming language to dispatch based on the dynamic type of both `this` and the second argument.

We know it's ideal to use super-type variable to refer to a sub-type, and let the language choose the actual method implementation for us upon the invocation of a method of that super-type.

```java
List<Integer> myList = new ArrayList<>();
```

Using an super-type to refer to a sub-type, instead of using the sub-type directly, brings several benefits:

1. We limits ourselves to a smaller set of function available in the interface than of the actual type.
2. Client code that use interface is more loosely coupled with the implementation, i.e. changing the underlying implementation without changing the client code is possible.

But the regretful thing is that, in single dispatch language, only the first argument `this` can be a super-type while enjoying the run time dispatch, while all other arguments enjoys nothing.

Visitor Pattern
==================
visitor pattern feels like outsourcing the logic of a method to other class

TODO: the relationship between visitor pattern and double dispatching

I bet visitor pattern does NOT necessarily involves a simulated double dispatch.

I don't agree "visitor pattern is implementing the double dispatching". (https://stackoverflow.com/a/255224/8311608)

I bet double dispatched is only simulated / resembled when visitor and visitee are coupled both through interface, i.e. the visitee sees a visitor of interface type, and the visitor sees a visitee of interface type:
```java
interface Visitee { void accept(Visitor v); }

interface Visitor { void visit(Visitee ve); }
```

Double dispatch happens

Why Sanders use visitor pattern to traverse the AST
==============================
If we don't use visitor pattern, the logic of "how to interpret a chunk" has to be written in `class Chunk` itself, maybe adding a method `void interpret()`. This requires us to add a new method `void interpret()` in `abstract class ASTNode` since all AST nodes will need to have such a method. Since each class has a separate source file, the logic of traversing an AST is distributed / shattered among different places, making the maintanance harder.

Also if we don't use visitor pattern, it's hard to add a new way of interpreting the AST. Say, one day we want to test a new associativity of an operator, while keeps the original interpreter logic intact. Using visitor pattern, this is simply adding a new visitor, without changing each sub-class of `ASTNode`.

Visitor pattern is really useful when you want "conceptually" add new method to a class without even modifying the code of that class. And with a hierarchical object structure like AST, it further allows you to collect all algorithmic logic in a single class.

