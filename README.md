# Design-Patterns-Java


**When to Use**:
- When you want to establish a subscription mechanism for notifying multiple objects about events
- When an object should be able to notify other objects without making assumptions about who these objects are
- When changes to one object require changing others, and you don't know how many objects need to be changed

### State Pattern

**Purpose**: Allow an object to alter its behavior when its internal state changes. The object will appear to change its class.

**Key Components**:
- **Context**: Maintains an instance of a ConcreteState subclass that defines the current state
- **State**: Interface defining the behavior associated with a state of the Context
- **Concrete State**: Subclasses implementing behavior associated with a state of the Context

**Implementation**:

```java
// State
interface State {
    void insertCoin();
    void ejectCoin();
    void turnCrank();
    void dispense();
}

// Concrete States
class NoCoinState implements State {
    GumballMachine gumballMachine;
    
    public NoCoinState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("You inserted a coin");
        gumballMachine.setState(gumballMachine.getHasCoinState());
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("You haven't inserted a coin");
    }
    
    @Override
    public void turnCrank() {
        System.out.println("You turned, but there's no coin");
    }
    
    @Override
    public void dispense() {
        System.out.println("You need to pay first");
    }
}

class HasCoinState implements State {
    GumballMachine gumballMachine;
    
    public HasCoinState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("You can't insert another coin");
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("Coin returned");
        gumballMachine.setState(gumballMachine.getNoCoinState());
    }
    
    @Override
    public void turnCrank() {
        System.out.println("You turned the crank");
        gumballMachine.setState(gumballMachine.getSoldState());
    }
    
    @Override
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}

// Context
class GumballMachine {
    private State noCoinState;
    private State hasCoinState;
    private State soldState;
    private State soldOutState;
    
    private State state;
    private int count = 0;
    
    public GumballMachine(int numberGumballs) {
        noCoinState = new NoCoinState(this);
        hasCoinState = new HasCoinState(this);
        soldState = new SoldState(this);
        soldOutState = new SoldOutState(this);
        
        this.count = numberGumballs;
        if (numberGumballs > 0) {
            state = noCoinState;
        } else {
            state = soldOutState;
        }
    }
    
    public void insertCoin() {
        state.insertCoin();
    }
    
    public void ejectCoin() {
        state.ejectCoin();
    }
    
    public void turnCrank() {
        state.turnCrank();
        state.dispense();
    }
    
    void setState(State state) {
        this.state = state;
    }
    
    void releaseBall() {
        System.out.println("A gumball comes rolling out");
        if (count > 0) {
            count--;
        }
    }
    
    // Getters for all states and count
    public State getNoCoinState() { return noCoinState; }
    public State getHasCoinState() { return hasCoinState; }
    public State getSoldState() { return soldState; }
    public State getSoldOutState() { return soldOutState; }
    public int getCount() { return count; }
}
```

**When to Use**:
- When an object's behavior depends on its state, and it must change behavior at runtime
- When operations have large, multipart conditional statements that depend on the object's state
- When state transitions are explicit and atomic

### Strategy Pattern

**Purpose**: Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Key Components**:
- **Strategy**: Interface common to all supported algorithms
- **Concrete Strategy**: Implements the algorithm using the Strategy interface
- **Context**: Maintains a reference to a Strategy object and can be configured with different strategies

**Implementation**:

```java
// Strategy
interface PaymentStrategy {
    void pay(int amount);
}

// Concrete Strategies
class CreditCardStrategy implements PaymentStrategy {
    private String name;
    private String cardNumber;
    private String cvv;
    private String dateOfExpiry;
    
    public CreditCardStrategy(String name, String cardNumber, String cvv, String dateOfExpiry) {
        this.name = name;
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.dateOfExpiry = dateOfExpiry;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println(amount + " paid with credit card");
    }
}

class PayPalStrategy implements PaymentStrategy {
    private String email;
    private String password;
    
    public PayPalStrategy(String email, String password) {
        this.email = email;
        this.password = password;
    }
    
    @Override
    public void pay(int amount) {
        System.out.println(amount + " paid using PayPal");
    }
}

// Context
class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public int calculateTotal() {
        int sum = 0;
        for (Item item : items) {
            sum += item.getPrice();
        }
        return sum;
    }
    
    public void pay(PaymentStrategy paymentMethod) {
        int amount = calculateTotal();
        paymentMethod.pay(amount);
    }
}
```

