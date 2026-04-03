# Proxy Pattern

## Definition
The Proxy Pattern provides a **placeholder or surrogate** for another object, controlling access to it. It acts as an intermediary that can add behavior before/after delegating to the real subject.

## Core Concept
- **Lazy Instantiation**: Defer creating expensive objects until needed
- **Access Control**: Control who can access the real object
- **Caching**: Cache results to improve performance
- **Logging/Monitoring**: Track interactions with the real object
- **Same Interface**: Proxy looks like the real object to client

## Problem It Solves
- Creating expensive objects upfront wastes resources
- Need to control access to a sensitive object
- Want to add behavior without modifying the real object
- Need to defer object creation until actually needed
- Require network requests or remote objects

## Types of Proxies

### 1. Virtual Proxy (Lazy Loading)

```csharp
// Expensive resource to load
public interface IImage
{
    void Display();
}

public class RealImage : IImage
{
    private string _filename;

    public RealImage(string filename)
    {
        _filename = filename;
        LoadImage();
    }

    private void LoadImage()
    {
        Console.WriteLine($"Loading image: {_filename}");
        System.Threading.Thread.Sleep(1000); // Expensive operation
        Console.WriteLine($"Image loaded: {_filename}");
    }

    public void Display()
    {
        Console.WriteLine($"Displaying: {_filename}");
    }
}

// Virtual Proxy - Defers loading until needed
public class ImageProxy : IImage
{
    private string _filename;
    private RealImage _realImage;

    public ImageProxy(string filename)
    {
        _filename = filename;
        // Don't load yet
    }

    public void Display()
    {
        // Load only when needed
        if (_realImage == null)
        {
            _realImage = new RealImage(_filename);
        }
        _realImage.Display();
    }
}

// Usage
IImage image = new ImageProxy("large_photo.jpg");
// Image not loaded yet

// Image loaded only when Display() is called
image.Display();
```

### 2. Protection Proxy (Access Control)

```csharp
// Interface for document access
public interface IDocument
{
    string GetContent();
    void SetContent(string content);
    void Delete();
}

public class Document : IDocument
{
    private string _content;

    public Document(string content) => _content = content;

    public string GetContent() => _content;
    public void SetContent(string content) => _content = content;
    public void Delete() => Console.WriteLine("Document deleted");
}

// Protection Proxy - Controls access based on user role
public class DocumentProxy : IDocument
{
    private Document _document;
    private string _userRole;

    public DocumentProxy(Document document, string userRole)
    {
        _document = document;
        _userRole = userRole;
    }

    public string GetContent()
    {
        if (_userRole == "Admin" || _userRole == "Editor")
        {
            return _document.GetContent();
        }
        throw new UnauthorizedAccessException("You don't have permission to read");
    }

    public void SetContent(string content)
    {
        if (_userRole != "Admin" && _userRole != "Editor")
            throw new UnauthorizedAccessException("You don't have permission to edit");
        
        _document.SetContent(content);
        Console.WriteLine("Content updated");
    }

    public void Delete()
    {
        if (_userRole != "Admin")
            throw new UnauthorizedAccessException("Only admins can delete");
        
        _document.Delete();
    }
}

// Usage
var document = new Document("Secret information");

var adminProxy = new DocumentProxy(document, "Admin");
adminProxy.SetContent("Updated secret");
adminProxy.Delete();

var userProxy = new DocumentProxy(document, "User");
try
{
    userProxy.SetContent("Attempt to change");  // Throws exception
}
catch (UnauthorizedAccessException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
```

### 3. Logging Proxy (Monitoring)

