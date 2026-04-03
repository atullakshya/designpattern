# Prototype Pattern

## Definition
The Prototype Pattern creates new objects by **copying an existing object (prototype)** rather than creating from scratch. This is useful when creating an object is expensive or complex, and you have a working object you want to duplicate.

## Core Concept
- **Clone, Don't Create**: Copy existing objects instead of creating new ones
- **Deep vs Shallow Copy**: Understand the difference for nested objects
- **Expensive Operations**: Avoid costly object initialization
- **Polymorphic Cloning**: Different object types implement cloning differently

## Problem It Solves
- Object creation is resource-intensive or time-consuming
- Objects have complex initialization requiring many steps
- Need multiple variations of similar objects
- Avoiding tight coupling to concrete classes during object creation

## Generic Implementation

### Basic Prototype Pattern with ICloneable

```csharp
// Prototype Interface
public interface IPrototype<T>
{
    T Clone();
}

// Concrete Prototype
public class Employee : IPrototype<Employee>
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public double Salary { get; set; }

    public Employee Clone()
    {
        // Shallow copy - sufficient for primitive types
        return (Employee)this.MemberwiseClone();
    }

    public override string ToString()
    {
        return $"Employee: Id={Id}, Name={Name}, Department={Department}, Salary={Salary}";
    }
}

// Usage
var originalEmployee = new Employee
{
    Id = 1,
    Name = "John",
    Department = "IT",
    Salary = 50000
};

var clonedEmployee = originalEmployee.Clone();
clonedEmployee.Id = 2;
clonedEmployee.Name = "John Copy";

Console.WriteLine(originalEmployee); // Original unchanged
Console.WriteLine(clonedEmployee);   // Modified clone
```

### Deep Copy Pattern (For Complex Objects)

```csharp
// Complex nested structure
public class Address : ICloneable
{
    public string Street { get; set; }
    public string City { get; set; }
    public string ZipCode { get; set; }

    public object Clone()
    {
        return new Address
        {
            Street = this.Street,
            City = this.City,
            ZipCode = this.ZipCode
        };
    }

    public override string ToString()
    {
        return $"{Street}, {City} {ZipCode}";
    }
}

public class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }  // Reference type
    public List<string> Hobbies { get; set; }

    // Deep copy - recursively clones all nested objects
    public object Clone()
    {
        return new Person
        {
            Name = this.Name,
            Age = this.Age,
            Address = (Address)this.Address?.Clone(),  // Clone nested object
            Hobbies = new List<string>(this.Hobbies)   // Clone collection
        };
    }

    public override string ToString()
    {
        var hobbies = string.Join(", ", Hobbies ?? new List<string>());
        return $"Person: {Name}, Age: {Age}, Address: {Address}, Hobbies: {hobbies}";
    }
}

// Usage
var originalPerson = new Person
{
    Name = "Alice",
    Age = 30,
    Address = new Address { Street = "123 Main St", City = "Boston", ZipCode = "02101" },
    Hobbies = new List<string> { "Reading", "Swimming" }
};

var clonedPerson = (Person)originalPerson.Clone();
clonedPerson.Name = "Alice Copy";
clonedPerson.Address.City = "New York";
clonedPerson.Hobbies.Add("Gaming");

Console.WriteLine("Original:");
Console.WriteLine(originalPerson);
Console.WriteLine("\nClone:");
Console.WriteLine(clonedPerson);
```

### Prototype Registry Pattern

```csharp
// Prototype that can be cloned
public abstract class Shape : ICloneable
{
    public string Color { get; set; }
    public abstract object Clone();
    public abstract void Draw();
}

public class Circle : Shape
{
    public int Radius { get; set; }

    public override object Clone()
    {
        return new Circle
        {
            Color = this.Color,
            Radius = this.Radius
        };
    }

    public override void Draw()
    {
        Console.WriteLine($"Drawing Circle: Color={Color}, Radius={Radius}");
    }
}

public class Rectangle : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public override object Clone()
    {
        return new Rectangle
        {
            Color = this.Color,
            Width = this.Width,
            Height = this.Height
        };
    }

    public override void Draw()
    {
        Console.WriteLine($"Drawing Rectangle: Color={Color}, Width={Width}, Height={Height}");
    }
}

// Shape Registry - Prototype Manager
public class ShapeRegistry
{
    private Dictionary<string, Shape> _prototypes = new();

    public void Register(string key, Shape prototype)
    {
        _prototypes[key] = prototype;
    }

    public Shape CreateShape(string key)
    {
        if (!_prototypes.ContainsKey(key))
            throw new ArgumentException($"Shape prototype '{key}' not registered");

        return (Shape)_prototypes[key].Clone();
    }

    public IEnumerable<string> GetRegisteredShapes() => _prototypes.Keys;
}

// Usage
var registry = new ShapeRegistry();

// Register prototypes
var redCirclePrototype = new Circle { Color = "Red", Radius = 5 };
var blueRectanglePrototype = new Rectangle { Color = "Blue", Width = 10, Height = 20 };

registry.Register("redCircle", redCirclePrototype);
registry.Register("blueRectangle", blueRectanglePrototype);

// Create clones from prototypes
var shape1 = registry.CreateShape("redCircle");
var shape2 = registry.CreateShape("blueRectangle");
var shape3 = registry.CreateShape("redCircle");

shape1.Draw();
shape2.Draw();
shape3.Draw();
```

## Real-World Examples

