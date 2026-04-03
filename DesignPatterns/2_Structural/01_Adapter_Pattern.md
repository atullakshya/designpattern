# Adapter Pattern

## Definition
The Adapter Pattern converts the interface of a class into a **different interface** that clients expect. It allows incompatible interfaces to work together by acting as a "translator" between them.

## Core Concept
- **Interface Translation**: Convert one interface to another
- **Compatibility Layer**: Bridge between incompatible systems
- **Legacy Integration**: Make old code work with new code
- **Unchanged Original**: Adapts without modifying original classes

## Problem It Solves
- Third-party libraries with incompatible interfaces
- Legacy code integration with modern systems
- Different naming conventions or method signatures
- Classes that would otherwise need modification

## Generic Implementation

### Class Adapter (Using Inheritance)

```csharp
// Legacy Interface (Old System)
public interface ILegacyPaymentGateway
{
    void ProcessLegacyPayment(double amount, string accountNumber);
}

public class LegacyPaymentGateway : ILegacyPaymentGateway
{
    public void ProcessLegacyPayment(double amount, string accountNumber)
    {
        Console.WriteLine($"Legacy payment: ${amount} from account {accountNumber}");
    }
}

// Modern Interface (Expected by new code)
public interface IModernPaymentProcessor
{
    void ProcessPayment(decimal amount, string paymentMethod);
}

// Class Adapter (Inherits from legacy, implements modern)
public class PaymentAdapter : LegacyPaymentGateway, IModernPaymentProcessor
{
    public void ProcessPayment(decimal amount, string paymentMethod)
    {
        // Translate modern interface to legacy
        double legacyAmount = (double)amount;
        string accountNumber = DeriveAccountNumber(paymentMethod);
        
        // Call legacy method
        this.ProcessLegacyPayment(legacyAmount, accountNumber);
    }

    private string DeriveAccountNumber(string paymentMethod)
    {
        return paymentMethod switch
        {
            "creditcard" => "4532-1111-2222-3333",
            "bankaccount" => "9876-5432-1098-7654",
            _ => "0000-0000-0000-0000"
        };
    }
}

// Usage
IModernPaymentProcessor processor = new PaymentAdapter();
processor.ProcessPayment(99.99m, "creditcard");
```

### Object Adapter (Using Composition)

```csharp
// Legacy sensor interface
public interface IOldTemperatureSensor
{
    double GetTemperatureInFahrenheit();
}

public class OldTemperatureSensor : IOldTemperatureSensor
{
    public double GetTemperatureInFahrenheit()
    {
        // Returns temperature in Fahrenheit
        return 72.5;
    }
}

// Modern sensor interface (expecting Celsius)
public interface ITemperatureSensor
{
    double GetTemperatureInCelsius();
}

// Object Adapter (Composes the old sensor)
public class TemperatureSensorAdapter : ITemperatureSensor
{
    private readonly IOldTemperatureSensor _oldSensor;

    public TemperatureSensorAdapter(IOldTemperatureSensor oldSensor)
    {
        _oldSensor = oldSensor;
    }

    public double GetTemperatureInCelsius()
    {
        double fahrenheit = _oldSensor.GetTemperatureInFahrenheit();
        // Convert Fahrenheit to Celsius: (F - 32) × 5/9
        return (fahrenheit - 32) * 5.0 / 9.0;
    }
}

// Usage
IOldTemperatureSensor oldSensor = new OldTemperatureSensor();
ITemperatureSensor adapter = new TemperatureSensorAdapter(oldSensor);
Console.WriteLine($"Temperature: {adapter.GetTemperatureInCelsius()}°C");
```

### Two-Way Adapter

```csharp
// Client A Interface
public interface ISystemA
{
    void OperationA(string input);
}

// Client B Interface
public interface ISystemB
{
    void OperationB(string input);
}

public class SystemAImpl : ISystemA
{
    public void OperationA(string input)
    {
        Console.WriteLine($"System A executing: {input}");
    }
}

public class SystemBImpl : ISystemB
{
    public void OperationB(string input)
    {
        Console.WriteLine($"System B executing: {input}");
    }
}

// Two-way adapter
public class TwoWayAdapter : ISystemA, ISystemB
{
    private ISystemA _systemA;
    private ISystemB _systemB;

    public TwoWayAdapter(ISystemA systemA, ISystemB systemB)
    {
        _systemA = systemA;
        _systemB = systemB;
    }

    public void OperationA(string input)
    {
        _systemA.OperationA($"Adapted: {input}");
    }

    public void OperationB(string input)
    {
        _systemB.OperationB($"Adapted: {input}");
    }
}

// Usage
var adapted = new TwoWayAdapter(new SystemAImpl(), new SystemBImpl());
adapted.OperationA("Task 1");
adapted.OperationB("Task 2");
```

## Real-World Examples

