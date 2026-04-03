# Facade Pattern

## Definition
The Facade Pattern provides a **single, simplified interface** to a complex subsystem of classes, libraries, or components. It acts as a "front door" that hides the complexity underneath.

## Core Concept
- **Simplification**: Provides clean interface to complex system
- **Decoupling**: Clients don't depend on complex subsystem
- **Unified Entry Point**: Single point to access multiple components
- **Layer of Abstraction**: Hides implementation details
- **Coordination**: Orchestrates interactions between subsystems

## Problem It Solves
- Complex subsystems with many interdependent classes
- Need to provide simple interface to clients
- Decoupling client code from internal complexity
- Reducing number of classes clients need to interact with
- Easy integration of large components

## Generic Implementation

### Basic Facade Pattern

```csharp
// Complex subsystem - Multiple classes with dependencies
public class DatabaseConnection
{
    public void Connect() => Console.WriteLine("Connecting to database...");
    public void Disconnect() => Console.WriteLine("Disconnecting from database...");
}

public class SecurityManager
{
    public void Authenticate(string username, string password)
    {
        Console.WriteLine($"Authenticating user: {username}");
        if (password == "password123")
            Console.WriteLine("Authentication successful");
        else
            Console.WriteLine("Authentication failed");
    }
}

public class Logger
{
    public void Log(string message) => Console.WriteLine($"[LOG] {message}");
}

public class DataProcessor
{
    public void ProcessData(string data)
    {
        Console.WriteLine($"Processing data: {data}");
        System.Threading.Thread.Sleep(100);
        Console.WriteLine("Data processing complete");
    }
}

// Facade - Simplifies the complex subsystem
public class ApplicationFacade
{
    private DatabaseConnection _db;
    private SecurityManager _security;
    private Logger _logger;
    private DataProcessor _processor;

    public ApplicationFacade()
    {
        _db = new DatabaseConnection();
        _security = new SecurityManager();
        _logger = new Logger();
        _processor = new DataProcessor();
    }

    // Simple method that orchestrates complex operations
    public void InitializeAndProcessData(string username, string password, string data)
    {
        _logger.Log("Application initialization started");
        _db.Connect();
        _security.Authenticate(username, password);
        _processor.ProcessData(data);
        _db.Disconnect();
        _logger.Log("Application initialization completed");
    }

    public void Login(string username, string password)
    {
        _security.Authenticate(username, password);
    }

    public void ProcessDataSafely(string username, string password, string data)
    {
        _security.Authenticate(username, password);
        _processor.ProcessData(data);
    }
}

// Usage
var facade = new ApplicationFacade();
facade.InitializeAndProcessData("john", "password123", "Important data");
```

### Layered Facade

```csharp
// Different layers of facades for different complexities

// Low-level components
public class MailService
{
    public void SendEmail(string to, string subject, string body)
        => Console.WriteLine($"Sending email to {to}: {subject}");
}

public class NotificationService
{
    private MailService _mailService = new();

    public void NotifyUser(string email, string message)
    {
        _mailService.SendEmail(email, "Notification", message);
    }
}

// Mid-level facade
public class CommunicationFacade
{
    private NotificationService _notificationService = new();
    private List<string> _subscribers = new();

    public void NotifyAll(string message)
    {
        foreach (var subscriber in _subscribers)
        {
            _notificationService.NotifyUser(subscriber, message);
        }
    }

    public void Subscribe(string email) => _subscribers.Add(email);
}

// High-level facade
public class SystemFacade
{
    private CommunicationFacade _communicationFacade = new();

    public void AnnounceMaintenanceWindow(string message)
    {
        _communicationFacade.NotifyAll($"MAINTENANCE: {message}");
    }

    public void AddSubscriber(string email)
    {
        _communicationFacade.Subscribe(email);
    }
}

// Usage
var systemFacade = new SystemFacade();
systemFacade.AddSubscriber("user@example.com");
systemFacade.AnnounceMaintenanceWindow("Server down for 2 hours");
```

## Real-World Examples

### Home Theater Facade
```csharp
// Complex subsystem components
public class Amplifier
{
    public void On() => Console.WriteLine("Amplifier is on");
    public void Off() => Console.WriteLine("Amplifier is off");
    public void SetVolume(int level) => Console.WriteLine($"Volume set to {level}");
}

public class DVDPlayer
{
    public void On() => Console.WriteLine("DVD Player is on");
    public void Off() => Console.WriteLine("DVD Player is off");
    public void Play(string movie) => Console.WriteLine($"Playing {movie}");
}

public class Lights
{
    public void Dim(int level) => Console.WriteLine($"Lights dimmed to {level}%");
    public void On() => Console.WriteLine("Lights on");
}

public class Screen
{
    public void Down() => Console.WriteLine("Screen down");
    public void Up() => Console.WriteLine("Screen up");
}

public class PopcornMaker
{
    public void On() => Console.WriteLine("Popcorn maker on");
    public void Off() => Console.WriteLine("Popcorn maker off");
}

// Facade - Simple interface to complex home theater system
public class HomeTheaterFacade
{
    private Amplifier _amplifier = new();
    private DVDPlayer _dvdPlayer = new();
    private Lights _lights = new();
    private Screen _screen = new();
    private PopcornMaker _popcornMaker = new();

    public void WatchMovie(string movieName)
    {
        Console.WriteLine("Get ready to watch a movie...\n");
        _lights.Dim(10);
        _screen.Down();
        _popcornMaker.On();
        _amplifier.On();
        _amplifier.SetVolume(5);
        _dvdPlayer.On();
        _dvdPlayer.Play(movieName);
    }

    public void EndMovie()
    {
        Console.WriteLine("Shutting down home theater...\n");
        _lights.On();
        _screen.Up();
        _popcornMaker.Off();
        _amplifier.Off();
        _dvdPlayer.Off();
    }
}

// Usage
var homeTheater = new HomeTheaterFacade();
homeTheater.WatchMovie("Inception");
// ... watching movie ...
homeTheater.EndMovie();
```

