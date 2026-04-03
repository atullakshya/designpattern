# Singleton Pattern

## Definition
The Singleton Pattern ensures that **only one instance** of a class can exist throughout the application's lifetime and provides a global point of access to that instance.

## Core Concept
- **Single Responsibility**: One instance = one state, preventing multiple instances from causing conflicts
- **Global Access**: The instance is accessible from anywhere in the application
- **Lazy or Eager Initialization**: Control when the instance is created
- **Thread Safety**: Ensure only one instance in multi-threaded environments
- **Memory Efficiency**: Avoid unnecessary object creation

## Problem It Solves
- Excessive object creation consuming memory and resources
- Multiple instances causing data inconsistency
- Need for centralized state management (e.g., logger, database connection pool)
- Resource sharing across application components
- Configuration management in distributed systems

## Types of Singleton Implementations

### 1. EAGER SINGLETON (Simple but Always Creates Instance)

```csharp
public sealed class EagerSingleton
{
    // Instance created at class loading time (eager initialization)
    // This happens when the class is first accessed, not when the application starts
    private static readonly EagerSingleton _instance = new EagerSingleton();

    // Private constructor prevents external instantiation
    // No parameters allowed - must be parameterless
    private EagerSingleton()
    {
        Console.WriteLine("EagerSingleton instance created at class load time");
        // Expensive initialization can happen here
        // This will always execute, even if instance is never used
    }

    // Public static property for global access
    // Property getter is thread-safe by default for readonly static fields
    public static EagerSingleton Instance => _instance;

    // Example method
    public void DoWork()
    {
        Console.WriteLine("EagerSingleton doing work");
    }
}

// WHY THIS APPROACH?
// + Simple to implement
// + Thread-safe by default (CLR guarantees)
// + No null checks needed
// - Instance created even if never used (wasteful)
// - No control over initialization timing
// - Cannot pass parameters to constructor
// - Exceptions in constructor are hard to handle
```

### 2. LAZY SINGLETON (Thread-Safe with Lazy<T>)

```csharp
public sealed class LazySingleton
{
    // Lazy<T> provides thread-safe lazy initialization
    // Generic Lazy<T> class handles all thread-safety automatically
    // Func<T> delegate specifies how to create the instance
    private static readonly Lazy<LazySingleton> _instance =
        new Lazy<LazySingleton>(() => new LazySingleton());

    // Private constructor - called only when Lazy<T>.Value is accessed first time
    private LazySingleton()
    {
        Console.WriteLine("LazySingleton instance created on first access");
        // Expensive operations here are deferred until needed
    }

    // Property returns the lazily initialized instance
    // First access triggers creation, subsequent accesses return same instance
    public static LazySingleton Instance => _instance.Value;

    public void DoWork()
    {
        Console.WriteLine("LazySingleton doing work");
    }
}

// WHY Lazy<T>?
// + Thread-safe without manual locking
// + Instance created only when first accessed
// + Exception handling in constructor works normally
// + Can use lambda for complex initialization
// + .NET built-in solution (recommended)
```

### 3. DOUBLE-CHECKED LOCKING SINGLETON (Manual Thread-Safe)

```csharp
public sealed class DoubleCheckedLockingSingleton
{
    // Volatile ensures changes are visible across threads immediately
    // Prevents compiler/CPU reordering optimizations that could break thread safety
    private static volatile DoubleCheckedLockingSingleton _instance;

    // Lock object for synchronization
    // Using object instead of lock(this) prevents deadlocks
    private static readonly object _lock = new object();

    // Private constructor
    private DoubleCheckedLockingSingleton()
    {
        Console.WriteLine("DoubleCheckedLockingSingleton instance created");
        // Simulate expensive initialization
        System.Threading.Thread.Sleep(100);
    }

    public static DoubleCheckedLockingSingleton Instance
    {
        get
        {
            // First check - no lock needed if already created
            if (_instance == null)
            {
                // Second check - with lock to prevent race conditions
                lock (_lock)
                {
                    // Double-check pattern: verify again inside lock
                    if (_instance == null)
                    {
                        _instance = new DoubleCheckedLockingSingleton();
                    }
                }
            }
            return _instance;
        }
    }

    public void DoWork()
    {
        Console.WriteLine("DoubleCheckedLockingSingleton doing work");
    }
}

// WHY DOUBLE-CHECKED LOCKING?
// + Performance: Lock only when instance is null
// + Thread-safe: Prevents multiple instances in multi-threaded scenarios
// + Lazy: Instance created only when needed
// - Complex: Easy to get wrong without volatile and double-check
// - Verbose: More code than Lazy<T> approach
// - volatile keyword needed for correctness
```