**When to Use**:
- When you want to define a family of algorithms
- When you need to avoid exposing complex algorithm-specific data structures
- When different variants of an algorithm are needed
- When a class defines many behaviors, and these appear as multiple conditional statements

### Template Method Pattern

**Purpose**: Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

**Key Components**:
- **Abstract Class**: Defines abstract operations that subclasses implement to execute specific steps of the algorithm
- **Concrete Class**: Implements the operations to carry out the primitive steps of the algorithm

**Implementation**:

```java
// Abstract Class
abstract class Game {
    // Template method
    public final void play() {
        initialize();
        startPlay();
        endPlay();
    }
    
    // These methods will be implemented by subclasses
    protected abstract void initialize();
    protected abstract void startPlay();
    protected abstract void endPlay();
}

// Concrete Classes
class Cricket extends Game {
    @Override
    protected void initialize() {
        System.out.println("Cricket Game Initialized");
    }
    
    @Override
    protected void startPlay() {
        System.out.println("Cricket Game Started");
    }
    
    @Override
    protected void endPlay() {
        System.out.println("Cricket Game Finished");
    }
}

class Football extends Game {
    @Override
    protected void initialize() {
        System.out.println("Football Game Initialized");
    }
    
    @Override
    protected void startPlay() {
        System.out.println("Football Game Started");
    }
    
    @Override
    protected void endPlay() {
        System.out.println("Football Game Finished");
    }
}
```

**When to Use**:
- When you want to implement the invariant parts of an algorithm once and leave the variable parts to subclasses
- When common behavior among subclasses should be factored and localized in a common class
- To control subclasses' extensions by letting them override only specific parts of an algorithm

### Visitor Pattern

**Purpose**: Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates.

**Key Components**:
- **Visitor**: Interface declaring visit operations for all concrete element types
- **Concrete Visitor**: Implements the Visitor interface
- **Element**: Interface declaring an accept operation that takes a visitor as an argument
- **Concrete Element**: Implements the Element interface and defines the accept operation

**Implementation**:

```java
// Element
interface Shape {
    void accept(Visitor visitor);
}

// Concrete Elements
class Circle implements Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    public double getRadius() {
        return radius;
    }
    
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

class Rectangle implements Shape {
    private double width;
    private double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    public double getWidth() {
        return width;
    }
    
    public double getHeight() {
        return height;
    }
    
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}

// Visitor
interface Visitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
}

// Concrete Visitor
class AreaCalculator implements Visitor {
    private double totalArea = 0;
    
    @Override
    public void visit(Circle circle) {
        totalArea += Math.PI * circle.getRadius() * circle.getRadius();
    }
    
    @Override
    public void visit(Rectangle rectangle) {
        totalArea += rectangle.getWidth() * rectangle.getHeight();
    }
    
    public double getTotalArea() {
        return totalArea;
    }
}
```

**When to Use**:
- When an object structure contains many classes of objects with differing interfaces, and you want to perform operations on these objects that depend on their concrete classes
- When many distinct and unrelated operations need to be performed on objects in an object structure
- When classes defining the object structure rarely change, but you often want to define new operations over the structure

## Summary of Design Patterns

### Creational Patterns
- **Prototype**: Create new objects by copying existing objects
- **Factory Method**: Delegates object creation to subclasses
- **Abstract Factory**: Creates families of related objects
- **Builder**: Separates object construction from its representation
- **Singleton**: Ensures a class has only one instance

### Structural Patterns
- **Adapter**: Makes incompatible interfaces compatible
- **Bridge**: Separates an abstraction from its implementation
- **Composite**: Composes objects into tree structures
- **Decorator**: Adds responsibilities to objects dynamically
- **Facade**: Provides a simplified interface to a complex subsystem
- **Flyweight**: Shares common state between multiple objects
- **Proxy**: Controls access to an object

### Behavioral Patterns
- **Chain of Responsibility**: Passes a request along a chain of handlers
- **Command**: Encapsulates a request as an object
- **Iterator**: Accesses elements of a collection sequentially
- **Mediator**: Defines simplified communication between objects
- **Memento**: Captures and restores an object's internal state
- **Observer**: Notifies dependents when an object changes
- **State**: Alters an object's behavior when its state changes
- **Strategy**: Encapsulates interchangeable algorithms
- **Template Method**: Defines the skeleton of an algorithm
- **Visitor**: Separates algorithms from the objects on which they operate

