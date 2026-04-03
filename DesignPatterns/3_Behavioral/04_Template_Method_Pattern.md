# Template Method Pattern

## Definition
The Template Method Pattern defines the **skeleton of an algorithm** in the base class, but lets **subclasses override specific steps** without changing the algorithm's structure. It follows the Hollywood Principle: "Don't call us, we'll call you."

## Core Concept
- **Algorithm Skeleton**: Define algorithm structure in base class
- **Flexible Steps**: Subclasses override specific steps
- **Invariant Structure**: Overall structure cannot be changed
- **Code Reuse**: Common logic in base class
- **Inversion of Control**: Base class calls subclass methods

## Problem It Solves
- Multiple classes have similar algorithms with minor variations
- Code duplication across similar classes
- Changing algorithm structure affects all subclasses
- Enforcing algorithm steps in specific order
- Avoiding complete override of similar methods

## Generic Implementation

### Basic Template Method Pattern

```csharp
// Abstract Base Class with Template Method
public abstract class DataProcessor
{
    // Template method - defines algorithm skeleton
    public void ProcessData(List<string> data)
    {
        Console.WriteLine("=== Processing Started ===");
        
        ValidateData(data);
        TransformData(data);
        SaveData(data);
        
        Console.WriteLine("=== Processing Completed ===\n");
    }

    // Common step (not overridden)
    private void ValidateData(List<string> data)
    {
        Console.WriteLine("Validating data...");
        if (data == null || data.Count == 0)
            throw new ArgumentException("No data to process");
        Console.WriteLine("Data validated");
    }

    // Steps to be overridden by subclasses (abstract methods)
    protected abstract void TransformData(List<string> data);
    protected abstract void SaveData(List<string> data);
}

// Concrete Implementations
public class CsvDataProcessor : DataProcessor
{
    protected override void TransformData(List<string> data)
    {
        Console.WriteLine("Transforming data to CSV format");
        foreach (var item in data)
            Console.WriteLine($"  CSV: {item}");
    }

    protected override void SaveData(List<string> data)
    {
        Console.WriteLine("Saving data to CSV file");
    }
}

public class JsonDataProcessor : DataProcessor
{
    protected override void TransformData(List<string> data)
    {
        Console.WriteLine("Transforming data to JSON format");
        Console.WriteLine("  [");
        foreach (var item in data)
            Console.WriteLine($"    {{\"{item}\"}}");
        Console.WriteLine("  ]");
    }

    protected override void SaveData(List<string> data)
    {
        Console.WriteLine("Saving data to JSON file");
    }
}

public class XmlDataProcessor : DataProcessor
{
    protected override void TransformData(List<string> data)
    {
        Console.WriteLine("Transforming data to XML format");
        foreach (var item in data)
            Console.WriteLine($"  <item>{item}</item>");
    }

    protected override void SaveData(List<string> data)
    {
        Console.WriteLine("Saving data to XML file");
    }
}

// Usage
var data = new List<string> { "John", "Jane", "Bob" };

DataProcessor processor = new CsvDataProcessor();
processor.ProcessData(data);

processor = new JsonDataProcessor();
processor.ProcessData(data);

processor = new XmlDataProcessor();
processor.ProcessData(data);
```

### Hook Methods (Optional Overrides)

```csharp
// Template Method with optional hooks
public abstract class BeverageMaker
{
    // Template method
    public void MakeBeverage()
    {
        Console.WriteLine("Making beverage...");
        BoilWater();
        Brew();
        
        // Hook - optional override
        if (WantCondiments())
        {
            AddCondiments();
        }
        
        Serve();
    }

    private void BoilWater()
    {
        Console.WriteLine("Boiling water");
    }

    protected abstract void Brew();
    protected abstract void AddCondiments();

    protected virtual bool WantCondiments()
    {
        return true; // Default true, can be overridden
    }

    private void Serve()
    {
        Console.WriteLine("Serving beverage\n");
    }
}

public class CoffeeMaker : BeverageMaker
{
    protected override void Brew()
    {
        Console.WriteLine("Brewing coffee");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("Adding sugar and milk");
    }

    protected override bool WantCondiments()
    {
        return true; // Always add condiments for coffee
    }
}

public class TeaMaker : BeverageMaker
{
    protected override void Brew()
    {
        Console.WriteLine("Steeping tea");
    }

    protected override void AddCondiments()
    {
        Console.WriteLine("Adding honey");
    }

    protected override bool WantCondiments()
    {
        return false; // Tea without condiments
    }
}

// Usage
BeverageMaker maker = new CoffeeMaker();
maker.MakeBeverage();

maker = new TeaMaker();
maker.MakeBeverage();
```