### Hotel Reservation System Facade
```csharp
// Subsystem classes
public class RoomService
{
    public Room FindRoom(int days) => new Room { RoomNumber = 101, PricePerNight = 100 };
}

public class PaymentService
{
    public bool ProcessPayment(decimal amount) 
    {
        Console.WriteLine($"Processing payment of ${amount}");
        return true;
    }
}

public class NotificationService
{
    public void SendConfirmation(string email, string reservationId)
    {
        Console.WriteLine($"Confirmation sent to {email} for reservation {reservationId}");
    }
}

public class Room
{
    public int RoomNumber { get; set; }
    public decimal PricePerNight { get; set; }
}

public class Reservation
{
    public string ReservationId { get; set; }
    public Room Room { get; set; }
    public int Days { get; set; }
}

// Facade
public class HotelReservationFacade
{
    private RoomService _roomService = new();
    private PaymentService _paymentService = new();
    private NotificationService _notificationService = new();

    public Reservation BookRoom(int days, string email)
    {
        Room room = _roomService.FindRoom(days);
        decimal cost = room.PricePerNight * days;

        if (!_paymentService.ProcessPayment(cost))
            throw new Exception("Payment failed");

        var reservation = new Reservation
        {
            ReservationId = Guid.NewGuid().ToString(),
            Room = room,
            Days = days
        };

        _notificationService.SendConfirmation(email, reservation.ReservationId);
        return reservation;
    }
}

// Usage
var hotelFacade = new HotelReservationFacade();
var reservation = hotelFacade.BookRoom(3, "guest@example.com");
Console.WriteLine($"Room {reservation.Room.RoomNumber} reserved for {reservation.Days} days");
```

### Report Generation Facade
```csharp
public class DataFetcher
{
    public List<string> GetData() => new List<string> { "Data1", "Data2", "Data3" };
}

public class DataProcessor
{
    public string ProcessData(List<string> data)
    {
        return string.Join(", ", data.Select(d => d.ToUpper()));
    }
}

public class ReportFormatter
{
    public string FormatAsHTML(string data) => $"<html><body>{data}</body></html>";
    public string FormatAsCSV(string data) => data;
    public string FormatAsJSON(string data) => $"{{\"data\": \"{data}\"}}";
}

public class ReportGeneratorFacade
{
    private DataFetcher _fetcher = new();
    private DataProcessor _processor = new();
    private ReportFormatter _formatter = new();

    public string GenerateReport(string format)
    {
        var data = _fetcher.GetData();
        var processed = _processor.ProcessData(data);
        
        return format.ToLower() switch
        {
            "html" => _formatter.FormatAsHTML(processed),
            "csv" => _formatter.FormatAsCSV(processed),
            "json" => _formatter.FormatAsJSON(processed),
            _ => throw new ArgumentException("Unknown format")
        };
    }
}

// Usage
var reportFacade = new ReportGeneratorFacade();
Console.WriteLine("HTML Report:\n" + reportFacade.GenerateReport("html"));
Console.WriteLine("\nJSON Report:\n" + reportFacade.GenerateReport("json"));
```

## Facade vs Other Patterns

| Pattern | Purpose | Complexity |
|---------|---------|-----------|
| **Facade** | Simplify complex subsystem | Reduces from many to one |
| **Adapter** | Convert incompatible interfaces | Bridges two systems |
| **Decorator** | Add features to objects | Keeps same interface |
| **Bridge** | Decouple abstraction from implementation | Separates concerns |

## Advantages

✅ **Simplification**: Hides complexity from clients  
✅ **Decoupling**: Clients don't depend on subsystem details  
✅ **Single Entry Point**: Easy to locate and modify logic  
✅ **Flexible**: Subsystem can change without affecting clients  
✅ **Reusable**: Facade can support many different clients  

## Disadvantages

❌ **God Object**: Facade can become too large  
❌ **Hides Options**: Clients can't access subsystem granularly  
❌ **Extra Class**: Adds abstraction layer  
❌ **Maintenance**: Must maintain facade alongside subsystem  

## When to Use

- Simplifying complex subsystems
- Reducing dependencies between client and subsystem
- Grouping related operations into fewer methods
- Decoupling client code from component libraries
- Creating entry points to multi-layered systems

## When NOT to Use

- Subsystem is already simple
- Clients need fine-grained control
- Facade would access few subsystem components
- When adapter or other patterns are more appropriate

## Best Practices

1. **Single Responsibility**: Each facade method should be cohesive
2. **Don't Over-Simplify**: Clients may need subsystem access
3. **Document Behavior**: Clarify what each facade method does
4. **Create Layered Facades**: Different levels of abstraction for different clients
5. **Maintain Subsystem**: Don't let facade become out of sync

## Summary

The Facade Pattern is invaluable for hiding complexity and providing a clean interface to clients. It's especially useful when integrating large libraries or managing complex internal systems, allowing clients to interact through a simplified, unified interface.