## Choosing the Right Pattern

1. **Identify the problem**: Analyze what needs to be flexible or extensible
2. **Consider intent**: Match the problem to patterns based on their intent
3. **Review context**: Consider program context and constraints
4. **Evaluate trade-offs**: Assess complexity, performance, and flexibility
5. **Start simple**: Choose the simplest pattern that solves your problem 
6. **Mix and match**: Patterns can be used together in a single design# Design Patterns in Java - Complete Overview

Design patterns are typical solutions to common problems in software design. They represent best practices evolved over time by experienced software developers. This document provides a concise overview of the major design patterns, organized by category, with Java implementation notes.

## Table of Contents

- [Creational Patterns](#creational-patterns)
- [Structural Patterns](#structural-patterns)
- [Behavioral Patterns](#behavioral-patterns)

## Creational Patterns

Creational patterns provide various object creation mechanisms that increase flexibility and reuse of existing code.

### Prototype Pattern

**Purpose**: Create new objects by copying existing ones, without making the code dependent on their classes.

**Key Components**:
- **Prototype Interface**: Declares the cloning method
- **Concrete Prototype**: Implements the cloning method
- **Client**: Creates new objects by copying existing ones

**Implementation**:

```java
public interface Prototype {
    Prototype copy();
}

public class Config implements Prototype {
    private String username;
    private String password;
    private String url;
    private int port;
    
    @Override
    public Config copy() {
        Config copy = new Config();
        copy.username = this.username;
        copy.password = this.password;
        copy.url = this.url;
        copy.port = this.port;
        return copy;
    }
}
```

**When to Use**:
- When object creation is expensive
- When your code shouldn't depend on concrete classes
- When dealing with composite objects or complex configurations

### Factory Method Pattern

**Purpose**: Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Key Components**:
- **Creator**: Declares the factory method
- **Concrete Creator**: Implements the factory method
- **Product**: The interface for objects created by the factory
- **Concrete Product**: The specific implementation of the product

**Implementation**:

```java
// Product interface
interface Vehicle {
    void drive();
}

// Concrete products
class Car implements Vehicle {
    @Override
    public void drive() {
        System.out.println("Driving a car");
    }
}

class Bike implements Vehicle {
    @Override
    public void drive() {
        System.out.println("Riding a bike");
    }
}

// Creator
abstract class VehicleFactory {
    abstract Vehicle createVehicle();
    
    public void deliverVehicle() {
        Vehicle vehicle = createVehicle();
        vehicle.drive();
    }
}

// Concrete creators
class CarFactory extends VehicleFactory {
    @Override
    Vehicle createVehicle() {
        return new Car();
    }
}

class BikeFactory extends VehicleFactory {
    @Override
    Vehicle createVehicle() {
        return new Bike();
    }
}
```

**When to Use**:
- When a class can't anticipate the type of objects it needs to create
- When a class wants its subclasses to specify the objects it creates
- When you want to localize the knowledge of which class gets created

### Abstract Factory Pattern

**Purpose**: Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

**Key Components**:
- **Abstract Factory**: Declares creation methods for each product
- **Concrete Factory**: Implements creation methods
- **Abstract Product**: Declares an interface for a type of product
- **Concrete Product**: Implements the abstract product interface
- **Client**: Uses only the interfaces declared by abstract factory and product

**Implementation**:

```java
// Abstract products
interface Button {
    void render();
}

interface Checkbox {
    void check();
}

// Concrete products
class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

class WindowsCheckbox implements Checkbox {
    @Override
    public void check() {
        System.out.println("Windows checkbox checked");
    }
}

class MacCheckbox implements Checkbox {
    @Override
    public void check() {
        System.out.println("Mac checkbox checked");
    }
}

// Abstract factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}
```

**When to Use**:
- When your code needs to work with various families of related products
- When you want to provide a library of products without revealing implementation details
- When you want to constrain object creation to ensure compatibility

### Builder Pattern

**Purpose**: Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Key Components**:
- **Builder**: Interface declaring steps to build parts of a product
- **Concrete Builder**: Implements builder interface, tracks its creation
- **Director**: Constructs object using the builder interface
- **Product**: The complex object being built

**Implementation**:

```java
// Product
class House {
    private String foundation;
    private String structure;
    private String roof;
    
    // Getters, setters, etc.
}

// Builder
interface HouseBuilder {
    void buildFoundation();
    void buildStructure();
    void buildRoof();
    House getResult();
}

// Concrete Builder
class ConcreteHouseBuilder implements HouseBuilder {
    private House house = new House();
    
    @Override
    public void buildFoundation() {
        house.setFoundation("Concrete Foundation");
    }
    
    @Override
    public void buildStructure() {
        house.setStructure("Concrete Walls");
    }
    
    @Override
    public void buildRoof() {
        house.setRoof("Concrete Roof");
    }
    
    @Override
    public House getResult() {
        return house;
    }
}

// Director
class HouseDirector {
    public House constructHouse(HouseBuilder builder) {
        builder.buildFoundation();
        builder.buildStructure();
        builder.buildRoof();
        return builder.getResult();
    }
}
```

**When to Use**:
- When the algorithm for creating a complex object should be independent of the parts and how they're assembled
- When the construction process must allow different representations for the object being constructed
- When you need to control the construction process more precisely

### Singleton Pattern

**Purpose**: Ensure a class has only one instance, and provide a global point of access to it.

**Key Components**:
- **Singleton Class**: Holds a private static instance of itself and provides a public method to access that instance

**Implementation**:

```java
public class Singleton {
    // Eager initialization
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {
        // Private constructor to prevent instantiation
    }
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// Lazy initialization (thread-safe)
public class LazySingleton {
    private static volatile LazySingleton instance;
    
    private LazySingleton() {
        // Private constructor to prevent instantiation
    }
    
    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

**When to Use**:
- When exactly one instance of a class is needed
- When you need stricter control over global variables
- For resources like configuration managers, connection pools, thread pools, caches, etc.

## Structural Patterns

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

### Adapter Pattern

**Purpose**: Allow incompatible interfaces to work together by wrapping an object in an adapter that conforms to another object's expected interface.

**Key Components**:
- **Target**: The interface the client expects to use
- **Adapter**: Converts one interface to another
- **Adaptee**: The incompatible interface that needs adapting
- **Client**: Collaborates with objects conforming to the Target interface

**Implementation**:

```java
// Target interface
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee interface
interface AdvancedMediaPlayer {
    void playVlc(String fileName);
    void playMp4(String fileName);
}

// Concrete Adaptee
class VlcPlayer implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String fileName) {
        System.out.println("Playing vlc file: " + fileName);
    }
    
    @Override
    public void playMp4(String fileName) {
        // Do nothing
    }
}

// Adapter
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer = new VlcPlayer();
        }
        // More types...
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        }
        // More types...
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        // Native support
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        } 
        // Adapter support
        else if (audioType.equalsIgnoreCase("vlc")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        }
        // More types...
    }
}
```

**When to Use**:
- When you want to use an existing class, but its interface doesn't match what you need
- When you want to create a reusable class that cooperates with unrelated or unforeseen classes
- When you need to use several existing subclasses but it's impractical to adapt their interface

### Bridge Pattern

**Purpose**: Decouple an abstraction from its implementation so that the two can vary independently.

**Key Components**:
- **Abstraction**: The interface that controls the abstraction
- **Refined Abstraction**: Extensions of the abstraction
- **Implementor**: The interface for the implementation classes
- **Concrete Implementor**: The specific implementation classes

**Implementation**:

```java
// Implementor
interface DrawAPI {
    void drawCircle(int radius, int x, int y);
}

