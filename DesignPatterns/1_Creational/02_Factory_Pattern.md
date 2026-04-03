# Factory Pattern

## Definition
The Factory Pattern defines an interface or abstract class for creating objects, but **delegates the instantiation logic** to the subclasses or a separate factory class. This hides complex object creation logic from the client.

## Core Concept
- **Encapsulation of Creation**: Object creation logic is separated from business logic
- **Loose Coupling**: Client code doesn't know the exact types being created
- **Flexibility**: Easy to add new types without modifying client code
- **Single Responsibility**: Creation logic in one place

## Problem It Solves
- Direct instantiation scatters object creation throughout the codebase
- Creating complex objects requires multiple steps
- Adding new types requires changes in many places
- Client code becomes tightly coupled to concrete classes

## Generic Implementation

### 1. Simple Factory (Not a GoF pattern, but commonly used)

```csharp
// Product Interface
public interface INotification
{
    void Send(string message);
}

// Concrete Products
public class EmailNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending email: {message}");
    }
}

public class SMSNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending SMS: {message}");
    }
}

public class PushNotification : INotification
{
    public void Send(string message)
    {
        Console.WriteLine($"Sending push notification: {message}");
    }
}

// Simple Factory Class
public class NotificationFactory
{
    public static INotification Create(string type)
    {
        return type switch
        {
            "email" => new EmailNotification(),
            "sms" => new SMSNotification(),
            "push" => new PushNotification(),
            _ => throw new ArgumentException($"Unknown notification type: {type}")
        };
    }
}

// Usage
var emailNotif = NotificationFactory.Create("email");
emailNotif.Send("Hello via Email");

var smsNotif = NotificationFactory.Create("sms");
smsNotif.Send("Hello via SMS");
```

### 2. Factory Method Pattern

```csharp
// Product Definition
public abstract class Transport
{
    public abstract void Deliver(string package);
}

// Concrete Products
public class Truck : Transport
{
    public override void Deliver(string package)
    {
        Console.WriteLine($"Delivering via truck: {package}");
    }
}

public class Ship : Transport
{
    public override void Deliver(string package)
    {
        Console.WriteLine($"Delivering via ship: {package}");
    }
}

public class Airplane : Transport
{
    public override void Deliver(string package)
    {
        Console.WriteLine($"Delivering via airplane: {package}");
    }
}

// Creator (Abstract Factory)
public abstract class Logistics
{
    // Factory Method - subclasses must implement this
    public abstract Transport CreateTransport();

    public void PlanDelivery(string package)
    {
        Transport transport = CreateTransport();
        transport.Deliver(package);
    }
}

// Concrete Creators
public class RoadLogistics : Logistics
{
    public override Transport CreateTransport() => new Truck();
}

public class SeaLogistics : Logistics
{
    public override Transport CreateTransport() => new Ship();
}

public class AirLogistics : Logistics
{
    public override Transport CreateTransport() => new Airplane();
}

// Usage
Logistics roadLogistics = new RoadLogistics();
roadLogistics.PlanDelivery("Books"); // Uses Truck

Logistics seaLogistics = new SeaLogistics();
seaLogistics.PlanDelivery("Furniture"); // Uses Ship
```

### 3. Abstract Factory Pattern

```csharp
// Abstract Product Families
public interface IButton
{
    void Render();
}

public interface ICheckbox
{
    void Render();
}

// Concrete Products - Windows Family
public class WindowsButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Windows Button");
}

public class WindowsCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Windows Checkbox");
}

// Concrete Products - Mac Family
public class MacButton : IButton
{
    public void Render() => Console.WriteLine("Rendering Mac Button");
}

public class MacCheckbox : ICheckbox
{
    public void Render() => Console.WriteLine("Rendering Mac Checkbox");
}

// Abstract Factory
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

// Concrete Factories
public class WindowsUIFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacUIFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}

// Usage
IUIFactory factory = GetFactory("Windows");
IButton button = factory.CreateButton();
ICheckbox checkbox = factory.CreateCheckbox();
button.Render();
checkbox.Render();

IUIFactory GetFactory(string osType) => osType switch
{
    "Windows" => new WindowsUIFactory(),
    "Mac" => new MacUIFactory(),
    _ => throw new ArgumentException("Unknown OS")
};
```

