# Singleton Design Pattern

## Introduction
The Singleton Design Pattern is one of the fundamental creational design patterns in software development. This pattern ensures that a class has only one instance and provides a global point of access to it.

## Table of Contents
1. [Introduction to Design Patterns](#introduction-to-design-patterns)
2. [Categories of Design Patterns](#categories-of-design-patterns)
3. [Singleton Design Pattern Overview](#singleton-design-pattern-overview)
4. [Problem Statement](#problem-statement)
5. [Implementation Versions](#implementation-versions)
6. [Thread Safety Considerations](#thread-safety-considerations)
7. [Complete Java Implementation](#complete-java-implementation)

## Introduction to Design Patterns

Design patterns are:
- A study of common problems with well-established solutions related to object-oriented design
- Repetitive solutions in software design
- Not vague - they are specific in how they guide classes/interfaces
- First documented by the "Gang of Four" (Gamma, Helm, Johnson, and Vlissides) in 1993
- A collection of 23 design patterns that represent elements of reusable object-oriented software

### Why Learn Design Patterns?
1. **Interviews** - Design patterns are commonly asked about in technical interviews
2. **Shared Vocabulary** - They provide a common language for developers to discuss solutions
   - Example: Saying "Dijkstra algorithm" immediately conveys a specific approach without needing to explain the entire algorithm

## Categories of Design Patterns

Design patterns are organized into three main categories:

### 1. Creational Design Patterns
These patterns deal with object creation mechanisms:
- How many objects to create
- How to create objects
- Examples:
  - **Singleton**
  - Builder
  - Factory Method
  - Abstract Factory
  - Prototype

### 2. Structural Design Patterns
These patterns concern the composition of classes and objects:
- Focus on class structure
- What attributes to include
- Examples:
  - Composite (shown in the notes with a tree-like structure)
  - Adapter
  - Bridge
  - Decorator
  - Facade

### 3. Behavioral Design Patterns
These patterns address how to add behaviors to classes:
- Focus on communication between objects
- Examples:
  - Observer (demonstrated with examples of cricket scores and financial data)
  - Iterator
  - Strategy
  - Command
  - Mediator

## Singleton Design Pattern Overview

### Problem Statement
How can we create a class for which only a single object can be created?

### Why is Singleton Required?
- For costly shared resources
- When multiple components need to access the same resource
- When object creation is expensive or resource-intensive

### Example Use Case: Database Connection
- A `DatabaseConnection` object is costly to create (takes time)
- The object can only be created after TCP connection is established between app server and DB server
- It's efficient to share this connection between modules (Users, Products, Employees)

## Implementation Versions

The notes show an evolution of the Singleton pattern implementation, addressing various issues:

### Version 0: Basic Class (Not Singleton)
```java
class DbConnection {
    // Fields and methods
    public DbConnection(url, username, pwd) {
        // Constructor
    }
}

class Client {
    main() {
        DbConnection c1 = new DbConnection(...);
        DbConnection c2 = new DbConnection(...);
    }
}
```
**Problem**: Multiple objects can be created

### Version 1: No Constructors
```java
class DbConnection {
    // No constructors defined
}
```
**Note**: Java will add the default constructor, so this doesn't solve the problem

### Version 2: Private Constructor
```java
class DbConnection {
    private DbConnection() {
        // Private constructor
    }
}

class Client {
    main() {
        // No object can be created
    }
}
```
**Problem**: Now no objects can be created at all

### Version 3: Static Method to Create Instance
```java
class DbConnection {
    private DbConnection() {}
    
    public static DbConnection getInstance() {
        DbConnection inst = new DbConnection();
        return inst;
    }
}

class Client {
    main() {
        DbConnection db = DbConnection.getInstance();
    }
}
```
**Problem**: This creates a new instance every time getInstance is called

### Version 4: Static Instance Variable
```java
class DbConnection {
    private DbConnection() {}
    
    public static DbConnection getInstance() {
        DbConnection inst = new DbConnection();
        return inst;
    }
}

class Client {
    main() {
        DbConnection db1 = DbConnection.getInstance();
        DbConnection db2 = DbConnection.getInstance();
        // db1 and db2 refer to different objects
    }
}
```
**Problem**: Still creates multiple objects

### Version 5: Static Instance Field
```java
class DbConnection {
    private DbConnection() {}
    private static DbConnection inst;
    
    public static DbConnection getInstance() {
        inst = new DbConnection();
        return inst;
    }
}
```
**Problem**: Still creates a new instance each time

### Version 6: First Reasonable Singleton (Single-Threaded)
```java
class DbConnection {
    private DbConnection() {}
    private static DbConnection inst;
    
    public static DbConnection getInstance() {
        if (inst == null) {
            inst = new DbConnection();
        }
        return inst;
    }
}
```
**Note**: This works for single-threaded applications but has race condition issues in multi-threaded environments

### Version 7: Thread-Safe Singleton
```java
class DbConnection {
    private DbConnection() {}
    private static DbConnection inst;
    
    public static synchronized DbConnection getInstance() {
        if (inst == null) {
            inst = new DbConnection();
        }
        return inst;
    }
}
```
**Improvement**: The `synchronized` keyword ensures thread safety but impacts performance

## Thread Safety Considerations

When multiple threads try to create a Singleton instance simultaneously:

1. Thread A checks if `inst == null` (true)
2. Thread B checks if `inst == null` (also true)
3. Thread A creates a new instance
4. Thread B creates another new instance
5. Race condition results in multiple instances being created

### Version 7: Synchronized Method
The first solution is to use synchronization:
- Make the `getInstance()` method synchronized
- This ensures only one thread can execute the method at a time
- Prevents the race condition but adds overhead

```java
public static synchronized DbConnection getInstance() {
    if (inst == null) {
        inst = new DbConnection();
    }
    return inst;
}
```

**Problem with Version 7**: Poor performance. While it works correctly as a singleton in multi-threaded environments, every thread must wait to enter the synchronized method even if the instance is already created. Only the first thread that creates the instance needs synchronization - subsequent threads that just read the instance are needlessly blocked.

### Version 8: Lock Block Approach (Problematic)
```java
public static DbConnection getInstance() {
    if (inst == null) {
        lock.lock();
        inst = new DbConnection();
        lock.unlock();
    }
    return inst;
}
```

**Problem with Version 8**: The lock only protects the creation of the instance, but multiple threads can still check `if (inst == null)` simultaneously before any of them acquire the lock, causing race conditions.

### Version 9: Double-Checked Locking
```java
public static DbConnection getInstance() {
    if (inst == null) {
        lock.lock();
        if (inst == null) {
            inst = new DbConnection();
        }
        lock.unlock();
    }
    return inst;
}
```

This version:
- Performs an initial check before acquiring the lock
- Only locks if the instance might need to be created
- Double-checks within the locked section to prevent race conditions
- Improves performance as most threads will skip the locking entirely
- Only threads that enter when instance is null will wait

Threads T1 and T2 are the first to enter and may need to create the instance.
Threads T3, T4, T5, T6, etc. will use the outside `if` check and get fast performance since they only read the instance value.

## Complete Java Implementation

Here's a complete implementation of a thread-safe Singleton pattern:

```java
public class DatabaseConnection {
    // Private constructor prevents instantiation from other classes
    private DatabaseConnection() {
        // Initialize connection parameters
        // Establish connection
    }
    
    // Static instance variable
    private static DatabaseConnection instance;
    
    // Public static method to get the singleton instance
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
    
    // Other methods for database operations
    public void executeQuery(String query) {
        // Execute SQL query
    }
    
    public void close() {
        // Close connection
    }
}
```

## Usage Example

```java
public class Client {
    public static void main(String[] args) {
        // Get the singleton instance
        DatabaseConnection db1 = DatabaseConnection.getInstance();
        
        // Another reference to the same instance
        DatabaseConnection db2 = DatabaseConnection.getInstance();
        
        // Both references point to the same object
        System.out.println(db1 == db2); // Prints: true
        
        // Use the database connection
        db1.executeQuery("SELECT * FROM users");
    }
}
```

## Advanced Implementations

### Double-Checked Locking (not shown in notes but relevant)
```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {}
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

### Version 10: Eager Loading
```java
public class DbConnection {
    // Create instance at class loading time
    private static final DbConnection inst = new DbConnection();
    
    private DbConnection() {}
    
    public static DbConnection getInstance() {
        return inst;
    }
}
```

This approach:
- Creates the singleton instance when the class is loaded
- Avoids all thread safety issues completely
- Works perfectly in multi-threaded environments
- Very fast performance as there are no checks or locks

**Problems with Version 10**:
- Java creates the instance immediately, parameters can't be passed to it
- Instance is created before it's actually needed (eager vs. lazy loading)

### Initialization-on-demand holder idiom
```java
public class DatabaseConnection {
    private DatabaseConnection() {}
    
    private static class InstanceHolder {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }
    
    public static DatabaseConnection getInstance() {
        return InstanceHolder.INSTANCE;
    }
}
```

## Summary

The Singleton pattern ensures that:
1. A class has only one instance
2. This instance is globally accessible
3. The instance is created only when needed
4. It's particularly useful for managing expensive resources
5. Thread-safety must be considered in multi-threaded environments