// Concrete Implementors
class RedCircle implements DrawAPI {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("Drawing red circle of radius " + radius);
    }
}

class BlueCircle implements DrawAPI {
    @Override
    public void drawCircle(int radius, int x, int y) {
        System.out.println("Drawing blue circle of radius " + radius);
    }
}

// Abstraction
abstract class Shape {
    protected DrawAPI drawAPI;
    
    protected Shape(DrawAPI drawAPI) {
        this.drawAPI = drawAPI;
    }
    
    public abstract void draw();
}

// Refined Abstraction
class Circle extends Shape {
    private int radius, x, y;
    
    public Circle(int radius, int x, int y, DrawAPI drawAPI) {
        super(drawAPI);
        this.radius = radius;
        this.x = x;
        this.y = y;
    }
    
    @Override
    public void draw() {
        drawAPI.drawCircle(radius, x, y);
    }
}
```

**When to Use**:
- When you want to avoid a permanent binding between an abstraction and its implementation
- When both the abstractions and their implementations should be extensible independently
- When changes in the implementation should not impact the client code

### Composite Pattern

**Purpose**: Compose objects into tree structures to represent part-whole hierarchies, allowing clients to treat individual objects and compositions uniformly.

**Key Components**:
- **Component**: The abstract class/interface for all components
- **Leaf**: Individual objects with no children
- **Composite**: Components that may contain children
- **Client**: Manipulates objects through the Component interface

**Implementation**:

```java
// Component
interface Employee {
    void showDetails();
}

