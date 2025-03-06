# Prototype Design Pattern

## Introduction

The Prototype Design Pattern is a creational design pattern that lets you copy existing objects without making your code dependent on their classes. It allows objects to create copies of themselves, which is particularly useful when the cost of creating a new object is more expensive than copying an existing one.

## Problem Statement

Given an object of a class, create a copy of that object while avoiding the following issues:

1. **Tight Coupling**: Client code should not be tightly coupled with the object class structure.
2. **Private Attributes**: Private attributes may not be accessible directly from client code.
3. **Child Class Objects**: Objects may be instances of child classes passed as parent class types, which can lead to incorrect copying.

## Rudimentary Solution (Not Recommended)

A naive approach might look like this:

```java
// Original object
Student st = new Student();

// Creating a copy manually
Student stCopy = new Student();
stCopy.name = st.name;
stCopy.age = st.age;
stCopy.email = st.email;
```

### Issues with this approach:

1. **Tight Coupling**: The client and object classes are tightly coupled
2. **Private Attributes**: Private attributes cannot be accessed from the client
3. **Child Class Objects**: We might end up creating a copy of only the parent class attributes if the object is actually a child class instance

## Copy Constructor Approach

Using a copy constructor is an improvement:

```java
Student copy = new Student(student);
```

However, this still doesn't solve the child class object problem completely. Consider:

```java
// Type checking logic becomes necessary
if (obj instanceof Config) {
    // create Config copy
} else if (obj instanceof SplConfig) {
    // create SplConfig copy
}
```

This approach can break easily and still requires knowledge of the class hierarchy.

## The Prototype Pattern Solution

The ideal solution is to let the object handle its own copying logic:

```java
// Client calls
Student copy = st.copy();
```

### Structure

1. The **Prototype** interface declares the cloning method.
2. **Concrete Prototype** classes implement the cloning method.
3. The **Client** can produce a copy of any object that follows the prototype interface.

## Implementation in Java

### 1. Prototype Interface

```java
public interface Prototype {
    Prototype copy();
}
```

### 2. Config Class Implementation

```java
public class Config implements Prototype {
    private String username;
    private String password;
    private String url;
    private int port;
    private String sessionName;
    private String billingAccount;
    
    // Constructors, getters, setters
    
    @Override
    public Config copy() {
        Config copy = new Config();
        copy.username = this.username;
        copy.password = this.password;
        copy.url = this.url;
        copy.port = this.port;
        copy.sessionName = this.sessionName;
        copy.billingAccount = this.billingAccount;
        return copy;
    }
}
```

### 3. SplConfig (Child Class) Implementation

```java
public class SplConfig extends Config {
    private String secretToken;
    
    // Constructors, getters, setters
    
    @Override
    public SplConfig copy() {
        SplConfig copy = new SplConfig();
        // Copy parent class attributes
        copy.setUsername(this.getUsername());
        copy.setPassword(this.getPassword());
        copy.setUrl(this.getUrl());
        copy.setPort(this.getPort());
        copy.setSessionName(this.getSessionName());
        copy.setBillingAccount(this.getBillingAccount());
        
        // Copy child class specific attributes
        copy.secretToken = this.secretToken;
        return copy;
    }
}
```

### 4. Client Usage

```java
public class Client {
    public static void main(String[] args) {
        // Using parent class
        Config config = new Config();
        // Set properties
        
        Config configCopy = config.copy();
        
        // Using child class
        SplConfig splConfig = new SplConfig();
        // Set properties
        
        SplConfig splConfigCopy = splConfig.copy();
        
        // Polymorphic usage (note the return is still the correct type)
        Config someConfig = getConfigFromSomewhere();
        Config copiedConfig = someConfig.copy(); // Will be the correct type (Config or SplConfig)
    }
}
```

## Registry Approach

A registry can be added to store and retrieve prototype instances:

```java
public class PrototypeRegistry {
    private Map<String, Prototype> items = new HashMap<>();
    
    public void addItem(String key, Prototype prototype) {
        items.put(key, prototype);
    }
    
    public Prototype getItem(String key) {
        return items.get(key).copy();
    }
}
```

Usage:

```java
PrototypeRegistry registry = new PrototypeRegistry();

Config config = new Config();
// Set up config
registry.addItem("basic", config);

SplConfig splConfig = new SplConfig();
// Set up special config
registry.addItem("special", splConfig);

// Later, get a new instance
Config newConfig = (Config) registry.getItem("basic");
SplConfig newSplConfig = (SplConfig) registry.getItem("special");
```

## Student Example

Here's a complete example using the Student class:

```java
public interface Prototype {
    Prototype copy();
}

public class Student implements Prototype {
    private String name;
    private int age;
    private String email;
    
    // Constructors, getters, setters
    
    @Override
    public Student copy() {
        Student copy = new Student();
        copy.name = this.name;
        copy.age = this.age;
        copy.email = this.email;
        return copy;
    }
}

public class GraduateStudent extends Student {
    private String researchTopic;
    
    // Constructors, getters, setters
    
    @Override
    public GraduateStudent copy() {
        GraduateStudent copy = new GraduateStudent();
        // Copy base attributes
        copy.setName(this.getName());
        copy.setAge(this.getAge());
        copy.setEmail(this.getEmail());
        
        // Copy specific attributes
        copy.researchTopic = this.researchTopic;
        return copy;
    }
}
```

## Benefits of Prototype Pattern

1. **No Tight Coupling**: The client doesn't need to know the concrete classes of objects it's copying
2. **Complete Copying**: All attributes, including private ones, can be properly copied
3. **Correct Type Preservation**: When copying child class objects, all their specific attributes are preserved
4. **Simplified Client Code**: The client just calls a simple `copy()` method without worrying about the details

## When to Use 

1. When your code shouldn't depend on the concrete classes of objects you need to copy
2. When you want to reduce the number of subclasses that only differ in initialization
3. When you need to create copies of objects that have complex configuration settings
4. When objects have lots of private fields that need to be included in the copy

## Class Diagram

```
+----------------+       +------------------------+
|   Prototype    |<------|        Client         |
+----------------+       +------------------------+
| + copy()       |
+----------------+
      ∧
      |
+------------------+----------------------+
|                  |                      |
+----------------+ +--------------------+ 
|    Config      | |      SplConfig     | 
+----------------+ +--------------------+ 
| - username     | | - secretToken      | 
| - password     | +--------------------+ 
| - url          | | + copy()           | 
| - port         | +--------------------+ 
| - sessionName  |
| - billingAccount|
+----------------+
| + copy()       |
+----------------+
```

## Summary

The Prototype Design Pattern allows us to:

1. Create copies of objects without coupling to their specific classes
2. Reduce initialization code by creating prototypes
3. Properly copy complex objects including all attributes 
4. Preserve the exact type of objects being copied