## Real-World Examples

### Database Connection Factory
```csharp
// Abstract Product
public interface IDatabase
{
    void Connect();
    void ExecuteQuery(string query);
}

// Concrete Products
public class MySqlDatabase : IDatabase
{
    public void Connect() => Console.WriteLine("Connecting to MySQL Server");
    public void ExecuteQuery(string query) => Console.WriteLine($"Executing MySQL query: {query}");
}

public class SqlServerDatabase : IDatabase
{
    public void Connect() => Console.WriteLine("Connecting to SQL Server");
    public void ExecuteQuery(string query) => Console.WriteLine($"Executing SQL Server query: {query}");
}

public class PostgreSqlDatabase : IDatabase
{
    public void Connect() => Console.WriteLine("Connecting to PostgreSQL");
    public void ExecuteQuery(string query) => Console.WriteLine($"Executing PostgreSQL query: {query}");
}

// Factory
public class DatabaseFactory
{
    public static IDatabase CreateDatabase(string databaseType)
    {
        return databaseType.ToLower() switch
        {
            "mysql" => new MySqlDatabase(),
            "sqlserver" => new SqlServerDatabase(),
            "postgresql" => new PostgreSqlDatabase(),
            _ => throw new ArgumentException($"Database type '{databaseType}' is not supported")
        };
    }
}

// Usage
var myDb = DatabaseFactory.CreateDatabase("mysql");
myDb.Connect();
myDb.ExecuteQuery("SELECT * FROM Users");
```

### Payment Processing Factory
```csharp
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
    void RefundPayment(decimal amount);
}

public class CreditCardPaymentProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount) 
        => Console.WriteLine($"Processing credit card payment: ${amount}");
    
    public void RefundPayment(decimal amount) 
        => Console.WriteLine($"Refunding credit card: ${amount}");
}

public class PayPalPaymentProcessor : IPaymentProcessor
{
    public void ProcessPayment(decimal amount) 
        => Console.WriteLine($"Processing PayPal payment: ${amount}");
    
    public void RefundPayment(decimal amount) 
        => Console.WriteLine($"Refunding PayPal: ${amount}");
}

public class PaymentFactory
{
    public static IPaymentProcessor CreatePaymentProcessor(string paymentMethod)
    {
        return paymentMethod.ToLower() switch
        {
            "creditcard" => new CreditCardPaymentProcessor(),
            "paypal" => new PayPalPaymentProcessor(),
            _ => throw new NotSupportedException()
        };
    }
}

// Usage
var processor = PaymentFactory.CreatePaymentProcessor("creditcard");
processor.ProcessPayment(99.99m);
```

## Comparison of Factory Types

| Factory Type | Use When | Complexity |
|---|---|---|
| **Simple Factory** | Limited number of types, simple logic | Low |
| **Factory Method** | Different subclasses create different objects | Medium |
| **Abstract Factory** | Need to create families of related objects | High |

## Advantages

✅ **Loose Coupling**: Client doesn't depend on concrete classes  
✅ **Single Responsibility**: Creation logic centralized  
✅ **Open/Closed Principle**: Add new types without modifying existing code  
✅ **Flexible**: Easy to switch implementations  
✅ **Maintainable**: Changes in creation logic happen in one place  

## Disadvantages

❌ **Over-Engineering**: Can add complexity for simple scenarios  
❌ **Extra Abstraction Layers**: More classes to understand  
❌ **Potential Performance Overhead**: Additional method calls  

## When to Use

- Creating objects of different types based on runtime conditions
- Complex object creation with multiple steps
- Need to decouple object creation from usage
- Multiple related families of objects (Abstract Factory)
- Changing implementations frequently

## When NOT to Use

- Very simple objects that don't need encapsulation
- Objects created in only one way
- When dependency injection would be cleaner

## Summary

The Factory Pattern provides a clean, extensible way to handle object creation. Choose the appropriate factory type based on complexity: Simple Factory for basic scenarios, Factory Method for inheritance hierarchies, and Abstract Factory for families of related objects.