// Leaf
class Developer implements Employee {
    private String name;
    private String position;
    
    public Developer(String name, String position) {
        this.name = name;
        this.position = position;
    }
    
    @Override
    public void showDetails() {
        System.out.println(name + " - " + position);
    }
}

// Composite
class Manager implements Employee {
    private String name;
    private String position;
    private List<Employee> employees = new ArrayList<>();
    
    public Manager(String name, String position) {
        this.name = name;
        this.position = position;
    }
    
    public void addEmployee(Employee employee) {
        employees.add(employee);
    }
    
    @Override
    public void showDetails() {
        System.out.println(name + " - " + position);
        System.out.println("Team members:");
        
        for (Employee employee : employees) {
            employee.showDetails();
        }
    }
}
```

**When to Use**:
- When you want to represent part-whole hierarchies of objects
- When you want clients to be able to ignore the difference between compositions of objects and individual objects
- When the structure can have any level of complexity

### Decorator Pattern

**Purpose**: Add responsibilities to objects dynamically, providing a flexible alternative to subclassing for extending functionality.

**Key Components**:
- **Component**: The interface for objects that can have responsibilities added to them
- **Concrete Component**: The object to which additional responsibilities can be attached
- **Decorator**: The class that maintains a reference to a Component and conforms to Component's interface
- **Concrete Decorator**: Adds responsibilities to the component

**Implementation**:

```java
// Component
interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete Component
class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 1.0;
    }
    
    @Override
    public String getDescription() {
        return "Simple coffee";
    }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
    
    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
}

// Concrete Decorator
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public double getCost() {
        return super.getCost() + 0.5;
    }
    
    @Override
    public String getDescription() {
        return super.getDescription() + ", with milk";
    }
}
```

**When to Use**:
- When you need to add responsibilities to objects dynamically and transparently
- When extension by subclassing is impractical
- When you have many small features that can be mixed and matched

### Facade Pattern

**Purpose**: Provide a unified interface to a set of interfaces in a subsystem, defining a higher-level interface that makes the subsystem easier to use.

**Key Components**:
- **Facade**: Provides a simplified interface to a complex subsystem
- **Subsystem Classes**: The complex components that the facade simplifies
- **Client**: Uses the facade instead of accessing the subsystem directly

**Implementation**:

```java
// Subsystem classes
class CPU {
    public void freeze() { System.out.println("CPU freezing"); }
    public void jump(long position) { System.out.println("CPU jumping to " + position); }
    public void execute() { System.out.println("CPU executing"); }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory loading data at " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive reading sector " + lba);
        return new byte[size];
    }
}

// Facade
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
}

// Client
class Client {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();
    }
}
```

**When to Use**:
- When you want to provide a simple interface to a complex subsystem
- When there are many dependencies between clients and the implementation classes
- When you want to layer your subsystems

### Flyweight Pattern

**Purpose**: Use sharing to support large numbers of fine-grained objects efficiently.

**Key Components**:
- **Flyweight**: The interface through which flyweights can receive and act on extrinsic state
- **Concrete Flyweight**: Implements the Flyweight interface and stores intrinsic state
- **Flyweight Factory**: Creates and manages flyweight objects
- **Client**: Maintains references to flyweights and computes/stores extrinsic state

**Implementation**:

```java
// Flyweight
interface TextFormat {
    void render(String text);
}

// Concrete Flyweight
class CharacterFormat implements TextFormat {
    private String font;
    private int size;
    private boolean bold;
    
    public CharacterFormat(String font, int size, boolean bold) {
        this.font = font;
        this.size = size;
        this.bold = bold;
    }
    
    @Override
    public void render(String text) {
        System.out.println("Rendering text '" + text + "' in " + font + 
                           " font, size " + size + (bold ? ", bold" : ""));
    }
}