```csharp
// Subject interface
public interface IDataRepository
{
    string GetData(int id);
    void SaveData(int id, string data);
}

public class DataRepository : IDataRepository
{
    public string GetData(int id) => $"Data for ID {id}";

    public void SaveData(int id, string data) => Console.WriteLine($"Saving data for ID {id}");
}

// Logging Proxy
public class LoggingDataRepositoryProxy : IDataRepository
{
    private DataRepository _repository;

    public LoggingDataRepositoryProxy(DataRepository repository)
    {
        _repository = repository;
    }

    public string GetData(int id)
    {
        Console.WriteLine($"[LOG] GetData called with id={id} at {DateTime.Now}");
        var result = _repository.GetData(id);
        Console.WriteLine($"[LOG] GetData returned: {result}");
        return result;
    }

    public void SaveData(int id, string data)
    {
        Console.WriteLine($"[LOG] SaveData called with id={id}, data={data} at {DateTime.Now}");
        _repository.SaveData(id, data);
        Console.WriteLine($"[LOG] SaveData completed");
    }
}

// Usage
IDataRepository repo = new LoggingDataRepositoryProxy(new DataRepository());
repo.GetData(1);
repo.SaveData(1, "New data");
```

### 4. Caching Proxy

```csharp
public interface IWeatherService
{
    string GetWeather(string city);
}

public class WeatherService : IWeatherService
{
    public string GetWeather(string city)
    {
        Console.WriteLine($"Fetching weather for {city} from API...");
        System.Threading.Thread.Sleep(500); // Simulates API call
        return $"Sunny, 25°C";
    }
}

// Caching Proxy
public class CachingWeatherServiceProxy : IWeatherService
{
    private WeatherService _service;
    private Dictionary<string, string> _cache = new();

    public CachingWeatherServiceProxy(WeatherService service)
    {
        _service = service;
    }

    public string GetWeather(string city)
    {
        if (_cache.ContainsKey(city))
        {
            Console.WriteLine($"[CACHE HIT] Returning cached weather for {city}");
            return _cache[city];
        }

        Console.WriteLine($"[CACHE MISS] Fetching fresh data for {city}");
        string weather = _service.GetWeather(city);
        _cache[city] = weather;
        return weather;
    }
}

// Usage
IWeatherService service = new CachingWeatherServiceProxy(new WeatherService());
Console.WriteLine(service.GetWeather("Boston"));   // Fetches from API
Console.WriteLine(service.GetWeather("Boston"));   // Returns from cache
Console.WriteLine(service.GetWeather("NewYork")); // Fetches from API
```

### 5. Remote Proxy (Network Objects)

```csharp
// Remote service interface
public interface IRemoteCalculator
{
    int Add(int a, int b);
    int Multiply(int a, int b);
}

// Would be on remote server
public class RemoteCalculator : IRemoteCalculator
{
    public int Add(int a, int b)
    {
        Console.WriteLine("[REMOTE] Computing add operation");
        return a + b;
    }

    public int Multiply(int a, int b)
    {
        Console.WriteLine("[REMOTE] Computing multiply operation");
        return a * b;
    }
}

// Remote Proxy - Simulates network communication
public class RemoteCalculatorProxy : IRemoteCalculator
{
    private RemoteCalculator _remoteService;
    private string _serverAddress = "192.168.1.100";

    public RemoteCalculatorProxy()
    {
        // In real scenario, would establish network connection
        Console.WriteLine($"Connecting to remote server at {_serverAddress}");
        _remoteService = new RemoteCalculator();
    }

    public int Add(int a, int b)
    {
        Console.WriteLine($"[NETWORK] Sending {a} + {b} request");
        int result = _remoteService.Add(a, b);
        Console.WriteLine($"[NETWORK] Received result: {result}");
        return result;
    }

    public int Multiply(int a, int b)
    {
        Console.WriteLine($"[NETWORK] Sending {a} * {b} request");
        int result = _remoteService.Multiply(a, b);
        Console.WriteLine($"[NETWORK] Received result: {result}");
        return result;
    }
}

// Usage
IRemoteCalculator calc = new RemoteCalculatorProxy();
Console.WriteLine(calc.Add(5, 3));
Console.WriteLine(calc.Multiply(4, 7));
```

