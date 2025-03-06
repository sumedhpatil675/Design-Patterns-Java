# Strategy Design Pattern

## Introduction
The Strategy Design Pattern is a behavioral design pattern that enables selecting an algorithm's behavior at runtime. It defines a family of algorithms, encapsulates each one, and makes them interchangeable within that family.

## Problem Statement
Looking at the evolution of a Google Maps-like navigation system described in the lecture notes:

### Version 1 Issues
The initial implementation had several problems:
- **Single Responsibility Principle (SRP) Violation**: One class handled multiple transportation modes
- **Open-Closed Principle (OCP) Violation**: Code needed modification when adding new transportation modes

```java
// Version 1 - Problematic Implementation
class GoogleMaps {
    findPath(from, to, mode) {
        if (mode == "Car") {
            // Path via Car
        } else if (mode == "Bike") {
            // Path via Bike
        } else if (mode == "Walk") {
            // Path on foot
        }
    }
}
```

## Solution: Strategy Pattern

### Key Components

1. **Strategy Interface**: Defines a common interface for all supported algorithms
2. **Concrete Strategies**: Implementations of the Strategy interface
3. **Context**: Maintains a reference to a Strategy object and delegates the algorithm execution to it

### Version 2 - Partial Improvement

This version separated responsibilities into different classes but still had issues:

```java
// Different PathFinders for different transportation modes
class CarPathFinder {
    findPath(from, to) {
        // Car-specific path finding
    }
}

class BikePathFinder {
    findPath(from, to) {
        // Bike-specific path finding
    }
}

class WalkPathFinder {
    findPath(from, to) {
        // Walk-specific path finding
    }
}

// GoogleMaps class that still has OCP violation
class GoogleMaps {
    findPath(from, to, mode) {
        if (mode == "Car") {
            CarPathFinder cpf = new CarPathFinder();
            return cpf.findPath(from, to);
        } else if (mode == "Bike") {
            BikePathFinder bpf = new BikePathFinder();
            return bpf.findPath(from, to);
        } else if (mode == "Walk") {
            WalkPathFinder wpf = new WalkPathFinder();
            return wpf.findPath(from, to);
        }
    }
}
```

Issues with Version 2:
1. SRP violation is solved: separate classes for each transport mode
2. OCP violation still exists: need to modify GoogleMaps when adding new transport modes
3. Dependency Inversion Principle (DIP) violation: tight coupling between GoogleMaps and concrete PathFinders

### Version 3 - Strategy Pattern Implementation

This version introduces the Strategy Pattern with a common interface:

```java
// Strategy Interface
interface PathFinder {
    void findPath(from, to);
}

// Concrete Strategies
class CarPathFinder implements PathFinder {
    @Override
    public void findPath(from, to) {
        // Car-specific path finding
    }
}

class BikePathFinder implements PathFinder {
    @Override
    public void findPath(from, to) {
        // Bike-specific path finding
    }
}

class WalkPathFinder implements PathFinder {
    @Override
    public void findPath(from, to) {
        // Walk-specific path finding
    }
}
```

### Dependency Injection Approaches

The notes highlight four ways to implement dependency injection:

#### 1. Constructor Injection

```java
class GoogleMaps {
    private PathFinder pf = null;
    
    public GoogleMaps(PathFinder inst) {
        this.pf = inst;
    }
    
    void findPath(from, to) {
        if (pf == null) {
            System.out.println("No algorithm");
            return;
        }
        pf.findPath(from, to);
    }
}
```

#### 2. Setter Injection

```java
class GoogleMaps {
    private PathFinder pf = null;
    
    public void setPathFinder(PathFinder inst) {
        this.pf = inst;
    }
    
    void findPath(from, to) {
        if (pf == null) {
            System.out.println("No algorithm");
            return;
        }
        pf.findPath(from, to);
    }
}
```

#### 3. Factory Method

```java
class GoogleMaps {
    void findPath(from, to, mode) {
        PathFinder pf = PracticalFactory.getPF(mode);
        pf.findPath(from, to);
    }
}

class PracticalFactory {
    static PathFinder getPF(String mode) {
        if (mode == "Car") {
            return new CarPathFinder();
        } else if (mode == "Bike") {
            return new BikePathFinder();
        } else if (mode == "Walk") {
            return new WalkPathFinder();
        }
        return null;
    }
}
```

#### 4. Abstract Factory

An advanced pattern that provides an interface for creating families of related objects.

### Client Usage Examples

#### Using Constructor Injection
```java
void main() {
    GoogleMaps gm = new GoogleMaps(null);
    gm.findPath("Delhi", "Bengaluru"); // Will show "No algorithm"
    
    gm.setPathFinder(new BikePathFinder());
    gm.findPath("Delhi", "Bengaluru"); // Using bike path
    
    gm.setPathFinder(new WalkPathFinder());
    gm.findPath("Delhi", "Bengaluru"); // Using walk path
}
```

#### Using Factory Method
```java
void main() {
    GoogleMaps gm = new GoogleMaps();
    
    // Change mode and find path
    changeMode("Car");
    gm.findPath("Delhi", "Bengaluru", "Car");
    
    changeMode("Bike");
    gm.findPath("Delhi", "Bengaluru", "Bike");
}

void changeMode(String mode) {
    PathFinder pf = PracticalFactory.getPF(mode);
    // Additional mode-change logic
}
```

## Advantages of Strategy Pattern

1. **Loose Coupling**: Context class doesn't need to know the specific algorithm implementations
2. **Open-Closed Principle**: Easy to add new strategies without changing existing code
3. **Eliminates Conditional Statements**: Replaces complex if-else structures with polymorphism
4. **Runtime Flexibility**: Algorithms can be swapped during program execution

## Diagram

```
┌───────────────┐
│ PathFinder    │
│  (Interface)  │
│───────────────│
│ findPath(f,t) │
└───────┬───────┘
        │
        ├─────────────┬──────────────┐
        │             │              │
┌───────▼─────┐ ┌─────▼───────┐ ┌────▼────────┐
│CarPathFinder│ │BikePathFinder│ │WalkPathFinder│
│─────────────│ │─────────────│ │──────────────│
│findPath(f,t)│ │findPath(f,t)│ │findPath(f,t) │
└─────────────┘ └─────────────┘ └──────────────┘
        ▲             ▲              ▲
        │             │              │
        └─────────────┼──────────────┘
                      │
               ┌──────▼──────┐
               │ GoogleMaps  │
               │─────────────│
               │private PF pf│
               │findPath(f,t)│
               └─────────────┘
```

## Real-World Applications

1. **Payment Processing**: Different payment methods (Credit Card, PayPal, Crypto)
2. **Sorting Algorithms**: Choosing different sorting strategies based on data characteristics
3. **Compression Algorithms**: Selecting compression methods based on file types
4. **UI Rendering**: Different display strategies for various devices
5. **Authentication Methods**: Supporting multiple login mechanisms

## Summary

The Strategy Pattern is essential when you have:
- Multiple algorithms that differ only in behavior
- Need to avoid conditional statements to select behaviors
- Want to hide complex algorithm implementation details
- Need to swap algorithms at runtime

By using interfaces and dependency injection techniques, the Strategy Pattern achieves flexible, maintainable, and extensible code.