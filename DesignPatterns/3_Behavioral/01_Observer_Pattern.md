# Observer Pattern

## Definition
The Observer Pattern defines a **one-to-many dependency** between objects, where when one object (subject) changes state, all its dependents (observers) are notified automatically. Also known as Publish-Subscribe pattern.

## Core Concept
- **Loose Coupling**: Subject and observers don't need to know details about each other
- **Dynamic Subscription**: Observers can subscribe/unsubscribe at runtime
- **Automatic Notification**: Subject pushes updates to observers automatically
- **Many-to-One Relationship**: Multiple observers can depend on one subject
- **Separation of Concerns**: Subject and observers handle their own logic

## Problem It Solves
- Multiple objects need to react to state changes in another object
- Number of dependent objects is unknown or changes dynamically
- Don't want tight coupling between event source and handlers
- Need to broadcast changes to many listeners
- Observer addition/removal should be runtime flexible

## Generic Implementation

### Basic Observer Pattern

```csharp
// Observer Interface
public interface IObserver
{
    void Update(string message);
}

// Subject Interface
public interface ISubject
{
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}

// Concrete Subject
public class StockMarket : ISubject
{
    private List<IObserver> _observers = new();
    private string _symbol;
    private decimal _price;

    public StockMarket(string symbol)
    {
        _symbol = symbol;
    }

    public void Attach(IObserver observer)
    {
        if (!_observers.Contains(observer))
            _observers.Add(observer);
    }

    public void Detach(IObserver observer)
    {
        _observers.Remove(observer);
    }

    public void Notify()
    {
        foreach (var observer in _observers)
        {
            observer.Update($"{_symbol} price changed to ${_price}");
        }
    }

    public void SetPrice(decimal newPrice)
    {
        _price = newPrice;
        Notify();
    }
}

// Concrete Observers
public class Investor : IObserver
{
    private string _name;

    public Investor(string name) => _name = name;

    public void Update(string message)
    {
        Console.WriteLine($"[Investor {_name}] {message}");
    }
}

public class TradingBot : IObserver
{
    public void Update(string message)
    {
        Console.WriteLine($"[Trading Bot] Executing trade based on: {message}");
    }
}

// Usage
var market = new StockMarket("AAPL");
var investor1 = new Investor("John");
var investor2 = new Investor("Jane");
var bot = new TradingBot();

market.Attach(investor1);
market.Attach(investor2);
market.Attach(bot);

market.SetPrice(150.00m);
market.SetPrice(152.50m);

market.Detach(investor2);
market.SetPrice(148.00m);
```

### Generic Observer Pattern

```csharp
// Generic Observer Interface
public interface IObserver<T>
{
    void Update(T data);
}

// Generic Subject
public class Subject<T>
{
    private List<IObserver<T>> _observers = new();

    public void Attach(IObserver<T> observer)
    {
        _observers.Add(observer);
    }

    public void Detach(IObserver<T> observer)
    {
        _observers.Remove(observer);
    }

    protected void Notify(T data)
    {
        foreach (var observer in _observers)
        {
            observer.Update(data);
        }
    }
}

// Concrete Implementation
public class TemperatureSensor : Subject<double>
{
    private double _temperature;

    public double Temperature
    {
        get => _temperature;
        set
        {
            _temperature = value;
            Notify(_temperature);
        }
    }
}

public class TemperatureDisplay : IObserver<double>
{
    private string _location;

    public TemperatureDisplay(string location) => _location = location;

    public void Update(double fahrenheit)
    {
        double celsius = (fahrenheit - 32) * 5 / 9;
        Console.WriteLine($"[{_location}] Temperature: {fahrenheit}°F ({celsius}°C)");
    }
}

// Usage
var sensor = new TemperatureSensor();
var indoorDisplay = new TemperatureDisplay("Indoor");
var outdoorDisplay = new TemperatureDisplay("Outdoor");

sensor.Attach(indoorDisplay);
sensor.Attach(outdoorDisplay);

sensor.Temperature = 72.0;
sensor.Temperature = 75.5;
```

