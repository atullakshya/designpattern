# Decorator Pattern

## Definition
The Decorator Pattern attaches **additional responsibilities** to an object dynamically, providing a flexible alternative to subclassing. It wraps an object with another object that adds new functionality while maintaining the same interface.

## Core Concept
- **Dynamic Properties**: Add behavior at runtime, not compile time
- **Same Interface**: Decorated object is indistinguishable from original
- **Composition Over Inheritance**: Avoids explosion of subclasses
- **Flexible Combinations**: Stack multiple decorators
- **Single Responsibility**: Each decorator adds one feature

## Problem It Solves
- Explosion of subclasses from combining features (e.g., PlainCoffee, CoffeeWithMilk, CoffeeWithMilkAndSugar, etc.)
- Need to add features dynamically without modifying original class
- Features should be independent and combinable
- Want to avoid inheritance hierarchies for optional features

## Generic Implementation

### Basic Decorator Pattern

```csharp
// Component Interface
public interface IComponent
{
    string GetDescription();
    double GetCost();
}

// Concrete Component
public class PlainCoffee : IComponent
{
    public string GetDescription() => "Plain Coffee";
    public double GetCost() => 2.00;
}

// Abstract Decorator
public abstract class CoffeeDecorator : IComponent
{
    protected IComponent _coffee;

    public CoffeeDecorator(IComponent coffee)
    {
        _coffee = coffee;
    }

    public virtual string GetDescription() => _coffee.GetDescription();
    public virtual double GetCost() => _coffee.GetCost();
}

// Concrete Decorators
public class MilkDecorator : CoffeeDecorator
{
    public MilkDecorator(IComponent coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Milk";
    public override double GetCost() => _coffee.GetCost() + 0.50;
}

public class SugarDecorator : CoffeeDecorator
{
    public SugarDecorator(IComponent coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Sugar";
    public override double GetCost() => _coffee.GetCost() + 0.25;
}

public class WhippedCreamDecorator : CoffeeDecorator
{
    public WhippedCreamDecorator(IComponent coffee) : base(coffee) { }

    public override string GetDescription() => $"{_coffee.GetDescription()}, Whipped Cream";
    public override double GetCost() => _coffee.GetCost() + 1.00;
}

// Usage
IComponent coffee = new PlainCoffee();
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");
// Output: Plain Coffee = $2

coffee = new MilkDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");
// Output: Plain Coffee, Milk = $2.50

coffee = new SugarDecorator(coffee);
coffee = new WhippedCreamDecorator(coffee);
Console.WriteLine($"{coffee.GetDescription()} = ${coffee.GetCost()}");
// Output: Plain Coffee, Milk, Sugar, Whipped Cream = $3.75
```

### Generic Decorator

```csharp
// Generic version for reusability
public abstract class Decorator<T> : T where T : class
{
    protected T _component;

    public Decorator(T component)
    {
        _component = component;
    }
}

// Applied to any interface
public interface IDataStream
{
    string ReadData();
    void WriteData(string data);
}

public class FileDataStream : IDataStream
{
    private string _data = "Original data";

    public string ReadData() => _data;
    public void WriteData(string data) => _data = data;
}

public class EncryptionDecorator : IDataStream
{
    private readonly IDataStream _stream;

    public EncryptionDecorator(IDataStream stream) => _stream = stream;

    public string ReadData()
    {
        string data = _stream.ReadData();
        return Decrypt(data);
    }

    public void WriteData(string data)
    {
        string encrypted = Encrypt(data);
        _stream.WriteData(encrypted);
    }

    private string Encrypt(string data) => Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(data));
    private string Decrypt(string data) => System.Text.Encoding.UTF8.GetString(Convert.FromBase64String(data));
}

// Usage
IDataStream stream = new FileDataStream();
stream = new EncryptionDecorator(stream);
stream.WriteData("Sensitive Information");
Console.WriteLine(stream.ReadData());
```

## Real-World Examples

### UI Component Decorator
```csharp
public interface IWindow
{
    void Draw();
    string GetDescription();
}

public class SimpleWindow : IWindow
{
    public void Draw() => Console.WriteLine("Drawing simple window");
    public string GetDescription() => "Simple window";
}

public class BorderDecorator : IWindow
{
    private IWindow _window;

    public BorderDecorator(IWindow window) => _window = window;

    public void Draw()
    {
        _window.Draw();
        DrawBorder();
    }

    public string GetDescription() => $"{_window.GetDescription()} with border";

    private void DrawBorder() => Console.WriteLine("Drawing border");
}

public class ScrollDecorator : IWindow
{
    private IWindow _window;

    public ScrollDecorator(IWindow window) => _window = window;

    public void Draw()
    {
        _window.Draw();
        DrawScrollbars();
    }

    public string GetDescription() => $"{_window.GetDescription()} with scrollbars";

    private void DrawScrollbars() => Console.WriteLine("Drawing scrollbars");
}

// Usage
IWindow window = new SimpleWindow();
window = new BorderDecorator(window);
window = new ScrollDecorator(window);
window.Draw();
Console.WriteLine(window.GetDescription());
```