### 4. LOCK-BASED SINGLETON (Simple Thread-Safe)

```csharp
public sealed class LockBasedSingleton
{
    private static LockBasedSingleton _instance;
    private static readonly object _lock = new object();

    private LockBasedSingleton()
    {
        Console.WriteLine("LockBasedSingleton instance created");
    }

    public static LockBasedSingleton Instance
    {
        get
        {
            // Always acquire lock - less efficient but simpler
            lock (_lock)
            {
                if (_instance == null)
                {
                    _instance = new LockBasedSingleton();
                }
                return _instance;
            }
        }
    }

    public void DoWork()
    {
        Console.WriteLine("LockBasedSingleton doing work");
    }
}

// WHY LOCK-BASED?
// + Simple to understand
// + Thread-safe
// + Lazy initialization
// - Performance: Lock acquired on every access (even after initialization)
// - Not as efficient as double-checked locking
// - Still more complex than Lazy<T>
```

### 5. SINGLETON WITH PARAMETERS (Using Factory Method)

```csharp
public sealed class ConfigurableSingleton
{
    private static readonly Lazy<ConfigurableSingleton> _instance =
        new Lazy<ConfigurableSingleton>(() => CreateInstance());

    private readonly string _configPath;
    private readonly int _maxConnections;

    // Private constructor with parameters
    private ConfigurableSingleton(string configPath, int maxConnections)
    {
        _configPath = configPath;
        _maxConnections = maxConnections;
        Console.WriteLine($"ConfigurableSingleton created with config: {_configPath}, maxConn: {_maxConnections}");
    }

    // Factory method for lazy initialization with parameters
    private static ConfigurableSingleton CreateInstance()
    {
        // Parameters can come from configuration, environment, etc.
        string configPath = Environment.GetEnvironmentVariable("CONFIG_PATH") ?? "default.config";
        int maxConnections = int.Parse(Environment.GetEnvironmentVariable("MAX_CONNECTIONS") ?? "10");

        return new ConfigurableSingleton(configPath, maxConnections);
    }

    public static ConfigurableSingleton Instance => _instance.Value;

    public void ShowConfig()
    {
        Console.WriteLine($"Config: {_configPath}, Max Connections: {_maxConnections}");
    }
}

// WHY PARAMETERS IN SINGLETON?
// + Configuration-driven initialization
// + Environment-specific settings
// + Dependency injection for singletons
// - Cannot change parameters after creation
// - Parameters must be available at creation time
```

### 6. SINGLETON WITH DISPOSAL (IDisposable Implementation)

```csharp
public sealed class DisposableSingleton : IDisposable
{
    private static readonly Lazy<DisposableSingleton> _instance =
        new Lazy<DisposableSingleton>(() => new DisposableSingleton());

    private bool _disposed = false;
    private readonly System.Timers.Timer _timer;

    private DisposableSingleton()
    {
        Console.WriteLine("DisposableSingleton created");
        // Initialize resources that need cleanup
        _timer = new System.Timers.Timer(1000);
        _timer.Elapsed += (s, e) => Console.WriteLine("Timer tick");
        _timer.Start();
    }

    public static DisposableSingleton Instance => _instance.Value;

    public void DoWork()
    {
        if (_disposed) throw new ObjectDisposedException(nameof(DisposableSingleton));
        Console.WriteLine("DisposableSingleton doing work");
    }

    // Cleanup resources
    public void Dispose()
    {
        if (!_disposed)
        {
            _disposed = true;
            _timer?.Dispose();
            Console.WriteLine("DisposableSingleton disposed");
        }
    }
}

// WHY DISPOSABLE SINGLETON?
// + Proper resource cleanup
// + Managed resources (timers, connections, etc.)
// + Prevents resource leaks
// + Application shutdown cleanup
// - Must ensure Dispose() is called
// - Cannot be garbage collected normally
```

### 7. SINGLETON REGISTRY (Multiple Singletons)

