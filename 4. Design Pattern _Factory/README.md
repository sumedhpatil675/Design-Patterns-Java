# Factory Design Pattern

## Table of Contents
- [Introduction](#introduction)
- [Factory Method Pattern](#factory-method-pattern)
- [Abstract Factory Pattern](#abstract-factory-pattern)
- [Practical Factory Pattern](#practical-factory-pattern)
- [Comparison](#comparison)
- [Class Explosion Problem](#class-explosion-problem)
- [When to Use](#when-to-use)
- [Code Examples](#code-examples)

## Introduction

Factory patterns are creational design patterns that deal with object creation for a class hierarchy. They help to decouple client code from concrete implementations of classes.

## Factory Method Pattern

Factory Method is a design pattern that defines an interface for creating objects but lets subclasses decide which classes to instantiate.

### Key Concept
- The Factory Method pattern allows a class to defer instantiation to subclasses.
- It helps create objects belonging to a class hierarchy.

### Example Structure

```java
// Abstract base class with factory method
abstract class A {
    // Abstract factory method
    abstract void work();
    
    void func() {
        System.out.println("pre-work");
        work(); // This will call the implementation in the subclass
        System.out.println("post-work");
    }
}

// Concrete implementations
class B extends A {
    void work() {
        System.out.println("work done in B");
    }
}

class C extends A {
    void work() {
        System.out.println("work done in C");
    }
}

// Client usage
public class Client {
    public static void main(String[] args) {
        // We can't instantiate A directly as it's abstract
        // A obj = new A(); // This would cause compile error
        
        // Using the factory method with implementation B
        A obj = new B();
        obj.func();
        
        // Using the factory method with implementation C
        A obj2 = new C();
        obj2.func();
    }
}
```

### Database Example

```java
// User service with factory method for creating queries
class UserService {
    void createUser(UserDTO user) {
        Database db = new Database("MySQL");
        db.execute("INSERT INTO...");
    }
    
    void deleteUser() {
        // Database operations...
    }
    
    void updateUser() {
        // Database operations...
    }
    
    void getUser(int userId) {
        // Database operations...
    }
}
```

## Database Factory Method Example

Version 1: Simple approach with conditionals

```java
class Database {
    private String type;
    
    public Database(String type) {
        this.type = type;
    }
    
    void execute(String queryText) {
        Query q = getQuery(queryText);
        q.execute();
    }
    
    Query getQuery(String qt) {
        if (type.equals("MySQL")) {
            return new MySQLQuery(qt);
        } else if (type.equals("MongoDB")) {
            return new MongoDBQuery(qt);
        }
        // Could add more database types here
        return null;
    }
}
```

Version 2: Improved with abstract factory method

```java
abstract class Database {
    void execute(String qt) {
        Query q = getQuery(qt);
        q.execute();
    }
    
    // Abstract factory method
    abstract Query getQuery(String qt);
}

class MySQLDB extends Database {
    Query getQuery(String qt) {
        return new MySQLQuery(qt);
    }
}

class MongoDB extends Database {
    Query getQuery(String qt) {
        return new MongoDBQuery(qt);
    }
}

class PostgresDB extends Database {
    Query getQuery(String qt) {
        return new PostgresQuery(qt);
    }
}
```

## Abstract Factory Pattern

The Abstract Factory pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes.

### Key Concept
- Creates families of related objects
- Client code works with interfaces, not implementation classes
- Promotes consistency among products

### Example Structure

```java
// Abstract factory interface
interface FlutterUIFactory {
    FlutterTextBox getFlutterTB();
    FlutterButton getFlutterB();
    FlutterDDL getFlutterDDL();
}

// Abstract product base class
abstract class FlutterPage {
    void setBgColor(Color c) { /* Implementation */ }
    void setRefreshRate() { /* Implementation */ }
    void setLayout() { /* Implementation */ }
    
    // Factory methods for UI components
    FlutterTextBox tb = getFlutterTB();
    FlutterButton b = getFlutterB();
    FlutterDDL ddl = getFlutterDDL();
}

// Concrete factories for different platforms
class AndroidUIFactory implements FlutterUIFactory {
    FlutterTextBox getFlutterTB() {
        return new AndroidTB();
    }
    
    FlutterButton getFlutterB() {
        return new AndroidB();
    }
    
    FlutterDDL getFlutterDDL() {
        return new AndroidDDL();
    }
}

class IOSUIFactory implements FlutterUIFactory {
    FlutterTextBox getFlutterTB() {
        return new IOSTB();
    }
    
    FlutterButton getFlutterB() {
        return new IOSB();
    }
    
    FlutterDDL getFlutterDDL() {
        return new IOSDDL();
    }
}

class WindowsUIFactory implements FlutterUIFactory {
    FlutterTextBox getFlutterTB() {
        return new WinTB();
    }
    
    FlutterButton getFlutterB() {
        return new WinB();
    }
    
    FlutterDDL getFlutterDDL() {
        return new WinDDL();
    }
}

// Client usage
class App {
    public static void main(String[] args) {
        FlutterPage fp;
        
        // Determine which factory to use based on platform
        if(platform.equals("Android")) {
            fp = new AndroidFP();
        }
        // More conditions for other platforms
        
        fp.setLayout();
    }
}
```

### Version 3: Abstract Factory with Database Example

```java
class Database {
    private QueryFactory qf;
    
    public void setQueryFactory(QueryFactory qf) {
        this.qf = qf;
    }
    
    void doWork() {
        Query q = qf.getQuery();
        q.execute();
    }
}

interface QueryFactory {
    Query getQuery();
}

class MySQLQueryFactory implements QueryFactory {
    Query getQuery() {
        return new MySQLQuery();
    }
}

class MongoDBQueryFactory implements QueryFactory {
    Query getQuery() {
        return new MongoDBQuery();
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        Database db = new Database();
        QueryFactory qf = new MySQLQueryFactory();
        db.setQueryFactory(qf);
        db.doWork();
    }
}
```

## Practical Factory Pattern

This pattern is not one of the 23 official GoF Design Patterns but is commonly found in codebases.

### Key Characteristics
- Violates OCP (Open-Closed Principle)
- Very prevalent and common in existing codebases
- Uses static factory methods

```java
class PracticalFactory {
    public static Query getQuery(String type) {
        if (type.equals("MySQL")) {
            return new MySQLQuery();
        } else if (type.equals("Mongo")) {
            return new MongoDBQuery();
        }
        // More conditions for other database types
        return null;
    }
}

// Usage in Database class
class Database {
    void doWork(String qt, String type) {
        Query q = PracticalFactory.getQuery(type);
        q.execute();
    }
}
```

## Comparison

### Factory Method vs. Abstract Factory

1. **Factory Method**
   - Template Method pattern
   - Creates a single object
   - Uses inheritance
   - Client knows about the Factory
   
2. **Abstract Factory**
   - Strategy pattern
   - Creates families of related objects
   - Uses composition
   - Client depends on interfaces

### Shortcoming of Factory Method covered by Abstract Factory

The Factory Method pattern can lead to a lot of subclassing when you need to create different object hierarchies, while Abstract Factory uses composition and provides more flexibility.

## Class Explosion Problem

When using inheritance for factories, adding a new product type can cause class explosion:

```
Database
├── MySQLDB
├── MongoDB
└── MSSQLDB

FlutterPage
├── AndroidFP
├── IOSFP
└── WindowsFP
```

If we need to add a new platform or database type, the number of classes grows exponentially.

## When to Use

- **Factory Method**: When a class can't anticipate the class of objects it must create, or when a class wants its subclasses to specify the objects it creates.

- **Abstract Factory**: When a system should be independent of how its products are created, composed, and represented, or when a system should be configured with one of multiple families of products.

- **Practical Factory**: For simpler cases where you need to create objects without adhering strictly to design principles.

## Code Examples

### Factory Method Example

```java
// Product interface
interface Transport {
    void deliver();
}

// Concrete products
class Truck implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering by land in a truck");
    }
}

class Ship implements Transport {
    @Override
    public void deliver() {
        System.out.println("Delivering by sea in a ship");
    }
}

// Creator abstract class with factory method
abstract class Logistics {
    public void planDelivery() {
        Transport t = createTransport();
        t.deliver();
    }
    
    // Factory method
    protected abstract Transport createTransport();
}

// Concrete creators
class RoadLogistics extends Logistics {
    @Override
    protected Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    @Override
    protected Transport createTransport() {
        return new Ship();
    }
}

// Client
public class Client {
    public static void main(String[] args) {
        Logistics logistics;
        
        // Determine which logistics to use
        String transportType = "sea";
        if (transportType.equals("road")) {
            logistics = new RoadLogistics();
        } else {
            logistics = new SeaLogistics();
        }
        
        logistics.planDelivery();
    }
}
```

### Dependency Injection Container (DIC)

The Factory Method pattern is commonly used in Dependency Injection Containers, which read configuration settings and create appropriate objects:

```java
// Example configuration (usually in XML or JSON)
// <interface name="Database" dependency="MySQLDB" />

// DIC implementation would read this configuration and create appropriate instances
```

## Homework

Implement both Template Method and Bridge Design Patterns based on the concept of the Factory Patterns.

---

## References

- Design Patterns: Elements of Reusable Object-Oriented Software by Gamma, Helm, Johnson, and Vlissides (Gang of Four)
- Factory Method Pattern: Creates objects through inheritance
- Abstract Factory Pattern: Creates objects through composition