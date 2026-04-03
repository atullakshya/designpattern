# Builder Pattern

## Definition
The Builder Pattern constructs complex objects **step-by-step** through a fluent interface, separating the construction logic from the representation. This makes creating objects with many optional parameters clean and readable.

## Core Concept
- **Step-by-Step Construction**: Build objects incrementally
- **Fluent Interface**: Chain method calls for readability
- **Separation of Concerns**: Construction logic separate from the object itself
- **Optional Parameters**: Elegant handling of many optional fields
- **Immutability**: Often used with immutable objects

## Problem It Solves
- Constructors with many parameters (telescoping constructors)
- Optional parameters scattered throughout constructors
- Complex initialization logic cluttering the main class
- Need for intermediate object states during construction

## Generic Implementation

### Basic Builder Pattern

```csharp
// Product Class
public class Car
{
    public string Engine { get; set; }
    public string Transmission { get; set; }
    public string Color { get; set; }
    public bool HasGPS { get; set; }
    public bool HasSunroof { get; set; }
    public int Doors { get; set; }

    public override string ToString()
    {
        return $"Car: {Engine} engine, {Transmission}, {Color}, " +
               $"Doors: {Doors}, GPS: {HasGPS}, Sunroof: {HasSunroof}";
    }
}

// Builder Class
public class CarBuilder
{
    private Car _car = new Car();

    // Fluent interface - each method returns 'this' for chaining
    public CarBuilder WithEngine(string engine)
    {
        _car.Engine = engine;
        return this;
    }

    public CarBuilder WithTransmission(string transmission)
    {
        _car.Transmission = transmission;
        return this;
    }

    public CarBuilder WithColor(string color)
    {
        _car.Color = color;
        return this;
    }

    public CarBuilder WithGPS(bool hasGPS = true)
    {
        _car.HasGPS = hasGPS;
        return this;
    }

    public CarBuilder WithSunroof(bool hasSunroof = true)
    {
        _car.HasSunroof = hasSunroof;
        return this;
    }

    public CarBuilder WithDoors(int doors)
    {
        _car.Doors = doors;
        return this;
    }

    public Car Build()
    {
        return _car;
    }
}

// Usage
var car = new CarBuilder()
    .WithEngine("V8")
    .WithTransmission("Automatic")
    .WithColor("Red")
    .WithGPS()
    .WithSunroof()
    .WithDoors(4)
    .Build();

Console.WriteLine(car);
// Output: Car: V8 engine, Automatic, Red, Doors: 4, GPS: True, Sunroof: True
```

### Inner Builder Class (Recommended)

```csharp
// Product Class with Inner Builder
public class Person
{
    public string FirstName { get; }
    public string LastName { get; }
    public int Age { get; }
    public string Email { get; }
    public string Phone { get; }
    public string Address { get; }

    // Private constructor - can only be called by Builder
    private Person(PersonBuilder builder)
    {
        FirstName = builder.FirstName;
        LastName = builder.LastName;
        Age = builder.Age;
        Email = builder.Email;
        Phone = builder.Phone;
        Address = builder.Address;
    }

    public override string ToString()
    {
        return $"{FirstName} {LastName}, Age: {Age}, Email: {Email}, " +
               $"Phone: {Phone}, Address: {Address}";
    }

    // Inner Builder Class
    public class PersonBuilder
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public int Age { get; set; }
        public string Email { get; set; }
        public string Phone { get; set; }
        public string Address { get; set; }

        public PersonBuilder SetFirstName(string firstName)
        {
            FirstName = firstName;
            return this;
        }

        public PersonBuilder SetLastName(string lastName)
        {
            LastName = lastName;
            return this;
        }

        public PersonBuilder SetAge(int age)
        {
            Age = age;
            return this;
        }

        public PersonBuilder SetEmail(string email)
        {
            Email = email;
            return this;
        }

        public PersonBuilder SetPhone(string phone)
        {
            Phone = phone;
            return this;
        }

        public PersonBuilder SetAddress(string address)
        {
            Address = address;
            return this;
        }

        public Person Build()
        {
            return new Person(this);
        }
    }
}

// Usage
var person = new Person.PersonBuilder()
    .SetFirstName("John")
    .SetLastName("Doe")
    .SetAge(30)
    .SetEmail("john@example.com")
    .SetPhone("123-456-7890")
    .SetAddress("123 Main St")
    .Build();

Console.WriteLine(person);
```