```csharp
public static class SingletonRegistry
{
    // Dictionary to store different singleton instances by type
    private static readonly Dictionary<Type, object> _instances = new();
    private static readonly object _lock = new object();

    // Generic method to get or create singleton instance
    public static T GetInstance<T>() where T : class, new()
    {
        Type type = typeof(T);

        // First check without lock
        if (!_instances.ContainsKey(type))
        {
            lock (_lock)
            {
                // Double-check inside lock
                if (!_instances.ContainsKey(type))
                {
                    _instances[type] = new T();
                    Console.WriteLine($"Created singleton instance of {type.Name}");
                }
            }
        }

        return (T)_instances[type];
    }

    // Clear all instances (useful for testing)
    public static void Clear()
    {
        lock (_lock)
        {
            _instances.Clear();
        }
    }
}

// Example singleton classes
public class DatabaseManager { }
public class CacheManager { }
public class SecurityManager { }

// Usage
var dbManager = SingletonRegistry.GetInstance<DatabaseManager>();
var cacheManager = SingletonRegistry.GetInstance<CacheManager>();
var securityManager = SingletonRegistry.GetInstance<SecurityManager>();

// WHY SINGLETON REGISTRY?
// + Manage multiple singletons centrally
// + Generic approach for any type
// + Easy to clear for testing
// + Type-safe generic implementation
// - All singletons share same lifecycle
// - Cannot have different initialization logic
```

### 8. SINGLETON WITH DEPENDENCY INJECTION

```csharp
// Interface for dependency injection
public interface ILogger
{
    void Log(string message);
}

public interface IConfiguration
{
    string GetConnectionString();
}

// Singleton that depends on injected services
public sealed class DatabaseService : IDisposable
{
    private static readonly Lazy<DatabaseService> _instance =
        new Lazy<DatabaseService>(() => CreateInstance());

    private readonly ILogger _logger;
    private readonly IConfiguration _config;
    private readonly string _connectionString;

    private DatabaseService(ILogger logger, IConfiguration config)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _config = config ?? throw new ArgumentNullException(nameof(config));
        _connectionString = _config.GetConnectionString();

        _logger.Log("DatabaseService singleton created");
    }

    // Factory method that can access DI container
    private static DatabaseService CreateInstance()
    {
        // In real app, this would come from DI container
        var logger = new ConsoleLogger();
        var config = new AppConfiguration();
        return new DatabaseService(logger, config);
    }

    public static DatabaseService Instance => _instance.Value;

    public void ExecuteQuery(string query)
    {
        _logger.Log($"Executing query: {query}");
        // Use _connectionString for actual database work
    }

    public void Dispose()
    {
        _logger.Log("DatabaseService disposed");
    }
}

// WHY DEPENDENCY INJECTION WITH SINGLETON?
// + Testable: Can inject mock dependencies
// + Flexible: Dependencies can be configured
// + Separation of Concerns: Singleton focuses on its logic
// + Inversion of Control: Dependencies provided externally
// - More complex setup
// - Dependencies must be available at creation time
```

### 9. THREAD-LOCAL SINGLETON (Per-Thread Instance)

```csharp
public sealed class ThreadLocalSingleton
{
    // ThreadLocal<T> creates separate instance per thread
    private static readonly ThreadLocal<ThreadLocalSingleton> _instance =
        new ThreadLocal<ThreadLocalSingleton>(() => new ThreadLocalSingleton());

    private readonly int _threadId;

    private ThreadLocalSingleton()
    {
        _threadId = System.Threading.Thread.CurrentThread.ManagedThreadId;
        Console.WriteLine($"ThreadLocalSingleton created for thread {_threadId}");
    }

    public static ThreadLocalSingleton Instance => _instance.Value;

    public void ShowThreadInfo()
    {
        Console.WriteLine($"Running on thread {_threadId}");
    }
}

// WHY THREAD-LOCAL SINGLETON?
// + Separate instance per thread
// + Thread-safe by design
// + Useful for thread-specific resources
// + No synchronization needed
// - Memory usage: One instance per thread
// - Not truly "singleton" across threads
// - Cleanup when threads end
```

### 10. SINGLETON WITH ASYNC INITIALIZATION

