# .NET Core Microservice Design Patterns

## 1. Singleton Pattern  
Provides a way to create a single instance of a class, ensuring that there is only one instance throughout the application.

**Example:**  
A logging service can be instantiated as a singleton to ensure that all components share the same logging instance.  

```csharp
public class Logger {
    private static Logger _instance;
    private static readonly object _lock = new object();

    private Logger() {}

    public static Logger Instance {
        get {
            lock (_lock) {
                if (_instance == null) {
                    _instance = new Logger();
                }
            }
            return _instance;
        }
    }
}
```

## 2. Factory Pattern  
Defines an interface for creating an object, but allows subclasses to alter the type of objects that will be created.

**Example:**  
Database connections can use a factory pattern to create connection objects based on different databases.

```csharp
public interface IDatabaseConnection  {
    void Connect();
}

public class SqlDatabaseConnection : IDatabaseConnection {
    public void Connect() {
        // SQL Database connection logic
    }
}

public class MongoDatabaseConnection : IDatabaseConnection {
    public void Connect() {
        // MongoDB connection logic
    }
}

public class DatabaseConnectionFactory {
    public static IDatabaseConnection CreateConnection(string type) {
        if (type == "sql") {
            return new SqlDatabaseConnection();
        } else if (type == "mongo") {
            return new MongoDatabaseConnection();
        }
        throw new ArgumentException("Invalid connection type");
    }
}
}
```

## 3. Repository Pattern  
Encapsulates the logic required to access data sources and provides a more testable and maintainable codebase.

**Example:**  
Data access logic can be encapsulated with interfaces to promote better separation of concerns.  

```csharp
public interface IRepository<T> {
    IEnumerable<T> GetAll();
    T GetById(int id);
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}

public class UserRepository : IRepository<User> {
    // Implementation of methods
}
```

## 4. CQRS (Command Query Responsibility Segregation)  
Separation of read and write operations into different models for better decoupling and scalability.

**Example:**  
A web API that handles creating, updating, and fetching user data can split commands and queries into different classes.

## 5. Event Sourcing  
Stores the state of a system as a sequence of events, enabling better tracking of changes over time.

**Example:**  
Any change to an aggregate can be stored as an event, facilitating the ability to reconstruct past states.

## 6. Dependency Injection  
A technique where the creation and management of dependencies are handled externally, promoting loose coupling.

**Example:**  
ASP.NET Core uses built-in dependency injection for service registrations.

## 7. Middleware  
Allows developers to insert custom middleware in the request pipeline to handle cross-cutting concerns, such as logging.

**Example:**  
Custom authentication can be implemented via middleware in the ASP.NET Core pipeline.

## Conclusion
These design patterns provide a robust framework for architectural decisions when building .NET Core microservices, enhancing maintainability and scalability.