## Real-World Examples

### Document Generation

```csharp
public abstract class DocumentGenerator
{
    public void GenerateDocument()
    {
        OpenDocument();
        AddHeader();
        AddContent();
        AddFooter();
        CloseDocument();
        Console.WriteLine();
    }

    private void OpenDocument()
    {
        Console.WriteLine("Opening document");
    }

    protected abstract void AddHeader();
    protected abstract void AddContent();
    protected abstract void AddFooter();

    private void CloseDocument()
    {
        Console.WriteLine("Closing document");
    }
}

public class PdfDocumentGenerator : DocumentGenerator
{
    protected override void AddHeader()
    {
        Console.WriteLine("Adding PDF header with metadata");
    }

    protected override void AddContent()
    {
        Console.WriteLine("Adding PDF content with formatting");
    }

    protected override void AddFooter()
    {
        Console.WriteLine("Adding PDF footer with page numbers");
    }
}

public class WordDocumentGenerator : DocumentGenerator
{
    protected override void AddHeader()
    {
        Console.WriteLine("Adding Word header");
    }

    protected override void AddContent()
    {
        Console.WriteLine("Adding Word content");
    }

    protected override void AddFooter()
    {
        Console.WriteLine("Adding Word footer");
    }
}

// Usage
DocumentGenerator generator = new PdfDocumentGenerator();
generator.GenerateDocument();

generator = new WordDocumentGenerator();
generator.GenerateDocument();
```

### Order Processing

```csharp
public abstract class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        Console.WriteLine($"Processing order #{order.OrderId}");
        ValidateOrder(order);
        ProcessPayment(order);
        PrepareShipment(order);
        SendNotification(order);
    }

    protected abstract void ValidateOrder(Order order);
    protected abstract void ProcessPayment(Order order);
    protected abstract void PrepareShipment(Order order);
    protected abstract void SendNotification(Order order);
}

public class OnlineOrderProcessor : OrderProcessor
{
    protected override void ValidateOrder(Order order)
    {
        Console.WriteLine("Validating online order inventory");
    }

    protected override void ProcessPayment(Order order)
    {
        Console.WriteLine("Processing credit card payment");
    }

    protected override void PrepareShipment(Order order)
    {
        Console.WriteLine("Preparing package for shipping");
    }

    protected override void SendNotification(Order order)
    {
        Console.WriteLine("Sending email notification");
    }
}

public class InStoreOrderProcessor : OrderProcessor
{
    protected override void ValidateOrder(Order order)
    {
        Console.WriteLine("Validating in-store order inventory");
    }

    protected override void ProcessPayment(Order order)
    {
        Console.WriteLine("Processing cash or card payment");
    }

    protected override void PrepareShipment(Order order)
    {
        Console.WriteLine("Preparing order for pickup");
    }

    protected override void SendNotification(Order order)
    {
        Console.WriteLine("Sending SMS notification");
    }
}

public class Order
{
    public int OrderId { get; set; }
    public decimal Total { get; set; }
}

// Usage
var order = new Order { OrderId = 123, Total = 99.99m };

OrderProcessor processor = new OnlineOrderProcessor();
processor.ProcessOrder(order);

processor = new InStoreOrderProcessor();
processor.ProcessOrder(order);
```

### Game Level Processing