```csharp
public sealed class AsyncSingleton
{
    private static readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);
    private static AsyncSingleton _instance;
    private bool _initialized = false;

    // Private constructor
    private AsyncSingleton() { }

    // Async initialization method
    public static async Task<AsyncSingleton> GetInstanceAsync()
    {
        if (_instance == null)
        {
            await _semaphore.WaitAsync();
            try
            {
                if (_instance == null)
                {
                    _instance = new AsyncSingleton();
                    await _instance.InitializeAsync();
                }
            }
            finally
            {
                _semaphore.Release();
            }
        }
        return _instance;
    }

    private async Task InitializeAsync()
    {
        if (_initialized) return;

        Console.WriteLine("AsyncSingleton initializing...");
        await Task.Delay(1000); // Simulate async initialization
        _initialized = true;
        Console.WriteLine("AsyncSingleton initialized");
    }

    public void DoWork()
    {
        if (!_initialized) throw new InvalidOperationException("Not initialized");
        Console.WriteLine("AsyncSingleton doing work");
    }
}

// WHY ASYNC SINGLETON?
// + Async initialization (database connections, network setup)
// + Non-blocking initialization
// + Proper async/await support
// + Thread-safe async initialization
// - More complex than sync versions
// - Must await GetInstanceAsync()
// - Cannot use simple property access
```

## Key Characteristics

| Aspect | Details |
|--------|---------|
| **Creation** | Instance created once, either at app start (eager) or first use (lazy) |
| **Lifespan** | Exists throughout the application lifecycle |
| **Access Scope** | Static member accessible globally |
| **Thread Safety** | Use `Lazy<T>`, locks, or double-checked locking |
| **Sealed Class** | Prevents inheritance and instantiation workarounds |
| **Memory** | Single instance vs multiple instances trade-off |
| **Initialization** | Eager vs Lazy vs Async initialization options |

## Implementation Comparison

| Implementation | Thread-Safe | Lazy | Performance | Complexity | Recommended |
|----------------|-------------|------|-------------|------------|-------------|
| **Eager** | ✅ | ❌ | Fastest | Low | Simple cases |
| **Lazy<T>** | ✅ | ✅ | Fast | Low | **Most cases** |
| **Double-Checked** | ✅ | ✅ | Good | Medium | Learning purposes |
| **Lock-Based** | ✅ | ✅ | Slower | Low | Simple multi-threading |
| **With Parameters** | ✅ | ✅ | Fast | Medium | Configurable singletons |
| **Disposable** | ✅ | ✅ | Fast | Medium | Resource management |
| **Registry** | ✅ | ✅ | Good | High | Multiple singletons |
| **DI-Enabled** | ✅ | ✅ | Fast | High | Testable singletons |
| **Thread-Local** | ✅ | ✅ | Good | Medium | Per-thread resources |
| **Async** | ✅ | ✅ | Good | High | Async initialization |

## Real-World Examples

### 1. Logger Singleton (Complete Implementation)
```csharp
public class LoggerSingleton
{
    private static readonly Lazy<LoggerSingleton> _instance = 
        new Lazy<LoggerSingleton>(() => new LoggerSingleton());
    
    private readonly List<string> _logs = new();
    private readonly object _logLock = new(); // For thread-safe logging
    
    // WHY: Private constructor prevents external instantiation
    private LoggerSingleton() 
    {
        Console.WriteLine("Logger singleton initialized");
    }

    // WHY: Static property provides global access point
    public static LoggerSingleton Instance => _instance.Value;

    // WHY: Thread-safe logging with lock
    public void Log(string message)
    {
        string logEntry = $"{DateTime.Now:yyyy-MM-dd HH:mm:ss} - {message}";
        lock (_logLock)
        {
            _logs.Add(logEntry);
        }
        Console.WriteLine(logEntry);
    }

    // WHY: Method to retrieve all logs (useful for debugging)
    public IEnumerable<string> GetAllLogs()
    {
        lock (_logLock)
        {
            return new List<string>(_logs); // Return copy to prevent external modification
        }
    }

    // WHY: Clear logs method for testing or maintenance
    public void ClearLogs()
    {
        lock (_logLock)
        {
            _logs.Clear();
        }
    }
}

// WHY THIS SINGLETON?
// - Centralized logging across entire application
// - Thread-safe for multi-threaded applications
// - Lazy initialization (created only when first log is written)
// - Memory efficient (only one logger instance)
// - Easy to extend with different log levels, file output, etc.
```