// Flyweight Factory
class FormatFactory {
    private static Map<String, TextFormat> formats = new HashMap<>();
    
    public static TextFormat getFormat(String font, int size, boolean bold) {
        String key = font + "-" + size + "-" + bold;
        
        if (!formats.containsKey(key)) {
            formats.put(key, new CharacterFormat(font, size, bold));
        }
        
        return formats.get(key);
    }
}
```

**When to Use**:
- When an application uses a large number of objects that have some shared state
- When most object state can be made extrinsic
- When you need to save RAM by sharing common parts of state between multiple objects

### Proxy Pattern

**Purpose**: Provide a surrogate or placeholder for another object to control access to it.

**Key Components**:
- **Subject**: Interface implemented by the RealSubject and Proxy
- **RealSubject**: The real object that the proxy represents
- **Proxy**: Maintains a reference to the RealSubject and controls access to it

**Implementation**:

```java
// Subject
interface Image {
    void display();
}

// Real Subject
class RealImage implements Image {
    private String fileName;
    
    public RealImage(String fileName) {
        this.fileName = fileName;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + fileName + " from disk");
    }
    
    @Override
    public void display() {
        System.out.println("Displaying " + fileName);
    }
}

// Proxy
class ProxyImage implements Image {
    private String fileName;
    private RealImage realImage;
    
    public ProxyImage(String fileName) {
        this.fileName = fileName;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(fileName);
        }
        realImage.display();
    }
}
```

**When to Use**:
- When you need a more versatile or sophisticated reference to an object than a simple pointer
- For lazy initialization (virtual proxy)
- For access control (protection proxy)
- For logging, caching, etc.

## Behavioral Patterns

Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects.

### Chain of Responsibility Pattern

**Purpose**: Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request.

**Key Components**:
- **Handler**: Interface for handling requests and accessing successor
- **Concrete Handler**: Handles requests it is responsible for, or passes to successor
- **Client**: Initiates the request to a handler in the chain

**Implementation**:

```java
// Handler
abstract class LogHandler {
    protected LogHandler nextHandler;
    protected int level;
    
    public void setNextHandler(LogHandler nextHandler) {
        this.nextHandler = nextHandler;
    }
    
    public void logMessage(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        
        if (nextHandler != null) {
            nextHandler.logMessage(level, message);
        }
    }
    
    protected abstract void write(String message);
}

// Concrete Handlers
class ConsoleLogger extends LogHandler {
    public ConsoleLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Console Logger: " + message);
    }
}

class FileLogger extends LogHandler {
    public FileLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("File Logger: " + message);
    }
}
```

**When to Use**:
- When more than one object may handle a request, and the handler isn't known in advance
- When you want to issue a request to one of several objects without specifying the receiver explicitly
- When the set of objects that can handle a request should be specified dynamically

### Command Pattern

**Purpose**: Encapsulate a request as an object, allowing you to parameterize clients with different requests, queue or log requests, and support undoable operations.

**Key Components**:
- **Command**: Interface for executing operation
- **Concrete Command**: Implements Command, defines binding between Receiver and action
- **Invoker**: Asks the command to execute the request
- **Receiver**: Knows how to perform the operation
- **Client**: Creates and configures concrete command objects

**Implementation**:

```java
// Receiver
class Light {
    public void turnOn() {
        System.out.println("Light is on");
    }
    
    public void turnOff() {
        System.out.println("Light is off");
    }
}

// Command
interface Command {
    void execute();
}

// Concrete Commands
class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
}

class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOff();
    }
}

// Invoker
class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
}
```

**When to Use**:
- When you want to parameterize objects with an action to perform
- When you want to specify, queue, and execute requests at different times
- When you need support for undoable operations
- When you want to structure a system around high-level operations built on primitive operations

### Iterator Pattern

**Purpose**: Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

**Key Components**:
- **Iterator**: Interface for accessing and traversing elements
- **Concrete Iterator**: Implements the Iterator interface
- **Aggregate**: Interface that defines creating an Iterator object
- **Concrete Aggregate**: Implements the Aggregate interface

**Implementation**:

```java
// Iterator
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Concrete Iterator
class NameIterator implements Iterator<String> {
    private String[] names;
    private int index;
    