## Real-World Example: Complete Database Proxy

```csharp
public interface IDatabase
{
    string Query(string sql);
    void Insert(string table, string data);
}

public class RealDatabase : IDatabase
{
    private string _connectionString;

    public RealDatabase(string connectionString)
    {
        _connectionString = connectionString;
        Console.WriteLine($"Database connected: {connectionString}");
    }

    public string Query(string sql)
    {
        System.Threading.Thread.Sleep(100);
        return $"Result from query: {sql}";
    }

    public void Insert(string table, string data)
    {
        Console.WriteLine($"Inserted into {table}: {data}");
    }
}

// Comprehensive proxy with logging, caching, and access control
public class DatabaseProxy : IDatabase
{
    private RealDatabase _database;
    private Dictionary<string, string> _queryCache = new();
    private string _userRole;
    private List<string> _queryLog = new();

    public DatabaseProxy(string connectionString, string userRole)
    {
        _userRole = userRole;
        _database = new RealDatabase(connectionString);
    }

    public string Query(string sql)
    {
        // Access control
        if (_userRole == "Guest")
            throw new UnauthorizedAccessException("Guests cannot query");

        // Check cache
        if (_queryCache.ContainsKey(sql))
        {
            Console.WriteLine("[CACHE] Returning cached result");
            return _queryCache[sql];
        }

        // Logging
        Console.WriteLine($"[LOG] Executing query: {sql}");
        _queryLog.Add(sql);

        // Execute on real database
        string result = _database.Query(sql);
        _queryCache[sql] = result;
        return result;
    }

    public void Insert(string table, string data)
    {
        // Access control
        if (_userRole != "Admin")
            throw new UnauthorizedAccessException("Only admins can insert");

        // Logging
        Console.WriteLine($"[LOG] Admin {_userRole} inserting into {table}");

        _database.Insert(table, data);
    }

    public List<string> GetQueryLog() => new List<string>(_queryLog);
}

// Usage
var proxy = new DatabaseProxy("Server=localhost", "User");
Console.WriteLine(proxy.Query("SELECT * FROM Users"));
Console.WriteLine(proxy.Query("SELECT * FROM Users")); // From cache
proxy.Insert("Users", "John"); // Throws exception - only admin can insert
```

## Advantages

✅ **Lazy Loading**: Create expensive objects only when needed  
✅ **Access Control**: Restrict object access based on permissions  
✅ **Caching**: Improve performance with result caching  
✅ **Logging**: Monitor and track object interactions  
✅ **Transparent**: Client sees same interface as real object  

## Disadvantages

❌ **Extra Class**: Adds another abstraction layer  
❌ **Performance Overhead**: Proxy check adds small overhead  
❌ **Complexity**: Logic duplication with proxy  
❌ **Debugging**: Extra layer to trace through  

## Proxy vs Other Patterns

| Pattern | Purpose | Relationship |
|---------|---------|---|
| **Proxy** | Control/enhance access to object | 1-to-1 with subject |
| **Decorator** | Add features to object | Wraps dynamically |
| **Adapter** | Convert incompatible interfaces | Converts between types |
| **Facade** | Simplify complex subsystem | 1-to-many |

## When to Use

- Objects are expensive to create (Virtual Proxy)
- Need to control access (Protection Proxy)
- Want to add logging/monitoring (Logging Proxy)
- Cache expensive operations (Caching Proxy)
- Access remote objects (Remote Proxy)

## When NOT to Use

- Simple objects with minimal overhead
- Direct access is already necessary
- Proxy logic becomes too complex
- Performance is critical

## Summary

The Proxy Pattern is powerful for controlling object access and adding behavior transparently. Use Virtual Proxy for lazy loading, Protection Proxy for access control, and specialized proxies for logging, caching, or remote access. The key is maintaining the same interface so clients interact naturally with either the proxy or real object.