### 2. Database Connection Pool Singleton
```csharp
public sealed class DatabaseConnectionPool : IDisposable
{
    private static readonly Lazy<DatabaseConnectionPool> _instance =
        new Lazy<DatabaseConnectionPool>(() => new DatabaseConnectionPool());

    private readonly List<IDbConnection> _availableConnections = new();
    private readonly List<IDbConnection> _usedConnections = new();
    private readonly object _poolLock = new();
    private readonly int _maxPoolSize = 10;
    private bool _disposed = false;

    // WHY: Private constructor initializes connection pool
    private DatabaseConnectionPool()
    {
        Console.WriteLine("Database connection pool initialized");
        InitializePool();
    }

    // WHY: Initialize pool with minimum connections
    private void InitializePool()
    {
        for (int i = 0; i < 5; i++) // Start with 5 connections
        {
            var connection = CreateConnection();
            _availableConnections.Add(connection);
        }
    }

    // WHY: Factory method for creating connections
    private IDbConnection CreateConnection()
    {
        // In real implementation, this would create actual database connections
        return new SqlConnection("Server=localhost;Database=mydb");
    }

    public static DatabaseConnectionPool Instance => _instance.Value;

    // WHY: Get connection from pool (thread-safe)
    public IDbConnection GetConnection()
    {
        if (_disposed) throw new ObjectDisposedException(nameof(DatabaseConnectionPool));

        lock (_poolLock)
        {
            if (_availableConnections.Count > 0)
            {
                var connection = _availableConnections[0];
                _availableConnections.RemoveAt(0);
                _usedConnections.Add(connection);
                return connection;
            }
            else if (_usedConnections.Count < _maxPoolSize)
            {
                var connection = CreateConnection();
                _usedConnections.Add(connection);
                return connection;
            }
            else
            {
                throw new InvalidOperationException("Connection pool exhausted");
            }
        }
    }

    // WHY: Return connection to pool
    public void ReturnConnection(IDbConnection connection)
    {
        if (_disposed) return;

        lock (_poolLock)
        {
            if (_usedConnections.Remove(connection))
            {
                _availableConnections.Add(connection);
            }
        }
    }

    // WHY: Proper resource cleanup
    public void Dispose()
    {
        if (!_disposed)
        {
            _disposed = true;
            lock (_poolLock)
            {
                foreach (var conn in _availableConnections)
                    conn.Dispose();
                foreach (var conn in _usedConnections)
                    conn.Dispose();
                _availableConnections.Clear();
                _usedConnections.Clear();
            }
            Console.WriteLine("Database connection pool disposed");
        }
    }
}

// WHY THIS SINGLETON?
// - Manages limited database connections efficiently
// - Thread-safe connection borrowing/returning
// - Prevents connection leaks
// - Configurable pool size
// - Proper resource cleanup on application shutdown
```

### 3. Application Configuration Singleton
```csharp
public sealed class AppConfiguration
{
    private static readonly Lazy<AppConfiguration> _instance =
        new Lazy<AppConfiguration>(() => new AppConfiguration());

    private readonly Dictionary<string, string> _settings = new();
    private readonly FileSystemWatcher _configWatcher;
    private readonly object _settingsLock = new();

    // WHY: Private constructor loads configuration
    private AppConfiguration()
    {
        LoadConfiguration();
        SetupFileWatcher();
        Console.WriteLine("Application configuration loaded");
    }

    // WHY: Load settings from multiple sources
    private void LoadConfiguration()
    {
        // Load from app.config
        LoadFromAppConfig();
        // Load from environment variables
        LoadFromEnvironment();
        // Load from external config file
        LoadFromFile("appsettings.json");
    }

    // WHY: Watch for configuration file changes
    private void SetupFileWatcher()
    {
        _configWatcher = new FileSystemWatcher
        {
            Path = AppDomain.CurrentDomain.BaseDirectory,
            Filter = "appsettings.json",
            NotifyFilter = NotifyFilters.LastWrite
        };
        _configWatcher.Changed += OnConfigFileChanged;
        _configWatcher.EnableRaisingEvents = true;
    }

    // WHY: Reload configuration when file changes
    private void OnConfigFileChanged(object sender, FileSystemEventArgs e)
    {
        Console.WriteLine("Configuration file changed, reloading...");
        LoadConfiguration();
    }

    private void LoadFromAppConfig() { /* Implementation */ }
    private void LoadFromEnvironment() { /* Implementation */ }
    private void LoadFromFile(string fileName) { /* Implementation */ }

    public static AppConfiguration Instance => _instance.Value;

    // WHY: Thread-safe settings access
    public string GetSetting(string key)
    {
        lock (_settingsLock)
        {
            return _settings.TryGetValue(key, out var value) ? value : null;
        }
    }

    // WHY: Thread-safe settings modification
    public void SetSetting(string key, string value)
    {
        lock (_settingsLock)
        {
            _settings[key] = value;
        }
    }

    // WHY: Get all settings (return copy for safety)
    public IReadOnlyDictionary<string, string> GetAllSettings()
    {
        lock (_settingsLock)
        {
            return new Dictionary<string, string>(_settings);
        }
    }
}

// WHY THIS SINGLETON?
// - Single source of truth for application configuration
// - Hot-reload capability for configuration changes
// - Thread-safe access in multi-threaded applications
// - Multiple configuration sources support
// - Prevents configuration inconsistencies
```

