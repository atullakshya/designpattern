# C# Design Patterns - Complete Guide

## Overview

This directory contains comprehensive documentation about C# Design Patterns organized into three main categories: Creational, Structural, and Behavioral patterns.

### What are Design Patterns?

Design patterns are reusable solutions to common programming problems. They provide tested, proven development paradigms that can speed up the development process and make code more maintainable and scalable.

**Key Benefits:**
- ✅ Proven solutions to recurring problems
- ✅ Improved code reusability
- ✅ Better code organization and structure
- ✅ Easier maintenance and refactoring
- ✅ Enhanced communication among developers
- ✅ Reduced development time

---

## 1. CREATIONAL PATTERNS

Creational patterns deal with **object creation mechanisms**. They abstract the instantiation process to make systems independent of how objects are composed and represented.

### 1.1 [Singleton Pattern](1_Creational/01_Singleton_Pattern.md)
**Purpose**: Ensure a class has only one instance and provide a global point of access.

**When to Use:**
- Logger instances
- Database connection pools
- Configuration managers
- Caching systems

**Key Concept**: Control instance creation to have exactly one object throughout application lifetime.

**Generic Approach**:
```csharp
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _instance = 
        new(() => new Singleton());
    
    private Singleton() { }
    public static Singleton Instance => _instance.Value;
}
```

---

### 1.2 [Factory Pattern](1_Creational/02_Factory_Pattern.md)
**Purpose**: Create objects without specifying their exact classes. Hide creation logic and make system independent of product types.

**Types:**
- Simple Factory (not GoF pattern)
- Factory Method
- Abstract Factory

**When to Use:**
- Multiple object types need creation
- Creation logic is complex
- Need to decouple client from concrete classes

**Generic Approach**:
```csharp
public interface IProduct { }
public class ConcreteProductA : IProduct { }
public class ConcreteProductB : IProduct { }

public class Factory
{
    public static IProduct CreateProduct(string type) =>
        type switch
        {
            "A" => new ConcreteProductA(),
            "B" => new ConcreteProductB(),
            _ => throw new ArgumentException()
        };
}
```

---

### 1.3 [Builder Pattern](1_Creational/03_Builder_Pattern.md)
**Purpose**: Construct complex objects step-by-step using a fluent interface.

**When to Use:**
- Objects with many optional parameters
- Complex initialization logic
- Need immutable objects
- Need different representations of objects

**Generic Approach**:
```csharp
public class ComplexObject
{
    public string Property1 { get; set; }
    public string Property2 { get; set; }
}

public class Builder
{
    private ComplexObject _object = new();
    
    public Builder WithProperty1(string value)
    {
        _object.Property1 = value;
        return this;
    }
    
    public ComplexObject Build() => _object;
}

// Usage
var obj = new Builder()
    .WithProperty1("value")
    .Build();
```

---

### 1.4 [Prototype Pattern](1_Creational/04_Prototype_Pattern.md)
**Purpose**: Create new objects by copying an existing prototype instead of creating from scratch.

**When to Use:**
- Object creation is expensive
- Need multiple similar object variations
- Runtime decision on object types
- Avoiding complex initialization

**Generic Approach**:
```csharp
public interface IPrototype<T>
{
    T Clone();
}

public class ConcretePrototype : IPrototype<ConcretePrototype>
{
    public ConcretePrototype Clone() => 
        (ConcretePrototype)this.MemberwiseClone();
}
```

---

## 2. STRUCTURAL PATTERNS

Structural patterns deal with **object composition**. They help compose objects into larger structures while keeping these structures flexible and efficient.

### 2.1 [Adapter Pattern](2_Structural/01_Adapter_Pattern.md)
**Purpose**: Convert interface of a class into another interface clients expect. Make incompatible interfaces work together.

**Types:**
- Class Adapter (inheritance)
- Object Adapter (composition)
- Two-Way Adapter

**When to Use:**
- Integrating third-party libraries
- Legacy code integration
- Converting data formats
- Unifying different interfaces

**Generic Approach**:
```csharp
// Incompatible interface
public interface IOldInterface { void OldMethod(); }

// Expected interface
public interface INewInterface { void NewMethod(); }

// Adapter
public class Adapter : INewInterface
{
    private IOldInterface _old;
    
    public void NewMethod() => _old.OldMethod();
}
```

---

### 2.2 [Decorator Pattern](2_Structural/02_Decorator_Pattern.md)
**Purpose**: Attach additional responsibilities to objects dynamically. Provides flexible alternative to subclassing.

**When to Use:**
- Adding features to objects at runtime
- Avoiding subclass explosion from feature combinations
- Independent feature addition
- Wrapping with multiple layers

**Generic Approach**:
```csharp
public interface IComponent { void Operation(); }

public abstract class Decorator : IComponent
{
    protected IComponent _component;
    
    public Decorator(IComponent component) => _component = component;
    public virtual void Operation() => _component.Operation();
}

public class ConcreteDecorator : Decorator
{
    public override void Operation()
    {
        base.Operation();
        Console.WriteLine("Added functionality");
    }
}
```