### Configuration Cloning
```csharp
public class DatabaseConfig : ICloneable
{
    public string Server { get; set; }
    public string Database { get; set; }
    public string Username { get; set; }
    public int ConnectionTimeout { get; set; }
    public Dictionary<string, string> Options { get; set; }

    public object Clone()
    {
        return new DatabaseConfig
        {
            Server = this.Server,
            Database = this.Database,
            Username = this.Username,
            ConnectionTimeout = this.ConnectionTimeout,
            Options = new Dictionary<string, string>(this.Options)
        };
    }
}

// Usage
var productionConfig = new DatabaseConfig
{
    Server = "prod-server",
    Database = "prod-db",
    Username = "admin",
    ConnectionTimeout = 30,
    Options = new Dictionary<string, string> { { "Pooling", "true" } }
};

var testConfig = (DatabaseConfig)productionConfig.Clone();
testConfig.Server = "test-server";
testConfig.Database = "test-db";
testConfig.Username = "testuser";

Console.WriteLine($"Production: {productionConfig.Server}/{productionConfig.Database}");
Console.WriteLine($"Test: {testConfig.Server}/{testConfig.Database}");
```

### Document Cloning
```csharp
public class Document : ICloneable
{
    public string Title { get; set; }
    public string Content { get; set; }
    public List<string> Tags { get; set; }
    public DateTime CreatedDate { get; set; }

    public object Clone()
    {
        return new Document
        {
            Title = this.Title,
            Content = this.Content,
            Tags = new List<string>(this.Tags),
            CreatedDate = this.CreatedDate
        };
    }

    public override string ToString()
    {
        return $"Document: Title='{Title}', Length={Content?.Length ?? 0}, " +
               $"Tags=[{string.Join(", ", Tags ?? new List<string>())}]";
    }
}

// Usage
var originalDoc = new Document
{
    Title = "Design Patterns",
    Content = "A comprehensive guide to design patterns...",
    Tags = new List<string> { "Programming", "Design" },
    CreatedDate = DateTime.Now
};

var draftDoc = (Document)originalDoc.Clone();
draftDoc.Title = "Design Patterns - DRAFT";
draftDoc.Tags.Add("Draft");

Console.WriteLine(originalDoc);
Console.WriteLine(draftDoc);
```

### Game Entity Cloning
```csharp
public class GameCharacter : ICloneable
{
    public string Name { get; set; }
    public int Health { get; set; }
    public int Mana { get; set; }
    public List<string> Skills { get; set; }
    public Dictionary<string, int> Stats { get; set; }  // Strength, Dexterity, etc.

    public object Clone()
    {
        return new GameCharacter
        {
            Name = this.Name,
            Health = this.Health,
            Mana = this.Mana,
            Skills = new List<string>(this.Skills),
            Stats = new Dictionary<string, int>(this.Stats)
        };
    }

    public override string ToString()
    {
        var stats = string.Join(", ", Stats.Select(s => $"{s.Key}:{s.Value}"));
        return $"{Name} (HP:{Health}, Mana:{Mana}) [{stats}]";
    }
}

// Usage
var warrior = new GameCharacter
{
    Name = "Aragorn",
    Health = 100,
    Mana = 50,
    Skills = new List<string> { "Sword Mastery", "Leadership" },
    Stats = new Dictionary<string, int>
    {
        { "Strength", 18 },
        { "Dexterity", 15 },
        { "Constitution", 16 }
    }
};

var clone = (GameCharacter)warrior.Clone();
clone.Name = "Aragorn Clone";
clone.Health = 50;

Console.WriteLine(warrior);
Console.WriteLine(clone);
```

## Shallow Copy vs Deep Copy

| Aspect | Shallow Copy | Deep Copy |
|--------|---|---|
| **Primitive Types** | Copied completely | Copied completely |
| **Reference Types** | Copied by reference (same object) | New copy created |
| **Memory** | Less memory used | More memory used |
| **Performance** | Faster | Slower |
| **Mutations** | Changes affect both copies | Changes isolated |
| **Use Case** | Immutable nested objects | Mutable nested objects |

## Advantages

✅ **Performance**: Faster than creating complex objects from scratch  
✅ **Flexibility**: Avoids subclassing for object creation  
✅ **Encapsulation**: Clients don't need to know concrete types  
✅ **Ease of Use**: Simple to clone existing objects  
✅ **Runtime Configuration**: Create variants at runtime  

## Disadvantages

❌ **Complexity**: Implementing clone can be tricky (shallow vs deep)  
❌ **Overhead**: Cloning can use significant memory  
❌ **Circular References**: Can cause issues if not handled carefully  
❌ **Shallow Copy Pitfalls**: Easy to miss when deep copy is needed  

## When to Use

- Creating new objects is expensive or complex
- Need multiple instances with slight variations
- Runtime decision on which type of object to create
- Avoiding complex initialization logic
- Creating "templates" for similar objects

## When NOT to Use

- Simple objects that are cheap to create
- Objects with unique state that shouldn't be shared
- When constructor or factory is cleaner
- Circular reference scenarios

## Best Practices

1. **Choose Shallow or Deep Carefully**: Understand your object's structure
2. **Document Cloning Behavior**: Make it clear what gets copied
3. **Test Clones Thoroughly**: Verify independence between original and clone
4. **Consider Copy Semantics**: Ensure cloned objects are truly independent
5. **Use Generic Interface**: `IPrototype<T>` is clearer than `ICloneable`
6. **Avoid Circular References**: Fix before cloning if present

## Summary

The Prototype Pattern is valuable when object creation is costly or complex. It allows you to create new objects by cloning existing ones, but requires careful attention to deep vs shallow copying and proper implementation of cloning semantics.