## Advantages

✅ **Memory Efficient**: Only one instance exists, reducing memory footprint  
✅ **Controlled Access**: Centralized point of access prevents unauthorized instantiation  
✅ **Global State Management**: Perfect for application-wide state that must be consistent  
✅ **Lazy Initialization**: Instance created only when needed (with Lazy<T>)  
✅ **Thread-Safe**: Proper implementations prevent race conditions  
✅ **Resource Management**: Can manage shared resources efficiently  
✅ **Performance**: No overhead of creating multiple instances  
✅ **Consistency**: Ensures single point of truth for shared data  

## Disadvantages

❌ **Hidden Dependencies**: Classes using singletons have implicit dependencies  
❌ **Difficult to Test**: Hard to mock singletons in unit tests  
❌ **Global State**: Can lead to unexpected behaviors and tight coupling  
❌ **Tight Coupling**: Forces classes to depend on singleton implementation  
❌ **Inheritance Prevention**: Sealed classes cannot be extended  
❌ **Lifetime Management**: Instance lives entire application lifetime  
❌ **Testing Isolation**: Singletons retain state between tests  
❌ **Violation of SRP**: Singletons often handle multiple responsibilities  

## When to Use

- **Logger instances**: Centralized logging across application layers
- **Database connection pools**: Limited database connections management
- **Configuration managers**: Single source of application settings
- **Cache managers**: Shared caching across application components
- **Thread pools**: Manage concurrent operations centrally
- **Hardware interfaces**: Single access point to hardware resources
- **Application state**: Global application state management
- **Resource managers**: File handles, network connections, etc.
- **Security contexts**: Single authentication/authorization point
- **Service locators**: Central service registration and discovery

## When NOT to Use

- When you need multiple instances with different states
- When testing requires creating different instances
- When the singleton computation is expensive and not always needed
- When thread safety becomes problematic or complex
- When you can use dependency injection instead
- When the singleton creates tight coupling between classes
- When inheritance or polymorphism is needed
- When the singleton manages too many responsibilities
- When global state causes testing or maintainability issues

## Best Practices

### Implementation Best Practices
1. **Always use `Lazy<T>`** for thread-safe lazy initialization in modern C#
2. **Seal the class** to prevent inheritance and instantiation workarounds
3. **Make constructor private** to prevent direct instantiation
4. **Use static property** for instance access (not method)
5. **Document thread-safety guarantees** clearly
6. **Consider async initialization** for expensive setup operations
7. **Implement IDisposable** if singleton manages resources

### Design Best Practices
8. **Prefer dependency injection** over singletons when possible
9. **Document the singleton** clearly in code and documentation
10. **Avoid storing mutable state** in singletons when possible
11. **Consider thread-safety** from the beginning
12. **Use interfaces** to make singletons more testable
13. **Keep singletons focused** on single responsibility
14. **Provide cleanup mechanisms** for resource management

### Testing Best Practices
15. **Use dependency injection** in application code to enable testing
16. **Create reset/clear methods** for testing scenarios
17. **Mock singleton dependencies** rather than the singleton itself
18. **Test thread-safety** with concurrent access patterns
19. **Isolate singleton state** between test runs

## Testing Considerations

### Problem: Hard to mock singletons in tests
```csharp
[TestClass]
public class DatabaseOperationTests
{
    [TestMethod]
    public void TestDatabaseOperation()
    {
        // PROBLEM: Cannot easily replace DatabaseConnection.Instance with a mock
        // This makes testing difficult and creates tight coupling
        
        var service = new DatabaseService();
        service.SaveData("test data");
        // How to verify the save operation without hitting real database?
    }
}
```