---

### 2.3 [Facade Pattern](2_Structural/03_Facade_Pattern.md)
**Purpose**: Provide unified simplified interface to complex subsystem of classes.

**When to Use:**
- Simplifying complex subsystems
- Reducing dependencies between clients and subsystem
- Creating entry points to multi-layered systems
- Grouping related operations

**Generic Approach**:
```csharp
// Complex subsystem with multiple classes
public class SubsystemA { }
public class SubsystemB { }
public class SubsystemC { }

// Facade - Simple interface
public class Facade
{
    private SubsystemA _a = new();
    private SubsystemB _b = new();
    private SubsystemC _c = new();
    
    public void SimpleOperation()
    {
        // Orchestrates complex operations
    }
}
```

---

### 2.4 [Proxy Pattern](2_Structural/04_Proxy_Pattern.md)
**Purpose**: Provide placeholder/surrogate for another object to control access.

**Types:**
- Virtual Proxy (lazy loading)
- Protection Proxy (access control)
- Logging Proxy (monitoring)
- Caching Proxy (performance)
- Remote Proxy (network objects)

**When to Use:**
- Expensive object creation (virtual proxy)
- Access control needed
- Logging/monitoring required
- Caching results
- Remote object access

**Generic Approach**:
```csharp
public interface ISubject { void Operation(); }

public class RealSubject : ISubject { public void Operation() { } }

public class Proxy : ISubject
{
    private RealSubject _real;
    
    public void Operation()
    {
        // Control access, lazy load, cache, etc.
        _real ??= new RealSubject();
        _real.Operation();
    }
}
```

---

## 3. BEHAVIORAL PATTERNS

Behavioral patterns concentrate on **object collaboration** and responsibility distribution. They characterize complex control flows that are difficult to follow at runtime.

### 3.1 [Observer Pattern](3_Behavioral/01_Observer_Pattern.md)
**Purpose**: Define one-to-many dependency where state change in one object notifies all dependent objects automatically.

**Also Known As:** Publish-Subscribe Pattern

**When to Use:**
- Multiple objects react to state changes
- Event-driven architecture
- Decoupling event sources from handlers
- Broadcasting changes to many listeners

**Generic Approach**:
```csharp
public interface IObserver { void Update(string message); }

public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}

public class ConcreteSubject : ISubject
{
    private List<IObserver> _observers = new();
    
    public void Attach(IObserver observer) => _observers.Add(observer);
    public void Detach(IObserver observer) => _observers.Remove(observer);
    
    public void Notify()
    {
        foreach (var observer in _observers)
            observer.Update("State changed");
    }
}
```

---

### 3.2 [Strategy Pattern](3_Behavioral/02_Strategy_Pattern.md)
**Purpose**: Define family of interchangeable algorithms and make them exchangeable. Encapsulate each algorithm.

**When to Use:**
- Multiple algorithms for same task
- Algorithm selected at runtime
- Frequent algorithm additions/changes
- Avoiding conditional logic branches
- Decoupling algorithm from context

**Generic Approach**:
```csharp
public interface IStrategy { void Execute(); }

public class ConcreteStrategyA : IStrategy { public void Execute() { } }
public class ConcreteStrategyB : IStrategy { public void Execute() { } }

public class Context
{
    private IStrategy _strategy;
    
    public void SetStrategy(IStrategy strategy) => _strategy = strategy;
    public void DoWork() => _strategy?.Execute();
}
```

---

### 3.3 [Command Pattern](3_Behavioral/03_Command_Pattern.md)
**Purpose**: Encapsulate request as an object, parameterizing clients with different requests. Support undo/redo.

**When to Use:**
- Decouple request sender from receiver
- Implement undo/redo functionality
- Queue or schedule requests
- Support transactional operations
- Logging/auditing commands

**Generic Approach**:
```csharp
public interface ICommand { void Execute(); }

public class ConcreteCommand : ICommand
{
    private Receiver _receiver;
    
    public ConcreteCommand(Receiver receiver) => _receiver = receiver;
    public void Execute() => _receiver.DoSomething();
}

public class Invoker
{
    private ICommand _command;
    
    public void SetCommand(ICommand command) => _command = command;
    public void ExecuteCommand() => _command?.Execute();
}
```

---

### 3.4 [Template Method Pattern](3_Behavioral/04_Template_Method_Pattern.md)
**Purpose**: Define algorithm skeleton in base class letting subclasses override specific steps without changing structure.

**When to Use:**
- Multiple classes with similar algorithms
- Avoiding code duplication
- Algorithm structure should be constant
- Only specific steps vary
- Enforcing implementation order

**Generic Approach**:
```csharp
public abstract class TemplateClass
{
    public void TemplateMethod()
    {
        FixedStep1();
        VariableStep();
        FixedStep2();
    }
    
    protected abstract void VariableStep();
    
    private void FixedStep1() { }
    private void FixedStep2() { }
}

public class ConcreteClass : TemplateClass
{
    protected override void VariableStep() { }
}
```

---

## Pattern Selection Guide

