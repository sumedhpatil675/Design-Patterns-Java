# Design Patterns: Decorator and Observer

This README contains detailed notes on the Decorator and Observer design patterns, based on lecture materials.

## Table of Contents
- [Decorator Pattern](#decorator-pattern)
  - [Overview](#overview)
  - [Real-world Example](#real-world-example)
  - [Structure](#structure)
  - [Code Example: Ice Cream Shop](#code-example-ice-cream-shop)
  - [Real-life Java Implementation: Java I/O](#real-life-java-implementation-java-io)
- [Observer Pattern](#observer-pattern)
  - [Overview](#overview-1)
  - [Real-world Example](#real-world-example-1)
  - [Structure](#structure-1)
  - [Code Example: E-commerce Order System](#code-example-e-commerce-order-system)

## Decorator Pattern

### Overview

The Decorator pattern is a **structural design pattern** that allows behavior to be added to individual objects dynamically, without affecting the behavior of other objects from the same class.

**Key Concept**: Dynamically add responsibilities to objects at runtime.

### Real-world Example

**Ice Cream Shop**: A perfect analogy for the Decorator pattern.

1. Start with a base ice cream (cone with specific flavor)
2. Add toppings and extras dynamically:
   - Different cones (Waffle, Sugar, Mayo)
   - Different flavors (Vanilla, Strawberry)
   - Different syrups (Chocolate, Strawberry, Mango)
   - Different scoops (Strawberry, Pistachio, Mango)
   - Different toppings (Chocolate chips, Pineapple, Dry fruits, Berries)

### Structure

```
            ┌─────────────┐
            │ Component   │
            │ (Interface) │
            └─────────────┘
                   ▲
       ┌───────────┴───────────┐
       │                       │
┌──────┴──────┐       ┌───────┴────────┐
│ ConcreteComp│       │ Decorator      │
│ onent       │       │                │
└─────────────┘       └───────┬────────┘
                              ▲
                      ┌───────┴───────┐
                      │               │
               ┌──────┴─────┐  ┌──────┴─────┐
               │ DecoratorA │  │ DecoratorB │
               └────────────┘  └────────────┘
```

1. **Component**: Interface defining operations that can be altered by decorators
2. **ConcreteComponent**: Basic implementation of the Component interface
3. **Decorator**: Abstract class implementing the Component interface and holding a reference to a Component
4. **ConcreteDecorator**: Adds responsibilities to the component

### Code Example: Ice Cream Shop

Here's how the Decorator pattern can be implemented for an ice cream shop:

**1. The Component Interface (Ingredient)**

```java
// The base interface - all ingredients must implement this
interface Ingredient {
    int getCost();
    String getDesc();
}
```

**2. The Base Component (IceCream)**

```java
// Basic implementation - base for all ice creams
class IceCream implements Ingredient {
    private List<Ingredient> contents = new ArrayList<>();
    
    public void addIngredient(Ingredient ing) {
        contents.add(ing);
    }
    
    @Override
    public int getCost() {
        int c = 0;
        for (Ingredient i : contents) {
            c += i.getCost();
        }
        return c;
    }
    
    @Override
    public int getDesc() {
        // Implementation to describe the ice cream with all ingredients
        // Similar to getCost() but returns a combined description
    }
}
```

**3. Problem with Version 1**

In the first version, all ingredients were treated equally, but in reality, they are different:
- A cone is not the same as a cherry
- Some ingredients are base/necessary (like cone)
- Others are add-ons/decorations (like toppings)

**4. Version 2: Using Decorator Pattern**

```java
// Base Component
interface IceCream {
    int getCost();
    String getDesc();
}

// Concrete Components (Base Ice Cream types)
class MayoCone implements IceCream {
    @Override
    public int getCost() {
        return 10; // base cost
    }
    
    @Override
    public String getDesc() {
        return "Mayo Cone";
    }
}

class VanillaCone implements IceCream {
    @Override
    public int getCost() {
        return 8;
    }
    
    @Override
    public String getDesc() {
        return "Vanilla Cone";
    }
}

// Decorator
abstract class IceCreamAddOn implements IceCream {
    protected IceCream icecream; // reference to a Component
    
    public IceCreamAddOn(IceCream ic) {
        this.icecream = ic;
    }
    
    @Override
    public int getCost() {
        // Delegate to the wrapped object
        return icecream.getCost();
    }
    
    @Override
    public String getDesc() {
        return icecream.getDesc();
    }
}

// Concrete Decorators
class VanillaScoopAddOn extends IceCreamAddOn {
    public VanillaScoopAddOn(IceCream ic) {
        super(ic);
    }
    
    @Override
    public int getCost() {
        // Add cost of vanilla scoop to the base
        return 7 + super.getCost();
    }
    
    @Override
    public String getDesc() {
        return super.getDesc() + ", Vanilla Scoop";
    }
}

class CherryAddOn extends IceCreamAddOn {
    public CherryAddOn(IceCream ic) {
        super(ic);
    }
    
    @Override
    public int getCost() {
        return 5 + super.getCost();
    }
    
    @Override
    public String getDesc() {
        return super.getDesc() + ", Cherry";
    }
}
```

**5. Client Usage**

```java
public class Client {
    public static void main(String[] args) {
        // Create a Mayo Cone with Vanilla Scoop and Cherry decorations
        IceCream ic = new MayoCone();
        IceCream icWithVS = new VanillaScoopAddOn(ic);
        IceCream icWithVSAndCherry = new CherryAddOn(icWithVS);
        
        // Calculate final cost
        int cost = icWithVSAndCherry.getCost(); // 10 + 7 + 5 = 22
        
        // Get description
        String desc = icWithVSAndCherry.getDesc(); // "Mayo Cone, Vanilla Scoop, Cherry"
        
        System.out.println("Cost: " + cost);
        System.out.println("Description: " + desc);
    }
}
```

This design creates a layered structure:
- Each decorator wraps a component
- Each decorator adds its own behavior (cost and description)
- Decorators can be stacked to create complex combinations

The recursive structure of calling `super.getCost()` and `super.getDesc()` creates a chain that:
1. Starts at the innermost concrete component (MayoCone)
2. Adds behavior from each wrapper (VanillaScoopAddOn)
3. Adds behavior from the outermost wrapper (CherryAddOn)

### Real-life Java Implementation: Java I/O

The Java I/O library is a classic example of the Decorator pattern:

```
            ┌───────┐
            │InputStream │
            └───────┘
                 ▲
     ┌───────────┴───────────┐
     │                       │
┌────┴─────┐          ┌──────┴──────┐
│FileStream│          │FilterStream │
└──────────┘          └──────┬──────┘
                             ▲
                   ┌─────────┴──────────┐
                   │                    │
         ┌─────────┴────────┐    ┌──────┴────────┐
         │BufferedInputStream│    │EncryptionStream│
         └──────────────────┘    └────────────────┘
```

Example usage:

```java
// Base component
InputStream fileStream = new FileInputStream("abc.txt");

// Decorate with encryption
InputStream encryptedStream = new EncryptionInputStream(fileStream);

// Further decorate with compression
InputStream compressedEncryptedStream = new CompressionInputStream(encryptedStream);

// Usage
compressedEncryptedStream.read(); // The read() call goes through all decorators
```

The call stack for a read operation would be:
1. `compressedEncryptedStream.read()`
2. Decompresses data and calls `encryptedStream.read()`
3. Decrypts data and calls `fileStream.read()`
4. Reads directly from the file

## Observer Pattern

### Overview

The Observer pattern is a **behavioral design pattern** that defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Key Problem Solved**: How can a component that has changed its state notify other components without knowing who they are?

### Real-world Example

**E-commerce Order Processing**:

Imagine an Amazon-like system where:
1. When a new order is created, multiple services need to be notified:
   - Inventory Service (IS) - to update stock
   - Email Service (ES) - to send confirmation email
   - SMS Service (SMS) - to send text notifications
   - App Notification Service (ANS) - to send mobile app alerts
   - Logistics Service (LS) - to prepare shipping

**Problem with Traditional Approach**:
- Whenever a new service needs to be notified, the Amazon Service/Order Processing code must be changed
- Requires recompiling and redeploying the main service

### Structure

```
┌─────────────┐       ┌───────────┐
│  Subject    │<>─────│ Observer  │
├─────────────┤       ├───────────┤
│ attach()    │       │ update()  │
│ detach()    │       └─────┬─────┘
│ notify()    │             ▲
└──────┬──────┘       ┌─────┴─────┐
       ▲              │ConcreteObs│
       │              └───────────┘
┌──────┴──────┐
│ConcreteSubj │
├─────────────┤
│ getState()  │
│ setState()  │
└─────────────┘
```

1. **Subject**: Interface defining operations for attaching and detaching observers
2. **ConcreteSubject**: Implements the Subject interface and maintains state
3. **Observer**: Interface defining the update method that gets called when Subject changes
4. **ConcreteObserver**: Implements the Observer interface to respond to Subject changes

### Code Example: E-commerce Order System

**1. The Observer Interface**

```java
// The Observer interface
interface OrderObserver {
    void newOrderCreated(OrderDetails od);
}
```

**2. Concrete Observers (Services)**

```java
// Email Service
class EmailService implements OrderObserver {
    @Override
    public void newOrderCreated(OrderDetails od) {
        // Logic for sending emails
        System.out.println("Email sent for order: " + od.getId());
    }
}

// SMS Service
class SMSService implements OrderObserver {
    @Override
    public void newOrderCreated(OrderDetails od) {
        // Logic for sending SMS
        System.out.println("SMS sent for order: " + od.getId());
    }
}

// App Notification Service
class AppNotificationService implements OrderObserver {
    @Override
    public void newOrderCreated(OrderDetails od) {
        // Logic for sending app notifications
        System.out.println("App notification sent for order: " + od.getId());
    }
}

// Inventory Service
class InventoryService implements OrderObserver {
    @Override
    public void newOrderCreated(OrderDetails od) {
        // Logic for updating inventory
        System.out.println("Inventory updated for order: " + od.getId());
    }
}

// Seller Service
class SellerService implements OrderObserver {
    @Override
    public void newOrderCreated(OrderDetails od) {
        // Logic for notifying sellers
        System.out.println("Seller notified for order: " + od.getId());
    }
}
```

**3. The Subject (Order Processing)**

```java
// The Subject (Amazon Service)
class AmazonService {
    private List<OrderObserver> observers = new ArrayList<>();
    
    // Register an observer
    public void register(OrderObserver observer) {
        observers.add(observer);
    }
    
    // Unregister an observer
    public void unregister(OrderObserver observer) {
        observers.remove(observer);
    }
    
    // Create a new order - notify all observers
    public void createOrder() {
        // Create the order in the database
        OrderDetails od = new OrderDetails();
        // Business logic...
        
        // Notify all registered observers
        for (OrderObserver ob : observers) {
            ob.newOrderCreated(od);
        }
    }
}
```

**4. Order Details Class**

```java
// Order details 
class OrderDetails {
    private String orderId;
    private String address;
    private List<Product> products;
    private double amount;
    
    // getters and setters
    public String getId() {
        return orderId;
    }
}
```

**5. Client Usage**

```java
public class Client {
    public static void main(String[] args) {
        // Create the Amazon service
        AmazonService amazonService = new AmazonService();
        
        // Create service observers
        OrderObserver emailService = new EmailService();
        OrderObserver smsService = new SMSService();
        OrderObserver appService = new AppNotificationService();
        OrderObserver inventoryService = new InventoryService();
        OrderObserver sellerService = new SellerService();
        
        // Register observers with the subject
        amazonService.register(emailService);
        amazonService.register(smsService);
        amazonService.register(appService);
        amazonService.register(inventoryService);
        amazonService.register(sellerService);
        
        // Create a new order - all observers will be notified
        amazonService.createOrder();
        
        // Later, if we want to stop SMS notifications
        amazonService.unregister(smsService);
        
        // Create another order - SMS won't be notified
        amazonService.createOrder();
    }
}
```

**Benefits of this approach**:
1. New services can be added without changing the AmazonService code
2. Services can be registered and unregistered dynamically at runtime
3. The Subject (AmazonService) doesn't need to know about the concrete implementations of its observers

## Conclusion

Both the Decorator and Observer patterns solve specific design problems:

- **Decorator** allows for dynamic extension of object behavior at runtime through composition rather than inheritance.
- **Observer** enables loose coupling between objects by establishing a one-to-many dependency where changes in one object trigger updates in multiple dependent objects.

These patterns are fundamental to creating flexible, maintainable, and extensible applications.