### Director Pattern (Standardized Construction)

```csharp
// Builder Interface
public interface IHouseBuilder
{
    void BuildFoundation();
    void BuildWalls();
    void BuildRoof();
    void BuildDoors();
    void BuildWindows();
    House GetHouse();
}

// Concrete Builder
public class StoneHouseBuilder : IHouseBuilder
{
    private House _house = new House();

    public void BuildFoundation() => _house.Foundation = "Stone";
    public void BuildWalls() => _house.Walls = "Stone";
    public void BuildRoof() => _house.Roof = "Slate";
    public void BuildDoors() => _house.Doors = 4;
    public void BuildWindows() => _house.Windows = 8;
    public House GetHouse() => _house;
}

public class WoodHouseBuilder : IHouseBuilder
{
    private House _house = new House();

    public void BuildFoundation() => _house.Foundation = "Concrete";
    public void BuildWalls() => _house.Walls = "Wood";
    public void BuildRoof() => _house.Roof = "Wood";
    public void BuildDoors() => _house.Doors = 3;
    public void BuildWindows() => _house.Windows = 6;
    public House GetHouse() => _house;
}

// Product
public class House
{
    public string Foundation { get; set; }
    public string Walls { get; set; }
    public string Roof { get; set; }
    public int Doors { get; set; }
    public int Windows { get; set; }

    public override string ToString()
    {
        return $"House: Foundation={Foundation}, Walls={Walls}, " +
               $"Roof={Roof}, Doors={Doors}, Windows={Windows}";
    }
}

// Director - Defines construction order
public class HouseDirector
{
    public House BuildStandardHouse(IHouseBuilder builder)
    {
        builder.BuildFoundation();
        builder.BuildWalls();
        builder.BuildRoof();
        builder.BuildDoors();
        builder.BuildWindows();
        return builder.GetHouse();
    }

    public House BuildMinimalHouse(IHouseBuilder builder)
    {
        builder.BuildFoundation();
        builder.BuildWalls();
        builder.BuildRoof();
        return builder.GetHouse();
    }
}

// Usage
var director = new HouseDirector();
var stoneHouse = director.BuildStandardHouse(new StoneHouseBuilder());
var woodHouse = director.BuildMinimalHouse(new WoodHouseBuilder());

Console.WriteLine(stoneHouse);
Console.WriteLine(woodHouse);
```

## Real-World Examples

### SQL Query Builder
```csharp
public class QueryBuilder
{
    private string _selectClause = "";
    private string _fromClause = "";
    private string _whereClause = "";
    private string _orderByClause = "";

    public QueryBuilder Select(string columns)
    {
        _selectClause = $"SELECT {columns}";
        return this;
    }

    public QueryBuilder From(string table)
    {
        _fromClause = $"FROM {table}";
        return this;
    }

    public QueryBuilder Where(string condition)
    {
        _whereClause = $"WHERE {condition}";
        return this;
    }

    public QueryBuilder OrderBy(string column)
    {
        _orderByClause = $"ORDER BY {column}";
        return this;
    }

    public string Build()
    {
        var query = new StringBuilder();
        if (!string.IsNullOrEmpty(_selectClause)) query.Append(_selectClause);
        if (!string.IsNullOrEmpty(_fromClause)) query.Append($" {_fromClause}");
        if (!string.IsNullOrEmpty(_whereClause)) query.Append($" {_whereClause}");
        if (!string.IsNullOrEmpty(_orderByClause)) query.Append($" {_orderByClause}");
        return query.ToString();
    }
}

// Usage
var query = new QueryBuilder()
    .Select("Id, Name, Email")
    .From("Users")
    .Where("Age > 18")
    .OrderBy("Name")
    .Build();

Console.WriteLine(query);
// Output: SELECT Id, Name, Email FROM Users WHERE Age > 18 ORDER BY Name
```

