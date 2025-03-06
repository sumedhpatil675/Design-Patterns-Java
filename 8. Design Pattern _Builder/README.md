# Builder Design Pattern

## Introduction

The Builder Design Pattern is a creational pattern that separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

## Problem Statement

1. There is a class with many attributes
2. The attributes require validation
3. The class needs to ensure that only valid objects are created
4. Traditional constructors can lead to multiple problems

## Evolution of Object Creation

### V1: Public Attributes (Bad Practice)

```java
class Student {
    // No constructor
    
    // Public data members (BAD OOP practice)
    public String name;
    public int age;
    public int psp; // Problem Solving Percentage
    public String email;
    public String phone;
}

// Client code
public static void main() {
    Student st = new Student();
    st.name = "John";
    st.age = 1000; // Invalid value, but no validation!
    st.psp = 120;  // Invalid value, but no validation!
}
```

**Problems in V1:**
1. Bad OOP - public data members
2. No validation checks

### V2: Constructor with Validation

```java
class Student {
    // Private data members
    private String name;
    private int age;
    private int psp;
    private String email;
    private String phone;
    
    // Public getters (no setters)
    
    // Constructor with validation
    public Student(String name, int age, int psp, String email, String phone) {
        if (name == null || name.length() == 0 || name.length() > 100) {
            throw new IllegalArgumentException("Name should be not null, not blank, ≤ 100 chars");
        }
        
        // More validations...
        
        this.name = name;
        this.age = age;
        // etc.
    }
}
```

**Problems in V2:**
1. Unreadable code, prone to errors
2. If the constructor changes, the code in all clients will also need to change (compile-time error)
3. SRP violation
4. No setters (which is good)

### V3: Multiple Constructors

```java
class Student {
    // Private data members
    private String name;
    private int age;
    private int psp;
    private String email;
    private String phone;
    
    // Public getters (no setters)
    
    // Multiple constructors
    public Student() {
        this("noname", 25, 100, "abc@abc.com", "999");
    }
    
    public Student(String name) {
        this(name, 25, 100, "abc@abc.com", "999");
    }
    
    public Student(String name, int age) {
        this(name, age, 100, "abc@abc.com", "999");
    }
    
    // etc.
}
```

**Problems in V3:**
1. Constructor explosion
2. Not always possible to create 2^n constructors for all attribute combinations

### V4: Map-based Constructor

```java
class Student {
    // Private data members
    private String name;
    private int age;
    private int psp;
    private String email;
    private String phone;
    
    // Constructor using a Map
    public Student(Map<String, Object> map) {
        if (!map.containsKey("name")) {
            throw new IllegalArgumentException("You forgot to pass name");
        }
        
        String name = (String) map.get("name");
        // Validation & setting
    }
}

// Client code
public static void main() {
    Map<String, Object> sb = new HashMap<>();
    sb.put("name", "John");
    sb.put("age", 25);
    sb.put("psp", 75);
    sb.put("email", "abc@abc.com");
    sb.put("phone", "999");
    
    Student st = new Student(sb);
}
```

**Problems in V4:**
1. If-else code has increased (cyclomatic complexity)
2. Error-prone (possible runtime errors with typos, wrong data types)
   - Example: `sb.put("pssp", 20)` - typo will be discovered at runtime, not compile time

## Builder Pattern (V5 & V6)

### V5: Basic Builder Pattern

```java
class Student {
    // Private data members
    private String name;
    private int age;
    private int psp;
    private String email;
    private String phone;
    
    // Private constructor - can only be called by the Builder
    private Student(Builder bldr) {
        this.name = bldr.name;
        this.age = bldr.age;
        this.psp = bldr.psp;
        // etc.
    }
}

class Builder {
    // Public data members (within the Builder)
    public String name;
    public int age;
    public int psp;
    public String email;
    public String phone;
    
    // Builder methods - returns 'this' for method chaining
    public Builder name(String name) {
        if (name == null || name.length() == 0 || name.length() > 100) {
            throw new IllegalArgumentException("Name should be not null, not blank, ≤ 100 chars");
        }
        this.name = name;
        return this;
    }
    
    public Builder age(int age) {
        // Validation
        this.age = age;
        return this;
    }
    
    // Build method - creates the Student
    public Student build() {
        // Validate all required fields
        if (this.name == null) {
            throw new IllegalArgumentException("Name is required");
        }
        
        return new Student(this);
    }
}

// Client code
public static void main() {
    Builder sb = new Builder();
    sb.name("John");
    sb.age(10);
    sb.psp(20);
    sb.email("abc@abc.com");
    sb.phone("999");
    
    Student st = new Student(sb); // Problem: Bypass validation!
}
```

**Problems Solved in V5:**
1. If-else reduced (readability improved)
2. Type safety - typos/data type mismatch become compile-time errors

**Problems in V5:**
1. Public data members in Builder (bad OOP)
2. SRP not solved (validation in Student class)
3. Client can create Student object directly: `new Student(builder)`, bypassing validation

### V6: Improved Builder Pattern with Inner Static Class

