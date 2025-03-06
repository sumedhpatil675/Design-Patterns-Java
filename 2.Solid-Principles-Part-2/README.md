# SOLID Principles: Advanced Implementation

## Table of Contents
- [Introduction](#introduction)
- [Recap](#recap)
- [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
- [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
- [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
- [Dependency Injection](#dependency-injection)
- [Full Implementation Examples](#full-implementation-examples)

## Introduction

This guide covers the advanced implementation of SOLID principles, focusing on LSP, ISP, DIP, and dependency injection techniques in Java. Each principle is illustrated with practical code examples and evolution of design.

## Recap

From our earlier discussion, we've covered:

1. **Single Responsibility Principle (SRP)**: Each class should have only one reason to change
2. **Open-Closed Principle (OCP)**: Software entities should be open for extension but closed for modification

We started with a `Bird` design that evolved from a monolithic class with if-else blocks to a better abstraction hierarchy.

### Initial Bird Implementation (V0)

```java
public class Bird {
    private String name;
    private String color;
    private String type;
    private double weight;
    
    public void makeSound() {
        if (type.equals("sparrow")) {
            System.out.println("Chi chi");
        } else if (type.equals("crow")) {
            System.out.println("Kaw kaw");
        } else if (type.equals("pigeon")) {
            System.out.println("Gutar gun");
        }
    }
    
    public void fly() {
        System.out.println(name + " is flying");
    }
}
```

### Abstract Bird Implementation (V2)

```java
public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    protected double weight;
    
    public abstract void makeSound();
    
    public abstract void fly();
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

public class Sparrow extends Bird {
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Crow extends Bird {
    @Override
    public void makeSound() {
        System.out.println("Kaw kaw");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Pigeon extends Bird {
    @Override
    public void makeSound() {
        System.out.println("Gutar gun");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}
```

## Liskov Substitution Principle (LSP)

**Definition**: Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.

### Problem: Penguin Can't Fly

Adding a `Penguin` class creates an LSP violation because penguins can't fly:

```java
public class Penguin extends Bird {
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    
    @Override
    public void fly() {
        // This violates LSP since Penguin can't fly
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// This code would break
public void letBirdsFly(Bird[] birds) {
    for (Bird bird : birds) {
        bird.fly(); // Will throw exception for Penguin!
    }
}
```

### LSP Rules

1. Don't surprise the client
2. Don't advertise something you can't do
3. Abstractions should align with implementations

### Solution: Proper Abstraction Hierarchy (V2.2)

```java
public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    protected double weight;
    
    public abstract void makeSound();
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

public abstract class FlyingBird extends Bird {
    public abstract void fly();
}

public abstract class NonFlyingBird extends Bird {
    // No fly method here
}

public class Sparrow extends FlyingBird {
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Penguin extends NonFlyingBird {
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    // No fly method to cause errors
}
```

### LSP Example: Rectangle-Square Problem

A classic LSP violation example involves rectangles and squares:

```java
public class Rectangle {
    protected int height;
    protected int width;
    
    public void setHeight(int h) {
        this.height = h;
    }
    
    public void setWidth(int w) {
        this.width = w;
    }
    
    public int getArea() {
        return height * width;
    }
}

public class Square extends Rectangle {
    // Square overrides setHeight to maintain square properties
    @Override
    public void setHeight(int h) {
        this.height = h;
        this.width = h; // Must set width too to maintain square property
    }
    
    // Square overrides setWidth to maintain square properties
    @Override
    public void setWidth(int w) {
        this.width = w;
        this.height = w; // Must set height too to maintain square property
    }
}

// This would break with Square
public void playWithRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(10);
    
    // For a Rectangle: 5 * 10 = 50
    // For a Square: 10 * 10 = 100 (because setWidth also changes height)
    assert r.getArea() == 50; // This will fail for Square!
}
```

## Interface Segregation Principle (ISP)

**Definition**: Clients should not be forced to depend on interfaces they don't use.

### Problem with Large Interfaces

When we try to add dance capability to flying birds, we might be tempted to create:

```java
// Violates ISP - forces birds to implement multiple behaviors
public interface DanceableFlyable {
    void fly();
    void dance();
}
```

### Solution: Segregated Interfaces (V3)

```java
// Segregated interfaces
public interface Flyable {
    void fly();
}

public interface Danceable {
    void dance();
}

// Bird base class
public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    protected double weight;
    
    public abstract void makeSound();
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

// Implementations
public class Sparrow extends Bird implements Flyable, Danceable {
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
    
    @Override
    public void dance() {
        System.out.println(name + " is dancing");
    }
}

public class Penguin extends Bird implements Danceable {
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    
    @Override
    public void dance() {
        System.out.println(name + " is dancing");
    }
}
```

### ISP with Functional Interfaces

Java 8+ promotes the ISP through functional interfaces:

```java
@FunctionalInterface
public interface Flyable {
    void fly();
}

@FunctionalInterface
public interface Danceable {
    void dance();
}

// Usage with lambda expressions
Flyable sparrowFly = () -> System.out.println("Sparrow flies fast");
Danceable penguinDance = () -> System.out.println("Penguin dances by waddling");
```

## Dependency Inversion Principle (DIP)

**Definition**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

### Problem: Tight Coupling

```java
// Tightly coupled code
public class Sparrow {
    public void fly() {
        public void fly() {
            // Implementation for flying algorithm
            System.out.println("Flying with Sparrow algorithm");
        }
    }
}

public class Client {
    public static void main(String[] args) {
        Sparrow s = new Sparrow();
        s.fly(); // Client depends directly on Sparrow implementation
    }
}
```

### Solution: Depend on Abstractions (V4.1)

```java
// Flight behavior abstraction
public interface FlyingBehavior {
    void makefly();
}

// Concrete implementations
public class HighFlyable implements FlyingBehavior {
    @Override
    public void makefly() {
        System.out.println("Flying high algorithm");
    }
}

public class LowFlyable implements FlyingBehavior {
    @Override
    public void makefly() {
        System.out.println("Flying low algorithm");
    }
}

// Bird uses behavior through abstraction
public class Sparrow extends Bird implements Flyable {
    private FlyingBehavior flyingBehavior;
    
    public Sparrow(FlyingBehavior fb) {
        this.flyingBehavior = fb;
    }
    
    @Override
    public void fly() {
        flyingBehavior.makefly();
    }
}

// Client code
public class Client {
    public static void main(String[] args) {
        // Client configures which implementation to use
        FlyingBehavior fb = new HighFlyable(); 
        Sparrow s = new Sparrow(fb);
        s.fly(); // Uses the injected behavior
    }
}
```

## Dependency Injection

Dependency injection is a technique for implementing DIP. There are three main types:

### 1. Constructor Injection

```java
public class Sparrow {
    private final FlyingBehavior flyingBehavior;
    
    // Constructor injection
    public Sparrow(FlyingBehavior flyingBehavior) {
        this.flyingBehavior = flyingBehavior;
    }
    
    public void fly() {
        flyingBehavior.makefly();
    }
}

// Usage
Sparrow s = new Sparrow(new HighFlyable());
```

**Benefits**:
- Dependency provided at object creation time
- Can make dependencies immutable (final)
- Clear indication of required dependencies

### 2. Method Injection

```java
public class Sparrow {
    private FlyingBehavior flyingBehavior;
    
    // Method injection
    public void setFlyingBehavior(FlyingBehavior flyingBehavior) {
        this.flyingBehavior = flyingBehavior;
    }
    
    public void fly() {
        if (flyingBehavior == null) {
            throw new IllegalStateException("Flying behavior not set");
        }
        flyingBehavior.makefly();
    }
}

// Usage
Sparrow s = new Sparrow();
s.setFlyingBehavior(new LowFlyable()); // Can change behavior at runtime
```

**Benefits**:
- Can switch implementations at runtime
- Object can exist without dependencies initially

### 3. Field Injection (Not Recommended)

```java
public class Sparrow {
    // Field injection (often done with frameworks like Spring)
    @Inject // or @Autowired in Spring
    private FlyingBehavior flyingBehavior;
    
    public void fly() {
        flyingBehavior.makefly();
    }
}
```

**Drawbacks**:
- Hides dependencies
- Makes testing harder
- Cannot use final fields

## Full Implementation Examples

### Strategy Pattern with Flight Behavior

```java
// Interfaces
public interface FlyingBehavior {
    void fly();
}

// Concrete strategies
public class FastFlying implements FlyingBehavior {
    @Override
    public void fly() {
        System.out.println("Flying fast with rapid wing movements");
    }
}

public class SlowFlying implements FlyingBehavior {
    @Override
    public void fly() {
        System.out.println("Flying slowly with gentle wing movements");
    }
}

// Context - Bird classes
public abstract class Bird {
    protected String name;
    protected String color;
    protected double weight;
    
    public abstract void makeSound();
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

public class Sparrow extends Bird {
    private FlyingBehavior flyingBehavior;
    
    public Sparrow(String name, String color) {
        this.name = name;
        this.color = color;
        // Default behavior can be set here
        this.flyingBehavior = new FastFlying();
    }
    
    public void setFlyingBehavior(FlyingBehavior fb) {
        this.flyingBehavior = fb;
    }
    
    public void performFly() {
        flyingBehavior.fly();
    }
    
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
}

public class Crow extends Bird {
    private FlyingBehavior flyingBehavior;
    
    public Crow(String name, String color) {
        this.name = name;
        this.color = color;
        this.flyingBehavior = new FastFlying();
    }
    
    public void setFlyingBehavior(FlyingBehavior fb) {
        this.flyingBehavior = fb;
    }
    
    public void performFly() {
        flyingBehavior.fly();
    }
    
    @Override
    public void makeSound() {
        System.out.println("Kaw kaw");
    }
}

// Client code
public class BirdSimulator {
    public static void main(String[] args) {
        Sparrow sparrow = new Sparrow("Sparky", "Brown");
        sparrow.performFly(); // Uses FastFlying
        
        // Change behavior at runtime
        sparrow.setFlyingBehavior(new SlowFlying());
        sparrow.performFly(); // Now uses SlowFlying
        
        Crow crow = new Crow("Blacky", "Black");
        crow.performFly();
    }
}
```

### Banking Example with DIP

```java
// Interface (abstraction)
public interface Banking {
    void sendMoney(double amount);
    void receiveMoney(double amount);
}

// Low-level modules
public class ICICI implements Banking {
    @Override
    public void sendMoney(double amount) {
        System.out.println("Sending " + amount + " via ICICI");
    }
    
    @Override
    public void receiveMoney(double amount) {
        System.out.println("Receiving " + amount + " via ICICI");
    }
}

public class YesBank implements Banking {
    @Override
    public void sendMoney(double amount) {
        System.out.println("Sending " + amount + " via YesBank");
    }
    
    @Override
    public void receiveMoney(double amount) {
        System.out.println("Receiving " + amount + " via YesBank");
    }
}

// High-level module
public class PhonePe {
    private Banking bank;
    
    // Constructor injection
    public PhonePe(Banking bank) {
        this.bank = bank;
    }
    
    public void transferMoney(double amount) {
        bank.sendMoney(amount);
    }
    
    public void acceptPayment(double amount) {
        bank.receiveMoney(amount);
    }
}

// Client code
public class BankingApp {
    public static void main(String[] args) {
        // Can easily switch between different banks
        Banking icici = new ICICI();
        PhonePe phonePe = new PhonePe(icici);
        
        phonePe.transferMoney(1000);
        
        // Switch to another bank
        Banking yesBank = new YesBank();
        phonePe = new PhonePe(yesBank);
        
        phonePe.transferMoney(2000);
    }
}
```

### String Example Showing Immutability and LSP

```java
public class StringExample {
    public static void main(String[] args) {
        // Immutable Strings vs Mutable StringBuilder
        String s1 = "hello";
        String s2 = new String("hello");
        
        // s1 and s2 are immutable, any operations create new objects
        System.out.println(s1 == s2); // false, different objects
        System.out.println(s1.equals(s2)); // true, same content
        
        // Using StringBuilder (mutable)
        StringBuilder sb = new StringBuilder();
        sb.append("h");
        sb.append("e");
        sb.append("l");
        sb.append("l");
        sb.append("o");
        
        System.out.println(sb.toString()); // "hello"
        System.out.println(sb.toString().equals(s1)); // true
        
        // Character comparison showing ASCII values
        char h = 'e';
        char l = 'l';
        char o = 'o';
        
        // Each character is stored as a numeric value
        System.out.println((int)h); // 101
        System.out.println((int)l); // 108
        System.out.println((int)o); // 111
    }
}
```

## Summary

By applying the SOLID principles correctly:

1. **Single Responsibility Principle (SRP)**: Each class has one reason to change
2. **Open-Closed Principle (OCP)**: Open for extension, closed for modification
3. **Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types
4. **Interface Segregation Principle (ISP)**: Many specific interfaces are better than one general-purpose interface
5. **Dependency Inversion Principle (DIP)**: Depend on abstractions, not concretions

We can create more maintainable, flexible, and robust software systems.