```csharp
public abstract class GameLevel
{
    public void Play()
    {
        Console.WriteLine("Level started");
        InitializeLevel();
        LoadAssets();
        StartGameplay();
        HandleGameCompletion();
    }

    protected abstract void InitializeLevel();
    protected abstract void LoadAssets();
    protected abstract void StartGameplay();
    protected abstract void HandleGameCompletion();
}

public class EasyLevel : GameLevel
{
    protected override void InitializeLevel()
    {
        Console.WriteLine("Setting up easy level");
    }

    protected override void LoadAssets()
    {
        Console.WriteLine("Loading easy level assets");
    }

    protected override void StartGameplay()
    {
        Console.WriteLine("Starting easy gameplay (slow enemies)");
    }

    protected override void HandleGameCompletion()
    {
        Console.WriteLine("Easy level completed! Unlocking medium level");
    }
}

public class HardLevel : GameLevel
{
    protected override void InitializeLevel()
    {
        Console.WriteLine("Setting up hard level");
    }

    protected override void LoadAssets()
    {
        Console.WriteLine("Loading hard level assets with HD graphics");
    }

    protected override void StartGameplay()
    {
        Console.println("Starting hard gameplay (fast enemies)");
    }

    protected override void HandleGameCompletion()
    {
        Console.WriteLine("Hard level completed! You are a master!");
    }
}

// Usage
GameLevel level = new EasyLevel();
level.Play();

level = new HardLevel();
level.Play();
```

### Data Migration

```csharp
public abstract class DataMigration
{
    public void Migrate(string source, string destination)
    {
        Console.WriteLine("Starting migration");
        BackupData(source);
        ExtractData(source);
        TransformData();
        LoadData(destination);
        VerifyIntegrity();
        CleanupTemporaryFiles();
    }

    protected abstract void ExtractData(string source);
    protected abstract void TransformData();
    protected abstract void LoadData(string destination);

    protected virtual void BackupData(string source)
    {
        Console.WriteLine($"Backing up data from {source}");
    }

    protected virtual void VerifyIntegrity()
    {
        Console.WriteLine("Verifying data integrity");
    }

    protected virtual void CleanupTemporaryFiles()
    {
        Console.WriteLine("Cleaning up temporary files");
    }
}

public class DatabaseMigration : DataMigration
{
    protected override void ExtractData(string source)
    {
        Console.WriteLine($"Extracting data from database {source}");
    }

    protected override void TransformData()
    {
        Console.WriteLine("Transforming schema and data structures");
    }

    protected override void LoadData(string destination)
    {
        Console.WriteLine($"Loading data to target database {destination}");
    }
}

// Usage
var migration = new DatabaseMigration();
migration.Migrate("SourceDB", "TargetDB");
```

## Template Method vs Strategy

| Aspect | Template Method | Strategy |
|--------|---|---|
| **Definition** | Algorithm skeleton in base class | Complete algorithm in strategy |
| **Inheritance** | Uses inheritance | Uses composition |
| **Structure** | Fixed structure | Completely interchangeable |
| **Runtime Change** | Not typically | Easily changed at runtime |
| **Flexibility** | Subclass override specific steps | Entire algorithm replaced |

## Advantages

✅ **Code Reuse**: Common logic in base class  
✅ **Enforced Structure**: Cannot change algorithm skeleton  
✅ **Flexibility**: Subclasses override specific steps  
✅ **Single Responsibility**: Each subclass handles one variant  
✅ **Easy Maintenance**: Common changes in one place  

## Disadvantages

❌ **Inheritance Required**: Need class hierarchy  
❌ **Runtime Inflexibility**: Can't change algorithm easily  
❌ **Inversion of Control**: Can be confusing (Hollywood Principle)  
❌ **Limited Flexibility**: Fixed skeleton can be restrictive  

## When to Use

- Multiple classes with similar algorithms
- Want to avoid code duplication
- Algorithm structure should remain constant
- Variations only in specific steps
- Want to enforce implementation order

## When NOT to Use

- Algorithms completely different
- Need to switch algorithms at runtime
- Algorithm is simple
- Composition would be clearer

## Best Practices

1. **Keep Base Class API Stable**: Don't add new methods often
2. **Document Hook Methods**: Clarify what can be overridden
3. **Use Protected for Overridable Methods**: Show intent
4. **Single Responsibility**: Each step should handle one task
5. **Reasonable Defaults**: Provide sensible default hooks
6. **Avoid Over-Engineering**: Use only if duplication exists

## Summary

The Template Method Pattern defines an algorithm's skeleton in the base class while allowing subclasses to override specific steps. It's perfect for scenarios where you have multiple variations of the same algorithm, promoting code reuse while maintaining a consistent structure.