### Event-Based Observer (Modern C# Style)

```csharp
// Using C# events instead of explicit interfaces
public class Button
{
    // Define event signature
    public event EventHandler OnClick;

    public void Click()
    {
        Console.WriteLine("Button clicked");
        // Raise event
        OnClick?.Invoke(this, EventArgs.Empty);
    }
}

public class UI
{
    private Button _button = new();

    public UI()
    {
        // Subscribe handlers
        _button.OnClick += HandleButtonClick;
        _button.OnClick += LogButtonClick;
    }

    private void HandleButtonClick(object sender, EventArgs e)
    {
        Console.WriteLine("[UI] Button click handled");
    }

    private void LogButtonClick(object sender, EventArgs e)
    {
        Console.WriteLine($"[Logger] Button clicked at {DateTime.Now}");
    }

    public void SimulateClick()
    {
        _button.Click();
    }
}

// Usage
var ui = new UI();
ui.SimulateClick();
```

## Real-World Examples

### Notification System

```csharp
public interface INotificationObserver
{
    void OnUserCreated(User user);
    void OnUserDeleted(User user);
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserService
{
    private List<INotificationObserver> _observers = new();

    public void Subscribe(INotificationObserver observer)
    {
        _observers.Add(observer);
    }

    public void Unsubscribe(INotificationObserver observer)
    {
        _observers.Remove(observer);
    }

    private void NotifyUserCreated(User user)
    {
        foreach (var observer in _observers)
        {
            observer.OnUserCreated(user);
        }
    }

    private void NotifyUserDeleted(User user)
    {
        foreach (var observer in _observers)
        {
            observer.OnUserDeleted(user);
        }
    }

    public void CreateUser(User user)
    {
        Console.WriteLine($"Creating user: {user.Name}");
        NotifyUserCreated(user);
    }

    public void DeleteUser(User user)
    {
        Console.WriteLine($"Deleting user: {user.Name}");
        NotifyUserDeleted(user);
    }
}

public class EmailNotificationObserver : INotificationObserver
{
    public void OnUserCreated(User user)
    {
        Console.WriteLine($"[EMAIL] Welcome email sent to {user.Email}");
    }

    public void OnUserDeleted(User user)
    {
        Console.WriteLine($"[EMAIL] Goodbye email sent to {user.Email}");
    }
}

public class AuditLogObserver : INotificationObserver
{
    public void OnUserCreated(User user)
    {
        Console.WriteLine($"[AUDIT] User created: {user.Name} at {DateTime.Now}");
    }

    public void OnUserDeleted(User user)
    {
        Console.WriteLine($"[AUDIT] User deleted: {user.Name} at {DateTime.Now}");
    }
}

// Usage
var userService = new UserService();
userService.Subscribe(new EmailNotificationObserver());
userService.Subscribe(new AuditLogObserver());

var user = new User { Id = 1, Name = "John", Email = "john@example.com" };
userService.CreateUser(user);
userService.DeleteUser(user);
```

### Model-View Pattern

```csharp
public interface IView
{
    void UpdateDisplay(string data);
}

public class Model
{
    private List<IView> _views = new();
    private string _data;

    public void Attach(IView view)
    {
        _views.Add(view);
    }

    public void SetData(string data)
    {
        _data = data;
        NotifyViews();
    }

    public string GetData() => _data;

    private void NotifyViews()
    {
        foreach (var view in _views)
        {
            view.UpdateDisplay(_data);
        }
    }
}

public class ListView : IView
{
    public void UpdateDisplay(string data)
    {
        Console.WriteLine($"[ListView] {data}");
    }
}

public class TreeView : IView
{
    public void UpdateDisplay(string data)
    {
        Console.WriteLine($"[TreeView] {data}");
    }
}

// Usage
var model = new Model();
model.Attach(new ListView());
model.Attach(new TreeView());

model.SetData("Updated data");
```

### File System Watcher

