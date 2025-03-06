# SOLID Principles in Software Design

## Table of Contents
- [Introduction](#introduction-to-solid-principles)
- [Single Responsibility Principle (SRP)](#single-responsibility-principle-srp)
- [Open-Closed Principle (OCP)](#open-closed-principle-ocp)
- [Liskov Substitution Principle (LSP)](#liskov-substitution-principle-lsp)
- [Interface Segregation Principle (ISP)](#interface-segregation-principle-isp)
- [Dependency Inversion Principle (DIP)](#dependency-inversion-principle-dip)
- [Design Evolution](#design-evolution)

## Introduction to SOLID Principles

SOLID is an acronym for five design principles that help engineers create better software systems. These principles serve as guidelines that should be followed as much as possible to create well-designed software.

### What are SOLID Principles?
- **S**: Single Responsibility Principle
- **O**: Open-Closed Principle
- **L**: Liskov Substitution Principle
- **I**: Interface Segregation Principle
- **D**: Dependency Inversion Principle

### Characteristics of Well-Designed Software
- **Extensible**: Easy to add new features
- **Maintainable**: Easy to fix bugs and keep things running
- **Readable**: Easy to understand the code
- **Testable**: Easy to verify functionality
- **Reusable**: Components can be used in multiple contexts
- **Reliable**: Consistent and dependable performance
- **Scalable**: Can handle growth in users or data

## Single Responsibility Principle (SRP)

### Core Concept
Every code unit (method, class, package) should have exactly one responsibility and exactly one reason to change.

### Example Problem: Bird Design (Version 0)

The initial implementation shows a `Bird` class with multiple responsibilities:

```java
// Version 0: Violating SRP
public class Bird {
    private String name;
    private String color;
    private String type;
    private int numberOfWings;
    private int numberOfLegs;
    private String region;
    private double weight;
    
    // Constructor and getters/setters omitted for brevity
    
    public void fly() {
        System.out.println(name + " is flying");
    }
    
    public void makeSound() {
        if (name.equals("sparrow")) {
            System.out.println("Chi chi");
        } else if (name.equals("crow")) {
            System.out.println("Caw caw");
        } else if (name.equals("pigeon")) {
            System.out.println("Gutar gun");
        }
        // What if we add more birds? We'd need to modify this method
    }
    
    public void dance() {
        System.out.println(name + " is dancing");
    }
}
```

### Problems with SRP Violations
1. **Understandability**: Code becomes harder to comprehend
2. **Hard to debug**: Errors are difficult to trace
3. **Code duplication**: Similar logic repeated in different places
4. **Maintainability issues**: Changes affect multiple functionalities
5. **Testing difficulties**: Complex units are harder to test

### Common SRP Code Smells
1. **Methods with many if-else blocks**: Indicates handling too many cases
2. **Monster methods**: Methods doing far more than what their name suggests
3. **Utility/Common packages**: Become dumping grounds for unrelated functionality

### Monster Method Example
```java
public void saveToDB(User user) {
    // Creating query
    String query = "INSERT INTO users VALUES ('" + 
                  user.getName() + "', '" + 
                  user.getEmail() + "')";
    
    // Establishing connection
    Database db = new Database();
    db.connect();
    db.login();
    
    // Executing query
    db.execute(query);
}
```

This should be refactored to:

```java
public void saveToDB(User user) {
    String query = getQuery(user);
    Database db = establishConnection();
    db.execute(query);
}

private String getQuery(User user) {
    return "INSERT INTO users VALUES ('" + 
           user.getName() + "', '" + 
           user.getEmail() + "')";
}

private Database establishConnection() {
    Database db = new Database();
    db.connect();
    db.login();
    return db;
}
```

### Utility Package Problem
Packages like "utils" often become catch-alls. Better approach:

```
utils/
  ├── interestUtils/  // All interest calculation related classes
  └── timezoneUtils/  // All timezone related classes
```

## Open-Closed Principle (OCP)

### Core Concept
Software entities should be open for extension but closed for modification. Adding new features shouldn't require changing existing code.

### Bird Example Continued (Version 1)
In the initial design, adding a new bird type would require modifying the `makeSound()` method, which violates OCP.

```java
// Version 1: Still violating OCP
public class Bird {
    private String name;
    private String color;
    private String type;
    // Other attributes omitted for brevity
    
    public void makeSound() {
        if (name.equals("sparrow")) {
            makeSparrowSound();
        } else if (name.equals("crow")) {
            makeCrowSound();
        } else if (name.equals("pigeon")) {
            makePigeonSound();
        } else if (name.equals("peacock")) {
            makePeacockSound(); // Added a new bird, had to modify code
        }
    }
    
    private void makeSparrowSound() {
        System.out.println("Chi chi");
    }
    
    private void makeCrowSound() {
        System.out.println("Caw caw");
    }
    
    private void makePigeonSound() {
        System.out.println("Gutar gun");
    }
    
    private void makePeacockSound() {
        System.out.println("Mayaaw");
    }
}
```

### Solution: Abstract Class and Inheritance (Version 2)

```java
// Version 2: Adhering to OCP with abstraction
public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    protected double weight;
    
    // Constructor and getters/setters omitted for brevity
    
    public abstract void makeSound();
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public abstract void fly();
}

public class Sparrow extends Bird {
    public Sparrow(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Sparrow";
    }
    
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
    public Crow(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Crow";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Caw caw");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Pigeon extends Bird {
    public Pigeon(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Pigeon";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Gutar gun");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Peacock extends Bird {
    public Peacock(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Peacock";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Mayaaw");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}
```

### Why OCP Matters
1. **Testing**: Changes to existing code could break functionality
2. **Regression**: Prevents breaking existing code when adding features

## Liskov Substitution Principle (LSP)

### Core Concept
Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.

### Problem Example: Penguin (Version 2.1)
When adding a Penguin class that inherits from Bird, it can't implement `fly()` properly.

```java
// Version 2.1: Violating LSP
public class Penguin extends Bird {
    public Penguin(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Penguin";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    
    @Override
    public void fly() {
        // Penguins can't fly!
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// This would cause problems:
public void letBirdsFly(Bird[] birds) {
    for (Bird bird : birds) {
        bird.fly(); // Throws exception for Penguin!
    }
}
```

### LSP Violation Rules
1. Don't surprise the client
2. Don't advertise something you can't do
3. Abstractions should not be out of sync with implementations

### Solution: Better Abstraction Hierarchy (Version 2.3)

```java
// Version 2.3: Adhering to LSP with better abstraction
public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    protected double weight;
    
    // Constructor and getters/setters omitted for brevity
    
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
    public Sparrow(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Sparrow";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
    
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
}

public class Crow extends FlyingBird {
    // Implementation details...
}

public class Pigeon extends FlyingBird {
    // Implementation details...
}

public class Peacock extends FlyingBird {
    // Implementation details...
}

public class Penguin extends NonFlyingBird {
    public Penguin(String name, String color) {
        this.name = name;
        this.color = color;
        this.type = "Penguin";
    }
    
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    
    // No fly method to cause errors
}

public class Ostrich extends NonFlyingBird {
    // Implementation details...
}
```

## Interface Segregation Principle (ISP)

### Core Concept
Clients should not be forced to depend on interfaces they don't use. Many specific interfaces are better than one general-purpose interface.

### Example: Bird Behaviors

Instead of having all behaviors in the Bird abstract class, we can use interfaces:

```java
public interface Soundable {
    void makeSound();
}

public interface Flyable {
    void fly();
}

public interface Swimmable {
    void swim();
}

public abstract class Bird {
    protected String name;
    protected String color;
    protected String type;
    
    // Common behavior for all birds
    public void eat() {
        System.out.println(name + " is eating");
    }
}

public class Sparrow extends Bird implements Flyable, Soundable {
    @Override
    public void fly() {
        System.out.println(name + " is flying");
    }
    
    @Override
    public void makeSound() {
        System.out.println("Chi chi");
    }
}

public class Penguin extends Bird implements Soundable, Swimmable {
    @Override
    public void makeSound() {
        System.out.println("Honk honk");
    }
    
    @Override
    public void swim() {
        System.out.println(name + " is swimming");
    }
    // No fly method to cause errors
}
```

## Dependency Inversion Principle (DIP)

### Core Concept
High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

### Example: Bird Simulation

```java
// Bad approach - high level component depends on low level component
public class BirdSimulation {
    private Sparrow sparrow;
    
    public BirdSimulation() {
        this.sparrow = new Sparrow("Chirpy", "Brown");
    }
    
    public void start() {
        sparrow.fly();
        sparrow.makeSound();
    }
}

// Better approach using DIP
public class BirdSimulation {
    private Flyable flyingBird;
    private Soundable soundMakingBird;
    
    // Dependency injection
    public BirdSimulation(Flyable flyingBird, Soundable soundMakingBird) {
        this.flyingBird = flyingBird;
        this.soundMakingBird = soundMakingBird;
    }
    
    public void start() {
        flyingBird.fly();
        soundMakingBird.makeSound();
    }
}

// Usage
public static void main(String[] args) {
    Sparrow sparrow = new Sparrow("Chirpy", "Brown");
    BirdSimulation simulation = new BirdSimulation(sparrow, sparrow);
    simulation.start();
    
    // We can also use any bird that can fly and make sound
    Crow crow = new Crow("Blacky", "Black");
    simulation = new BirdSimulation(crow, crow);
    simulation.start();
}
```

## Design Evolution

The bird example evolved through several iterations to better adhere to SOLID principles:

### Initial Design (V0)
- Single `Bird` class with if-else logic in `makeSound()`
- Problems: Violates SRP, OCP

### Version 1 (V1)
- Refactored `makeSound()` into separate methods
- Still violates OCP as new birds require code modification

### Version 2 (V2)
- Abstract `Bird` class with concrete implementations
- Problem: LSP violation with non-flying birds like Penguin

### Version 2.1
- Penguin throws exception for `fly()`
- Problem: Violates LSP, surprises clients

### Version 2.3
- Bird hierarchy with `FlyingBird` and `NonFlyingBird` abstractions
- Better adherence to LSP

### Final Version
- Uses interfaces (`Flyable`, `Soundable`, `Swimmable`)
- Adheres to ISP and DIP
- Each bird only implements the behaviors it supports

This evolution demonstrates how applying SOLID principles leads to more maintainable, extensible, and robust code.