### Stream Decorator
```csharp
public interface IDataSource
{
    string Read();
}

public class FileDataSource : IDataSource
{
    public string Read() => "Data from file";
}

public class CompressionDecorator : IDataSource
{
    private IDataSource _source;

    public CompressionDecorator(IDataSource source) => _source = source;

    public string Read()
    {
        string data = _source.Read();
        return Compress(data);
    }

    private string Compress(string data) => $"[Compressed] {data}";
}

public class LoggingDecorator : IDataSource
{
    private IDataSource _source;

    public LoggingDecorator(IDataSource source) => _source = source;

    public string Read()
    {
        Console.WriteLine("Reading data...");
        string data = _source.Read();
        Console.WriteLine($"Data read: {data}");
        return data;
    }
}

// Usage
IDataSource source = new FileDataSource();
source = new CompressionDecorator(source);
source = new LoggingDecorator(source);
Console.WriteLine(source.Read());
```

### Request Handler Decorator
```csharp
public interface IRequestHandler
{
    void Handle(string request);
}

public class BasicRequestHandler : IRequestHandler
{
    public void Handle(string request)
    {
        Console.WriteLine($"Processing request: {request}");
    }
}

public class LoggingDecorator : IRequestHandler
{
    private IRequestHandler _handler;

    public LoggingDecorator(IRequestHandler handler) => _handler = handler;

    public void Handle(string request)
    {
        Console.WriteLine($"[LOG] Request received: {request}");
        _handler.Handle(request);
        Console.WriteLine($"[LOG] Request completed: {request}");
    }
}

public class AuthenticationDecorator : IRequestHandler
{
    private IRequestHandler _handler;

    public AuthenticationDecorator(IRequestHandler handler) => _handler = handler;

    public void Handle(string request)
    {
        if (!IsAuthenticated())
        {
            Console.WriteLine("[AUTH] Access denied");
            return;
        }
        Console.WriteLine("[AUTH] Access granted");
        _handler.Handle(request);
    }

    private bool IsAuthenticated() => true;
}

public class CachingDecorator : IRequestHandler
{
    private IRequestHandler _handler;
    private Dictionary<string, string> _cache = new();

    public CachingDecorator(IRequestHandler handler) => _handler = handler;

    public void Handle(string request)
    {
        if (_cache.ContainsKey(request))
        {
            Console.WriteLine("[CACHE] Returning cached result");
            return;
        }
        _handler.Handle(request);
        _cache[request] = "result";
    }
}

// Usage
IRequestHandler handler = new BasicRequestHandler();
handler = new LoggingDecorator(handler);
handler = new AuthenticationDecorator(handler);
handler = new CachingDecorator(handler);
handler.Handle("GetUser(123)");
```

## Decorator vs Inheritance

| Aspect | Decorator | Inheritance |
|--------|---|---|
| **At Runtime** | Can add/remove features | Fixed at compile time |
| **Combinations** | Any combination | Creates explosion of subclasses |
| **Complexity** | Linear | Exponential with features |
| **Single Responsibility** | Each decorator is focused | May violate SRP |
| **Code Reuse** | High | Can be low |

## Advantages

✅ **Flexibility**: Add functionality at runtime  
✅ **No Subclass Explosion**: Avoid numerous subclass combinations  
✅ **Single Responsibility**: Each decorator handles one feature  
✅ **Composable**: Stack decorators in any order  
✅ **Transparent**: Maintains original interface  

## Disadvantages

❌ **Complexity**: More objects to manage  
❌ **Order Matters**: Decorators may interact based on order  
❌ **Performance**: Multiple object layers can add overhead  
❌ **Debugging**: Harder to trace through decorator layers  

## When to Use

- Adding features to objects dynamically
- Avoiding large subclass hierarchies
- Combining features in multiple ways
- Want to add responsibilities without modifying original class
- Features may be applied in different orders

## When NOT to Use

- Simple inheritance is sufficient
- Decorators would be too complex
- Performance is critical
- Feature combination order is fixed

## Best Practices

1. **Keep Decorators Independent**: Each should handle one responsibility
2. **Maintain Same Interface**: Decorated object should look like original
3. **Document Decorator Order**: Some decorators may depend on others
4. **Use Type Checking Carefully**: Avoid relying on specific decorator types
5. **Consider Component Caching**: Store references to unwrapped objects if needed

## Summary

The Decorator Pattern is perfect for dynamically adding features without subclass explosion. Use it when you need flexible, composable behavior combinations. Each decorator wraps a component and adds one responsibility while maintaining the original interface.