    public NameIterator(String[] names) {
        this.names = names;
        this.index = 0;
    }
    
    @Override
    public boolean hasNext() {
        return index < names.length;
    }
    
    @Override
    public String next() {
        if (this.hasNext()) {
            return names[index++];
        }
        return null;
    }
}

// Aggregate
interface Container {
    Iterator<String> getIterator();
}

// Concrete Aggregate
class NameRepository implements Container {
    private String[] names = {"Alice", "Bob", "Charlie", "David"};
    
    @Override
    public Iterator<String> getIterator() {
        return new NameIterator(names);
    }
}
```

**When to Use**:
- When you want to access an aggregate object's contents without exposing its internal representation
- When you want to support multiple traversals of aggregate objects
- When you want to provide a uniform interface for traversing different aggregate structures
- When you need to decouple algorithms from the objects on which they operate

### Mediator Pattern

**Purpose**: Define an object that encapsulates how a set of objects interact, promoting loose coupling by keeping objects from referring to each other explicitly.

**Key Components**:
- **Mediator**: Interface that defines communication between colleague objects
- **Concrete Mediator**: Coordinates communication between colleague objects
- **Colleague**: Interface for objects that communicate through the mediator
- **Concrete Colleague**: Implements the Colleague interface

**Implementation**:

```java
// Mediator
interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}

// Concrete Mediator
class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();
    
    @Override
    public void sendMessage(String message, User user) {
        for (User u : users) {
            // Message should not be received by the user sending it
            if (u != user) {
                u.receive(message);
            }
        }
    }
    
    @Override
    public void addUser(User user) {
        users.add(user);
    }
}

// Colleague
abstract class User {
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message);
}

// Concrete Colleague
class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }
    
    @Override
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }
    
    @Override
    public void receive(String message) {
        System.out.println(name + " receives: " + message);
    }
}
```

**When to Use**:
- When communication between objects is complex with many-to-many relationships
- When reusing an object is difficult because it communicates with many other objects
- When you want to customize behavior distributed between several classes without creating too many subclasses

### Memento Pattern

**Purpose**: Capture and externalize an object's internal state so that the object can be restored to this state later, without violating encapsulation.

**Key Components**:
- **Originator**: Creates a memento containing a snapshot of its internal state
- **Memento**: Stores the internal state of the Originator
- **Caretaker**: Responsible for the memento's safekeeping

**Implementation**:

```java
// Memento
class EditorMemento {
    private final String content;
    
    public EditorMemento(String content) {
        this.content = content;
    }
    
    public String getContent() {
        return content;
    }
}

// Originator
class Editor {
    private String content;
    
    public void setContent(String content) {
        this.content = content;
    }
    
    public String getContent() {
        return content;
    }
    
    public EditorMemento save() {
        return new EditorMemento(content);
    }
    
    public void restore(EditorMemento memento) {
        content = memento.getContent();
    }
}

// Caretaker
class History {
    private List<EditorMemento> states = new ArrayList<>();
    
    public void push(EditorMemento state) {
        states.add(state);
    }
    
    public EditorMemento pop() {
        int lastIndex = states.size() - 1;
        EditorMemento lastState = states.get(lastIndex);
        states.remove(lastState);
        return lastState;
    }
}
```

**When to Use**:
- When you need to save snapshots of an object's state so you can restore it later
- When a direct interface to obtaining the state would expose implementation details and break encapsulation
- For implementing undo mechanisms
- For implementing transaction rollbacks

### Observer Pattern

**Purpose**: Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Key Components**:
- **Subject**: Interface with methods to attach and detach observers
- **Concrete Subject**: Implements the Subject interface, maintains state of interest to observers
- **Observer**: Interface for objects that should be notified of changes in a subject
- **Concrete Observer**: Implements the Observer interface, maintains a reference to a Subject

**Implementation**:

```java
// Observer
interface Observer {
    void update(float temperature, float humidity, float pressure);
}

// Subject
interface Subject {
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

// Concrete Subject
class WeatherData implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private float temperature;
    private float humidity;
    private float pressure;
    
    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }
    
    @Override
    public void removeObserver(Observer o) {
        observers.remove(o);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObservers();
    }
}

// Concrete Observer
class Display implements Observer {
    private float temperature;
    private float humidity;
    private Subject weatherData;
    
    public Display(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
    
    @Override
    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }
    
    private void display() {
        System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
    }
}