### HTTP Request Builder
```csharp
public class HttpRequestBuilder
{
    private string _method = "GET";
    private string _url = "";
    private Dictionary<string, string> _headers = new();
    private object _body = null;

    public HttpRequestBuilder WithMethod(string method)
    {
        _method = method;
        return this;
    }

    public HttpRequestBuilder WithUrl(string url)
    {
        _url = url;
        return this;
    }

    public HttpRequestBuilder AddHeader(string key, string value)
    {
        _headers[key] = value;
        return this;
    }

    public HttpRequestBuilder WithBody(object body)
    {
        _body = body;
        return this;
    }

    public HttpRequest Build()
    {
        return new HttpRequest
        {
            Method = _method,
            Url = _url,
            Headers = _headers,
            Body = _body
        };
    }
}

public class HttpRequest
{
    public string Method { get; set; }
    public string Url { get; set; }
    public Dictionary<string, string> Headers { get; set; }
    public object Body { get; set; }
}

// Usage
var request = new HttpRequestBuilder()
    .WithMethod("POST")
    .WithUrl("https://api.example.com/users")
    .AddHeader("Content-Type", "application/json")
    .AddHeader("Authorization", "Bearer token123")
    .WithBody(new { Name = "John", Email = "john@example.com" })
    .Build();
```

### Configuration Builder
```csharp
public class AppConfigBuilder
{
    private readonly Dictionary<string, object> _config = new();

    public AppConfigBuilder SetDatabase(string connectionString)
    {
        _config["Database"] = connectionString;
        return this;
    }

    public AppConfigBuilder SetTimeout(int seconds)
    {
        _config["Timeout"] = seconds;
        return this;
    }

    public AppConfigBuilder SetLogLevel(string level)
    {
        _config["LogLevel"] = level;
        return this;
    }

    public AppConfigBuilder EnableFeature(string feature)
    {
        _config[feature] = true;
        return this;
    }

    public AppConfig Build()
    {
        return new AppConfig(_config);
    }
}

public class AppConfig
{
    private readonly Dictionary<string, object> _settings;

    public AppConfig(Dictionary<string, object> settings)
    {
        _settings = settings;
    }

    public T Get<T>(string key) => (T)_settings.GetValueOrDefault(key);
}

// Usage
var config = new AppConfigBuilder()
    .SetDatabase("Server=localhost;Database=mydb")
    .SetTimeout(30)
    .SetLogLevel("Debug")
    .EnableFeature("EmailNotifications")
    .Build();
```

## Advantages

✅ **Readability**: Fluent interface is clear and self-documenting  
✅ **Flexibility**: Add/remove optional parameters easily  
✅ **Immutability**: Can create immutable objects  
✅ **Single Responsibility**: Separates construction from representation  
✅ **Cleaner Code**: Avoids telescoping constructors  

## Disadvantages

❌ **Extra Classes**: Requires builder class alongside product  
❌ **Verbose**: More code for simple objects  
❌ **Memory Overhead**: Temporary builder object creation  

## When to Use

- Objects with many optional parameters
- Complex initialization logic
- Need different representations of the same object
- Immutable objects
- Multiple construction steps

## When NOT to Use

- Simple objects with few properties
- Constructor parameters are minimal
- When constructor overloading is sufficient

## Summary

The Builder Pattern shines when dealing with complex objects or many optional parameters. It provides a clean, readable way to construct objects step-by-step, making code more maintainable and intuitive compared to telescoping constructors or setter chains.