### Better Approach: Use Dependency Injection
```csharp
// Define interface for the service
public interface IDatabaseConnection
{
    void Connect();
    void SaveData(string data);
}

// Singleton implements the interface
public sealed class DatabaseConnection : IDatabaseConnection
{
    private static readonly Lazy<DatabaseConnection> _instance = 
        new Lazy<DatabaseConnection>(() => new DatabaseConnection());

    private DatabaseConnection() { }

    public static DatabaseConnection Instance => _instance.Value;

    public void Connect() => Console.WriteLine("Connected");
    public void SaveData(string data) => Console.WriteLine($"Saved: {data}");
}

// Service depends on interface, not concrete singleton
public class DatabaseService
{
    private readonly IDatabaseConnection _connection;

    public DatabaseService(IDatabaseConnection connection)
    {
        _connection = connection;
    }

    public void SaveData(string data)
    {
        _connection.Connect();
        _connection.SaveData(data);
    }
}

// Testing becomes easy
[TestClass]
public class DatabaseOperationTests
{
    [TestMethod]
    public void TestDatabaseOperation()
    {
        // Create mock
        var mockConnection = new Mock<IDatabaseConnection>();
        
        // Inject mock into service
        var service = new DatabaseService(mockConnection.Object);
        
        // Test the service
        service.SaveData("test data");
        
        // Verify mock was called
        mockConnection.Verify(c => c.SaveData("test data"), Times.Once);
    }
}
```

## Common Pitfalls and Solutions

### Pitfall 1: Not Thread-Safe
```csharp
// PROBLEMATIC: Not thread-safe
public class UnsafeSingleton
{
    private static UnsafeSingleton _instance;
    
    private UnsafeSingleton() { }
    
    public static UnsafeSingleton Instance
    {
        get
        {
            if (_instance == null)
                _instance = new UnsafeSingleton(); // Race condition!
            return _instance;
        }
    }
}

// SOLUTION: Use Lazy<T> or proper locking
```

### Pitfall 2: Inheritance Workaround
```csharp
// PROBLEMATIC: Can be inherited and instantiated
public class BrokenSingleton
{
    private static BrokenSingleton _instance = new BrokenSingleton();
    protected BrokenSingleton() { } // Protected, not private!
    public static BrokenSingleton Instance => _instance;
}

// SOLUTION: Seal the class
public sealed class FixedSingleton : BrokenSingleton
{
    // Can create new instances by inheriting!
}
```

### Pitfall 3: Exception in Constructor
```csharp
// PROBLEMATIC: Exception during creation leaves null instance
public class ProblematicSingleton
{
    private static ProblematicSingleton _instance;
    
    private ProblematicSingleton()
    {
        throw new Exception("Construction failed!");
    }
    
    public static ProblematicSingleton Instance => _instance ?? (_instance = new ProblematicSingleton());
}

// SOLUTION: Use Lazy<T> which handles exceptions properly
```

### Pitfall 4: Memory Leaks
```csharp
// PROBLEMATIC: Holds references preventing garbage collection
public class LeakySingleton
{
    private static LeakySingleton _instance = new LeakySingleton();
    private List<byte[]> _largeData = new List<byte[]>();
    
    private LeakySingleton()
    {
        // Load lots of data that never gets cleaned up
        for (int i = 0; i < 1000; i++)
            _largeData.Add(new byte[1024 * 1024]); // 1MB each
    }
    
    public static LeakySingleton Instance => _instance;
}

// SOLUTION: Implement proper cleanup or use disposable pattern
```

## Summary

The Singleton Pattern is a powerful tool for ensuring single instance and global access, but it must be used judiciously. The **Lazy<T> approach** is recommended for most modern C# applications due to its simplicity, thread-safety, and performance.

**Key Takeaways:**
- Use `Lazy<T>` for thread-safe lazy initialization
- Seal classes to prevent inheritance workarounds
- Prefer dependency injection for testability
- Implement proper resource cleanup when needed
- Document thread-safety guarantees
- Avoid overusing singletons - they create tight coupling

**Remember**: Singletons are not inherently evil, but they should be used when the benefits outweigh the coupling and testing challenges they introduce.
    }
}
```

## Summary

The Singleton Pattern is powerful for controlling instance creation and providing global access, but should be used judiciously. In modern C# development, **dependency injection** is often preferred over singletons for better testability and loose coupling.
