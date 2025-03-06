# Flyweight Design Pattern

## Introduction
The Flyweight Design Pattern is a structural design pattern that focuses on minimizing memory usage by sharing as much data as possible with similar objects. It's particularly useful when you need to create a large number of similar objects.

## Intent
- **Save memory** when dealing with a large number of similar objects
- Separate intrinsic (shared) state from extrinsic (unique) state
- Reduce the overall memory footprint of an application

## Real-World Analogy
Consider a game like Counter-Strike, Battle City, or similar games where there are multiple instances of the same type of enemy or player. Without the Flyweight pattern, each instance would require significant memory.

## Problem Scenario
Let's consider a game with numerous player objects:

```java
class Player {
    private int currentPosition;  // 12 bytes
    private int currentHealth;    // 4 bytes
    private int offensivePower;   // 4 bytes
    private int initialHealth;    // 4 bytes
    private String avatar;        // ~1 MB (image data)
    private int score;            // 4 bytes
    private int direction;        // 12 bytes
    // ...and more attributes
}
```

If we have 10,000 player objects, each consuming approximately 1MB, we would need ~10GB of memory. This is problematic, especially with limited RAM (e.g., 256 MB).

## Solution: Flyweight Pattern

The key insight is to separate properties into:

1. **Intrinsic properties** (shared, unchanging)
   - Avatar image
   - Initial health
   - Offensive power

2. **Extrinsic properties** (unique to each instance)
   - Current position
   - Current health
   - Score
   - Direction

### Memory Calculation:
- Before applying Flyweight: 10,000 objects × 1MB ≈ 10GB
- After applying Flyweight: 
  - 4 flyweight objects (different character types) × 1MB ≈ 4MB
  - 10,000 player objects with only extrinsic properties: 10,000 × (12+4+4+12) bytes ≈ 320KB
  - Total: ~4.4MB (significant memory savings)

## Implementation Details

### Class Structure

```java
// Flyweight class that stores intrinsic state
class PlayerFlyweight {
    private int offensivePower;  // 4 bytes
    private int initialHealth;   // 4 bytes
    private String avatar;       // ~1 MB
    
    // Constructor, getters, etc.
}

// Class using the flyweight
class Player {
    private int currentPosition;  // 12 bytes
    private int currentHealth;    // 4 bytes
    private int score;            // 4 bytes
    private int direction;        // 12 bytes
    private PlayerFlyweight playerType;  // Reference (4 bytes)
    
    // Constructor, methods, etc.
}
```

### Flyweight Factory

```java
class PlayerFlyweightFactory {
    private Map<String, PlayerFlyweight> flyweights = new HashMap<>();
    
    public PlayerFlyweight getFlyweight(String type) {
        if (!flyweights.containsKey(type)) {
            switch (type) {
                case "DumbEnemy":
                    flyweights.put(type, new PlayerFlyweight(10, 10, "dumb_enemy.png"));
                    break;
                case "Bomber":
                    flyweights.put(type, new PlayerFlyweight(50, 50, "bomber.png"));
                    break;
                case "SmallTank":
                    flyweights.put(type, new PlayerFlyweight(100, 100, "small_tank.png"));
                    break;
                case "Machine":
                    flyweights.put(type, new PlayerFlyweight(1000, 1000, "machine.png"));
                    break;
            }
        }
        return flyweights.get(type);
    }
}
```

### Usage Example

```java
// Create the factory
PlayerFlyweightFactory factory = new PlayerFlyweightFactory();

// Create different enemy types
PlayerFlyweight dumbEnemyFW = factory.getFlyweight("DumbEnemy");
PlayerFlyweight bomberFW = factory.getFlyweight("Bomber");
PlayerFlyweight smallTankFW = factory.getFlyweight("SmallTank");
PlayerFlyweight machineFW = factory.getFlyweight("Machine");

// Create actual game objects
List<Player> gameObjects = new ArrayList<>();

// Create 7000 dumb enemies
for (int i = 0; i < 7000; i++) {
    gameObjects.add(new Player(getRandomPosition(), 10, 0, getRandomDirection(), dumbEnemyFW));
}

// Create 2000 bombers
for (int i = 0; i < 2000; i++) {
    gameObjects.add(new Player(getRandomPosition(), 50, 0, getRandomDirection(), bomberFW));
}

// Create 700 small tanks
for (int i = 0; i < 700; i++) {
    gameObjects.add(new Player(getRandomPosition(), 100, 0, getRandomDirection(), smallTankFW));
}

// Create 300 machines
for (int i = 0; i < 300; i++) {
    gameObjects.add(new Player(getRandomPosition(), 1000, 0, getRandomDirection(), machineFW));
}
```

## UML Diagram for Flyweight Pattern

```
+----------------+       +--------------------+
|     Player     |------>| PlayerFlyweight    |
+----------------+       +--------------------+
| -currentPos    |       | -offensivePower    |
| -currentHealth |       | -initialHealth     |
| -score         |       | -avatar            |
| -direction     |       +--------------------+
| -playerType    |
+----------------+
        ^
        |
        | creates
        |
+----------------------+
| PlayerFlyweightFactory|
+----------------------+
| -flyweights          |
| +getFlyweight()      |
+----------------------+
```

## Common Questions

### Q1: Why can't we keep intrinsic properties in the Player class itself?
If we kept intrinsic properties in each Player object, we would need to duplicate large data (like avatar images) for every single object. By extracting these into flyweights, we store them only once.

### Q2: Why are intrinsic properties not static inside the Player class?
Using static would give us only one copy of each property for the entire Player class, but we need different sets of intrinsic properties for different player types (e.g., DumbEnemy, Bomber, etc.). The Flyweight pattern allows for sharing among instances of the same type.

## Benefits of the Flyweight Pattern

1. **Memory efficiency**: Significantly reduces memory usage when dealing with a large number of similar objects
2. **Performance improvement**: Less memory allocation/deallocation means better performance
3. **Scalability**: Allows the application to handle more objects within memory constraints
4. **Separation of concerns**: Clearly separates intrinsic (shared) and extrinsic (unique) state

## When to Use

- When your application needs to create a large number of similar objects
- When memory consumption is a concern
- When most of an object's state can be made extrinsic (moved outside the object)
- When removing extrinsic state would allow you to replace many groups of objects with relatively few shared objects

## When Not to Use

- When you don't have many similar objects
- When memory consumption is not a concern
- When the shared state is minimal compared to the unique state
- When the complexity of implementing the pattern outweighs the memory benefits