### Quick Decision Tree

**Need to create objects?**
→ Use **Creational Patterns** (Singleton, Factory, Builder, Prototype)

**Need to compose objects?**
→ Use **Structural Patterns** (Adapter, Decorator, Facade, Proxy)

**Need to manage communication?**
→ Use **Behavioral Patterns** (Observer, Strategy, Command, Template Method)

### By Problem Type

| Problem | Pattern |
|---------|---------|
| Only one instance needed | Singleton |
| Complex object creation | Builder, Factory |
| Incompatible interfaces | Adapter |
| Add features dynamically | Decorator |
| Simplify complex system | Facade |
| Control object access | Proxy |
| Multiple objects react to events | Observer |
| Multiple algorithms for task | Strategy |
| Encapsulate requests | Command |
| Similar algorithms variations | Template Method |

### By Frequency of Changes

| Strategy | Pattern |
|----------|---------|
| Frequently add algorithms | Strategy, Factory |
| Frequently add features | Decorator |
| Frequently add object types | Factory, Prototype |
| Algorithm steps vary | Template Method |
| Access patterns vary | Proxy |

---

## Common Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Overusing Singleton
**Problem**: Creates global state and tight coupling
**Solution**: Use Dependency Injection instead

### ❌ Anti-Pattern 2: Inheritance Explosion
**Problem**: Creating too many subclasses for feature combinations
**Solution**: Use Decorator or Composition

### ❌ Anti-Pattern 3: God Objects
**Problem**: Facade growing too large with too many responsibilities
**Solution**: Break into multiple facades or use composition

### ❌ Anti-Pattern 4: Too Much Abstraction
**Problem**: Using patterns for simple scenarios
**Solution**: Apply YAGNI (You Aren't Gonna Need It) principle

### ❌ Anti-Pattern 5: Memory Leaks in Observer
**Problem**: Observers not unsubscribing
**Solution**: Implement proper cleanup and use weak references when needed

---

## Best Practices

### General Rules

1. **Choose Composition Over Inheritance**
   - Decorator, Strategy, and Proxy prefer composition
   - More flexible and maintainable

2. **Keep It Simple**
   - Don't use patterns for non-existent problems
   - Add patterns only when needed

3. **Document Patterns Used**
   - Clearly state which pattern is being used
   - Help team members understand code structure

4. **Test Thoroughly**
   - Each pattern change should be tested
   - Ensure no unintended side effects

5. **Maintain Consistency**
   - Use same patterns across project
   - Consistency aids understanding

### Code Quality Metrics

- **Cohesion**: Related code should be together
- **Coupling**: Dependencies should be minimal
- **Complexity**: Each class should have single responsibility
- **Duplication**: Don't repeat similar code patterns

---

## Learning Path

### Beginner Level
1. Start with Singleton and Factory
2. Understand basic composition with Adapter
3. Add Decorator for flexibility

### Intermediate Level
1. Master Strategy for algorithm variations
2. Observer for event-driven architecture
3. Command for undo/redo systems

### Advanced Level
1. Template Method for complex hierarchies
2. Proxy variations for special access patterns
3. Facade design for subsystem integration
4. Pattern combinations for complex scenarios

---

## Additional Resources

### Key Takeaways

| Pattern | Uses | Complexity |
|---------|------|-----------|
| Singleton | Single instance | Low |
| Factory | Object creation | Low-Medium |
| Builder | Complex objects | Medium |
| Prototype | Object copying | Low |
| Adapter | Interface conversion | Low-Medium |
| Decorator | Feature addition | Medium |
| Facade | System simplification | Medium |
| Proxy | Access control | Medium |
| Observer | Event notification | Medium-High |
| Strategy | Algorithm selection | Medium |
| Command | Request encapsulation | Medium |
| Template Method | Algorithm skeleton | Medium |

---

## Conclusion

Design patterns are powerful tools that help create maintainable, scalable, and flexible code. Understanding when and how to apply them is crucial for good software design. Start with simpler patterns and gradually incorporate more complex ones as your needs grow.

**Remember**: Patterns are guides, not rules. Use them wisely and adapt them to your specific needs.

---

## Quick Navigation

### Creational Patterns
- [Singleton](1_Creational/01_Singleton_Pattern.md)
- [Factory](1_Creational/02_Factory_Pattern.md)
- [Builder](1_Creational/03_Builder_Pattern.md)
- [Prototype](1_Creational/04_Prototype_Pattern.md)

### Structural Patterns
- [Adapter](2_Structural/01_Adapter_Pattern.md)
- [Decorator](2_Structural/02_Decorator_Pattern.md)
- [Facade](2_Structural/03_Facade_Pattern.md)
- [Proxy](2_Structural/04_Proxy_Pattern.md)

### Behavioral Patterns
- [Observer](3_Behavioral/01_Observer_Pattern.md)
- [Strategy](3_Behavioral/02_Strategy_Pattern.md)
- [Command](3_Behavioral/03_Command_Pattern.md)
- [Template Method](3_Behavioral/04_Template_Method_Pattern.md)