```csharp
public interface IFileSystemObserver
{
    void OnFileCreated(string fileName);
    void OnFileDeleted(string fileName);
    void OnFileModified(string fileName);
}

public class FileSystem
{
    private List<IFileSystemObserver> _observers = new();

    public void Subscribe(IFileSystemObserver observer) => _observers.Add(observer);
    public void Unsubscribe(IFileSystemObserver observer) => _observers.Remove(observer);

    private void NotifyFileCreated(string fileName)
    {
        foreach (var observer in _observers)
            observer.OnFileCreated(fileName);
    }

    private void NotifyFileDeleted(string fileName)
    {
        foreach (var observer in _observers)
            observer.OnFileDeleted(fileName);
    }

    private void NotifyFileModified(string fileName)
    {
        foreach (var observer in _observers)
            observer.OnFileModified(fileName);
    }

    public void CreateFile(string fileName)
    {
        Console.WriteLine($"Creating file: {fileName}");
        NotifyFileCreated(fileName);
    }

    public void DeleteFile(string fileName)
    {
        Console.WriteLine($"Deleting file: {fileName}");
        NotifyFileDeleted(fileName);
    }

    public void ModifyFile(string fileName)
    {
        Console.WriteLine($"Modifying file: {fileName}");
        NotifyFileModified(fileName);
    }
}

public class Logger : IFileSystemObserver
{
    public void OnFileCreated(string fileName) => Console.WriteLine($"[LOG] File created: {fileName}");
    public void OnFileDeleted(string fileName) => Console.WriteLine($"[LOG] File deleted: {fileName}");
    public void OnFileModified(string fileName) => Console.WriteLine($"[LOG] File modified: {fileName}");
}

public class Indexer : IFileSystemObserver
{
    public void OnFileCreated(string fileName) => Console.WriteLine($"[INDEXER] Indexing new file: {fileName}");
    public void OnFileDeleted(string fileName) => Console.WriteLine($"[INDEXER] Removing from index: {fileName}");
    public void OnFileModified(string fileName) => Console.WriteLine($"[INDEXER] Reindexing: {fileName}");
}

// Usage
var fileSystem = new FileSystem();
fileSystem.Subscribe(new Logger());
fileSystem.Subscribe(new Indexer());

fileSystem.CreateFile("document.txt");
fileSystem.ModifyFile("document.txt");
fileSystem.DeleteFile("document.txt");
```

## Push vs Pull Model

| Model | Details |
|-------|---------|
| **Push** | Subject sends all data to observers (what we showed) |
| **Pull** | Subject notifies observers, they query for data |
| **Hybrid** | Subject sends message, observers request details |

## Advantages

✅ **Loose Coupling**: Subject and observers independent  
✅ **Dynamic Subscription**: Add/remove observers at runtime  
✅ **Broadcast Communication**: One-to-many communication  
✅ **Separation of Concerns**: Each observer handles its logic  
✅ **Reusable**: Observers can observe multiple subjects  

## Disadvantages

❌ **Unpredictable Order**: Observers notified in unpredictable order  
❌ **Memory Leaks**: Forgotten unsubscribe keeps observer references  
❌ **Performance**: Many observers = many notifications  
❌ **Debugging**: Hard to trace notification flow  

## When to Use

- Multiple objects need to react to state changes
- Number of observers unknown at design time
- Event-driven architecture
- Decoupling event sources from handlers
- Model-View frameworks

## When NOT to Use

- Simple one-to-one communication
- Notification order is critical
- Performance is critical with many observers
- Tight coupling is acceptable

## Best Practices

1. **Always Unsubscribe**: Prevent memory leaks
2. **Use Events in C#**: Modern approach is cleaner
3. **Consider Push vs Pull**: Choose appropriate data flow
4. **Document Notifications**: List all events a subject raises
5. **Handle Observer Exceptions**: Don't let one observer break others
6. **Avoid Circular Updates**: Observer shouldn't trigger subject update

## Summary

The Observer Pattern is fundamental for event-driven systems and decoupled architectures. It enables multiple objects to automatically react to changes in another object without tight coupling. Modern C# uses events for this, but the pattern principles remain the same.