```java
class Student {
    // Private data members
    private String name;
    private int age;
    private int psp;
    private String email;
    private String phone;
    
    // Private constructor - only accessible to inner Builder class
    private Student(Builder bldr) {
        this.name = bldr.name;
        this.age = bldr.age;
        this.psp = bldr.psp;
        // etc.
    }
    
    // Static inner Builder class
    public static class Builder {
        // Private data members
        private String name;
        private int age;
        private int psp;
        private String email;
        private String phone;
        
        // Builder methods
        public Builder name(String name) {
            if (name == null || name.length() == 0 || name.length() > 100) {
                throw new IllegalArgumentException("Name should be not null, not blank, ≤ 100 chars");
            }
            this.name = name;
            return this;
        }
        
        // Build method - creates and returns the Student
        public Student build() {
            // Final validation
            if (this.name == null) {
                throw new IllegalArgumentException("Name is required");
            }
            
            return new Student(this);
        }
    }
}

// Client code
public static void main() {
    Student.Builder sb = new Student.Builder();
    sb.name("John")
      .age(10)
      .psp(20)
      .email("abc@abc.com")
      .phone("999");
    
    Student st = sb.build();
}
```

## Why is Builder class static?

The Builder object is created at a time when the Student object does not yet exist. Therefore, we need to use an inner class without requiring an outer object instance. This is why the Builder is defined as a static inner class.

## Benefits of Builder Pattern

1. **Readability**: Makes the code more readable and maintainable
2. **Validation**: Ensures only valid objects are created
3. **Flexibility**: Allows creating different representations with the same process
4. **Telescoping constructors problem solved**: No need for multiple constructors
5. **Immutability**: The created object can be immutable as there are no setters
6. **Method chaining**: Provides a fluent interface for building objects

## When to Use

1. When object creation is complex with many attributes
2. When you need immutability for the created objects
3. When you want to enforce certain validations during object creation
4. When you want to create different representations of the same object

## Class Diagram

```
┌────────────────┐       ┌────────────────────┐
│    Student     │       │  Student.Builder   │
├────────────────┤       ├────────────────────┤
│ - name: String │       │ - name: String     │
│ - age: int     │◄──────│ - age: int         │
│ - psp: int     │builds │ - psp: int         │
│ - email: String│       │ - email: String    │
│ - phone: String│       │ - phone: String    │
├────────────────┤       ├────────────────────┤
│ - Student()    │       │ + name(): Builder  │
│ + getters()    │       │ + age(): Builder   │
│                │       │ + psp(): Builder   │
│                │       │ + email(): Builder │
│                │       │ + phone(): Builder │
│                │       │ + build(): Student │
└────────────────┘       └────────────────────┘
```

## Complete Example

```java
public class Student {
    // Attributes with constraints
    private final String name;     // not null, not blank, ≤ 100 chars
    private final int age;         // 15 ≤ age ≤ 50
    private final int psp;         // 0 ≤ psp ≤ 100
    private final String email;    // not null, not blank, contains '@'
    private final String phone;    // not null, not blank, 10 digits

    // Private constructor
    private Student(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.psp = builder.psp;
        this.email = builder.email;
        this.phone = builder.phone;
    }

    // Getters (no setters for immutability)
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public int getPsp() {
        return psp;
    }

    public String getEmail() {
        return email;
    }

    public String getPhone() {
        return phone;
    }

    // Static Builder class
    public static class Builder {
        // Same attributes as Student
        private String name;
        private int age;
        private int psp;
        private String email;
        private String phone;

        // Builder methods
        public Builder name(String name) {
            if (name == null || name.isEmpty() || name.length() > 100) {
                throw new IllegalArgumentException("Name should be not null, not blank, and ≤ 100 characters");
            }
            this.name = name;
            return this;
        }

        public Builder age(int age) {
            if (age < 15 || age > 50) {
                throw new IllegalArgumentException("Age should be between 15 and 50");
            }
            this.age = age;
            return this;
        }

        public Builder psp(int psp) {
            if (psp < 0 || psp > 100) {
                throw new IllegalArgumentException("PSP should be between 0 and 100");
            }
            this.psp = psp;
            return this;
        }

        public Builder email(String email) {
            if (email == null || email.isEmpty() || !email.contains("@")) {
                throw new IllegalArgumentException("Email should be not null, not blank, and contain '@'");
            }
            this.email = email;
            return this;
        }

        public Builder phone(String phone) {
            if (phone == null || phone.isEmpty() || phone.length() != 10) {
                throw new IllegalArgumentException("Phone should be not null, not blank, and have 10 digits");
            }
            this.phone = phone;
            return this;
        }

        // Build method with final validation
        public Student build() {
            // Check if required fields are set
            if (name == null) {
                throw new IllegalArgumentException("Name is required");
            }
            if (email == null) {
                throw new IllegalArgumentException("Email is required");
            }
            
            // Set defaults for optional fields
            if (age == 0) {
                age = 25; // Default age
            }
            if (psp == 0) {
                psp = 100; // Default psp
            }
            if (phone == null) {
                phone = "0000000000"; // Default phone
            }
            
            return new Student(this);
        }
    }
    
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", psp=" + psp +
                ", email='" + email + '\'' +
                ", phone='" + phone + '\'' +
                '}';
    }

    // Example usage
    public static void main(String[] args) {
        Student student = new Student.Builder()
                .name("John Doe")
                .age(25)
                .psp(95)
                .email("john@example.com")
                .phone("1234567890")
                .build();
        
        System.out.println(student);
        
        // Using default values
        Student student2 = new Student.Builder()
                .name("Jane Doe")
                .email("jane@example.com")
                .build();
        
        System.out.println(student2);
    }
}
```

## Related Patterns

- **Factory Pattern**: Similar to Builder but used when the creation process is more straightforward.
- **Abstract Factory**: Creates families of related objects.
- **Prototype**: Used to copy existing objects rather than building new ones.
- **Singleton**: Ensures a class has only one instance with a global point of access.