### XML to JSON Adapter
```csharp
// Legacy XML Service
public interface IXMLService
{
    string GetDataAsXML();
}

public class LegacyXMLService : IXMLService
{
    public string GetDataAsXML()
    {
        return "<user><name>John</name><email>john@example.com</email></user>";
    }
}

// Modern JSON Interface
public interface IJSONService
{
    string GetDataAsJSON();
}

// Adapter
public class XMLToJSONAdapter : IJSONService
{
    private readonly IXMLService _xmlService;

    public XMLToJSONAdapter(IXMLService xmlService)
    {
        _xmlService = xmlService;
    }

    public string GetDataAsJSON()
    {
        string xmlData = _xmlService.GetDataAsXML();
        // Simplified XML to JSON conversion
        return ConvertXMLToJSON(xmlData);
    }

    private string ConvertXMLToJSON(string xmlData)
    {
        // In real scenario, use proper XML parsing and JSON serialization
        string json = "{\"name\": \"John\", \"email\": \"john@example.com\"}";
        return json;
    }
}

// Usage
IXMLService xmlService = new LegacyXMLService();
IJSONService jsonService = new XMLToJSONAdapter(xmlService);
Console.WriteLine(jsonService.GetDataAsJSON());
```

### Database Interface Adapter
```csharp
// Old Database Interface
public interface IOldDatabase
{
    List<Dictionary<string, object>> Query(string sql);
    void Insert(string table, Dictionary<string, object> data);
}

// New ORM Interface
public interface IRepository<T>
{
    IEnumerable<T> GetAll();
    void Add(T entity);
    T GetById(int id);
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// Adapter
public class OldDatabaseToRepositoryAdapter : IRepository<User>
{
    private readonly IOldDatabase _oldDb;

    public OldDatabaseToRepositoryAdapter(IOldDatabase oldDb)
    {
        _oldDb = oldDb;
    }

    public IEnumerable<User> GetAll()
    {
        var results = _oldDb.Query("SELECT * FROM Users");
        return results.Select(row => new User
        {
            Id = (int)row["Id"],
            Name = (string)row["Name"],
            Email = (string)row["Email"]
        });
    }

    public void Add(User entity)
    {
        var data = new Dictionary<string, object>
        {
            { "Name", entity.Name },
            { "Email", entity.Email }
        };
        _oldDb.Insert("Users", data);
    }

    public User GetById(int id)
    {
        var results = _oldDb.Query($"SELECT * FROM Users WHERE Id = {id}");
        if (results.Count == 0) return null;

        var row = results[0];
        return new User
        {
            Id = (int)row["Id"],
            Name = (string)row["Name"],
            Email = (string)row["Email"]
        };
    }
}

// Usage
IOldDatabase oldDb = new LegacyDatabaseImpl();
IRepository<User> userRepo = new OldDatabaseToRepositoryAdapter(oldDb);
var users = userRepo.GetAll();
```

### Third-Party Library Adapter
```csharp
// External library interface (can't modify)
public class ThirdPartyLogger
{
    public void LogMessage(int severity, string message)
    {
        Console.WriteLine($"[{severity}] {message}");
    }
}

// Application expects this interface
public interface ILogger
{
    void LogInfo(string message);
    void LogWarning(string message);
    void LogError(string message);
}

// Adapter
public class LoggerAdapter : ILogger
{
    private readonly ThirdPartyLogger _logger;

    public LoggerAdapter(ThirdPartyLogger logger)
    {
        _logger = logger;
    }

    public void LogInfo(string message)
    {
        _logger.LogMessage(1, $"INFO: {message}");
    }

    public void LogWarning(string message)
    {
        _logger.LogMessage(2, $"WARNING: {message}");
    }

    public void LogError(string message)
    {
        _logger.LogMessage(3, $"ERROR: {message}");
    }
}

// Usage
var thirdPartyLogger = new ThirdPartyLogger();
ILogger logger = new LoggerAdapter(thirdPartyLogger);
logger.LogInfo("Application started");
logger.LogWarning("Low memory");
logger.LogError("Failed to connect");
```

## Class Adapter vs Object Adapter

| Aspect | Class Adapter | Object Adapter |
|--------|---|---|
| **Implementation** | Inheritance | Composition |
| **Flexibility** | Less flexible | More flexible |
| **Reusable** | Single inheritance limit | Can adapt multiple classes |
| **Method Overrides** | Can override methods | Delegates calls |
| **Dependencies** | Tighter coupling | Looser coupling |
| **Complexity** | Simpler to implement | Slightly more complex |

## Advantages

✅ **Integration**: Makes incompatible interfaces work together  
✅ **No Modification**: Doesn't require changes to original classes  
✅ **Reusable**: Adapter can work with multiple implementations  
✅ **Single Responsibility**: Separation of concerns  
✅ **Flexible**: Object adapter can adapt at runtime  

## Disadvantages

❌ **Extra Class**: Adds another layer of abstraction  
❌ **Performance**: Small overhead from translation  
❌ **Complexity**: Can hide incompatibilities instead of fixing them  
❌ **Misuse**: Shouldn't be used to hide poor design  

## When to Use

- Integrating third-party libraries
- Legacy code integration
- Multiple interfaces need to work together
- Converting data formats (XML to JSON, Celsius to Fahrenheit)
- Creating uniform interfaces from diverse sources

## When NOT to Use

- When modifying the original class is better
- For minor interface differences
- When the adaptation is too complex
- To hide design flaws (fix the real issue)

## Summary

The Adapter Pattern is essential for working with incompatible interfaces, whether from third-party libraries, legacy code, or different systems. Use Object Adapter for flexibility and composition, Class Adapter when inheritance makes sense.
