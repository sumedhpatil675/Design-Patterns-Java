# UML Diagrams

## Introduction
UML (Unified Modeling Language) is a standardized way to visualize the design of a system. It provides a common language for software developers to document, visualize, and design software systems.

## Importance of UML Diagrams

### Collaboration and Communication
- Necessary for stakeholders to write software effectively
- Enables non-ambiguous communication
- Fast and easy to understand compared to text-based communication

### Benefits of Visual Communication
- "A picture is worth a thousand words"
- Text (emails, docs) can be:
  - Ambiguous
  - Slow to understand

### Standardization
- Images can also be ambiguous if not standardized
- UML provides a common visual language for software concepts
- Resolves ambiguity through standardized notation

## Categories of UML Diagrams

### Structural UML Diagrams
Focuses on **how codebase is structured**:
1. Class Diagram
2. Object Diagram
3. Package Diagram
4. Component Diagram

### Behavioral UML Diagrams
Focuses on **how the system works**:
1. Use Case Diagram
2. Activity Diagram
3. Sequence Diagram

## Class Diagrams

### Purpose
1. Represent entities in a software system:
   - Classes
   - Interfaces
   - Enums
   - Abstract classes

2. Show relationships between classes:
   - Implementation relationships (who implements what)
   - Inheritance relationships (what extends what)
   - Composition/aggregation relationships (what is an attribute of what)

### Representation of a Class
A rectangle with 3 sections:
1. Name (top section)
2. Attributes (middle section)
3. Methods (bottom section)

### Representing Attributes
```
class Student {
    private int age;
    private String name;
    public static String schoolName;
    public static final int FEES;
}
```

In UML:
```
        Student
-------------------------
- age : int
- name : String
+ schoolName : String
+ FEES : int
-------------------------
```

### Access Modifiers in UML
- `-` (minus): private
- `+` (plus): public
- `#` (hash): protected
- `~` (tilde): default/package

### Special Notations
- Static members are underlined
- Final members are shown in CAPITAL letters

### Representing Methods
```java
class Student {
    public int solveQuiz(int qId, int qType) {
        // Implementation
    }
}
```

In UML:
```
        Student
-------------------------
-------------------------
+ solveQuiz(qId : int, qType : int) : int
```

### Representing Interfaces
```
<<Interface>>
Flyable
-------------------------
+ fly() : void
```

### Abstract Classes and Methods
- Abstract class names are in *italics*
- Abstract methods are in *italics*

```
     *Animal*
-------------------------
-------------------------
+ fly() : void
+ *eat()* : void
```

### Enums
```
    TrafficLight
-------------------------
RED, GREEN, YELLOW
-------------------------
```

## Relationships in UML

### 1. IS-A Relationship (Inheritance)
- Class implements an interface
- Class extends another class
- Represented by an arrow pointing from child to parent

```
    Animal
      ^
      |
    Cat
```

### 2. HAS-A Relationship (Association)
- One class contains reference(s) to another
- Two types:
  - **Aggregation** (Weak Association): represented by an empty diamond
  - **Composition** (Strong Association): represented by a filled diamond

#### Aggregation Example
```java
class Batch {
    List<Student> students;
}
```

Diagram:
```
    Batch ◇———— Student
```

The "contained" class (Student) has a separate, independent existence from the "container" class (Batch).

#### Composition Example
```java
class Show {
    List<Ticket> tickets;
}
```

Diagram:
```
    Show ◆———— Ticket
```

In composition, the "contained" class (Ticket) cannot exist without the "container" class (Show).

## Best Practices for UML Diagrams

1. Keep diagrams simple and focused
2. Use consistent notation
3. Include only relevant details
4. Use proper layout (arrangement of elements)
5. Add notes where clarification is needed
6. Use appropriate diagram types for specific tasks
7. Consider your audience when creating diagrams