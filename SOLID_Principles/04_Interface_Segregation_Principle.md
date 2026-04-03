# SOLID Principles - Interface Segregation Principle (ISP)

## What is the Interface Segregation Principle (ISP)?

**Question:** What does the Interface Segregation Principle state?

**Answer:** The Interface Segregation Principle states that **clients should not be forced to depend on interfaces they do not use**. This means that large, monolithic interfaces should be broken down into smaller, more specific interfaces so that clients only need to know about the methods that are of interest to them.

## Why is ISP Important?

**Question:** Why should we follow the Interface Segregation Principle?

**Answer:** ISP is important because:
- **Reduced coupling**: Clients depend only on what they need
- **Easier maintenance**: Changes to one interface don't affect unrelated clients
- **Better testability**: Smaller interfaces are easier to mock and test
- **Increased flexibility**: Clients can implement only relevant methods
- **Clearer contracts**: Interfaces represent specific responsibilities

## Real-World Analogy

**Question:** Can you give a real-world analogy for ISP?

**Answer:** Think of a smartphone. You don't need all features for basic calling. Different users need different capabilities:
- Basic user: Just calling and texting
- Photographer: Camera features
- Business user: Email and calendar
- Gamer: Gaming features

A "do everything" interface would force everyone to learn all features.

## Violation Example

**Question:** What does code violating ISP look like?

**Answer:** Here's a classic ISP violation:

```csharp
// VIOLATION: Fat interface with multiple responsibilities
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void Code();
    void Test();
    void Design();
    void Manage();
}

// PROBLEM: Different workers need different methods
public class Developer : IWorker
{
    public void Work() { Console.WriteLine("Developing software"); }
    public void Eat() { Console.WriteLine("Eating lunch"); }
    public void Sleep() { Console.WriteLine("Sleeping"); }
    public void Code() { Console.WriteLine("Writing code"); }
    public void Test() { Console.WriteLine("Writing tests"); }

    // FORCED to implement methods they don't need
    public void Design() { throw new NotImplementedException(); }
    public void Manage() { throw new NotImplementedException(); }
}

public class Manager : IWorker
{
    public void Work() { Console.WriteLine("Managing team"); }
    public void Eat() { Console.WriteLine("Eating lunch"); }
    public void Sleep() { Console.WriteLine("Sleeping"); }
    public void Manage() { Console.WriteLine("Managing projects"); }

    // FORCED to implement methods they don't need
    public void Code() { throw new NotImplementedException(); }
    public void Test() { throw new NotImplementedException(); }
    public void Design() { throw new NotImplementedException(); }
}

public class Robot : IWorker
{
    public void Work() { Console.WriteLine("Working 24/7"); }
    public void Code() { Console.WriteLine("Processing code"); }
    public void Test() { Console.WriteLine("Running automated tests"); }

    // ROBOTS don't eat or sleep!
    public void Eat() { throw new NotImplementedException(); }
    public void Sleep() { throw new NotImplementedException(); }
    public void Design() { throw new NotImplementedException(); }
    public void Manage() { throw new NotImplementedException(); }
}
```

**Why this violates ISP:**
- Clients are forced to implement methods they don't use
- NotImplementedException indicates poor design
- Changes to one method affect all implementers
- Hard to understand what each class actually does

## Correct Implementation

**Question:** How should we refactor the above code to follow ISP?

**Answer:** We need to break down the fat interface into smaller, focused interfaces:

```csharp
// Segregated interfaces based on responsibilities
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public interface ISleepable
{
    void Sleep();
}

public interface ICodeable
{
    void Code();
}

public interface ITestable
{
    void Test();
}

public interface IDesignable
{
    void Design();
}

public interface IManageable
{
    void Manage();
}

// Implement only what each class needs
public class Developer : IWorkable, IEatable, ISleepable, ICodeable, ITestable
{
    public void Work() { Console.WriteLine("Developing software"); }
    public void Eat() { Console.WriteLine("Eating lunch"); }
    public void Sleep() { Console.WriteLine("Sleeping"); }
    public void Code() { Console.WriteLine("Writing code"); }
    public void Test() { Console.WriteLine("Writing tests"); }
}

public class Manager : IWorkable, IEatable, ISleepable, IManageable
{
    public void Work() { Console.WriteLine("Managing team"); }
    public void Eat() { Console.WriteLine("Eating lunch"); }
    public void Sleep() { Console.WriteLine("Sleeping"); }
    public void Manage() { Console.WriteLine("Managing projects"); }
}

public class Robot : IWorkable, ICodeable, ITestable
{
    public void Work() { Console.WriteLine("Working 24/7"); }
    public void Code() { Console.WriteLine("Processing code"); }
    public void Test() { Console.WriteLine("Running automated tests"); }
}

// Client code that depends only on what it needs
public class WorkerService
{
    public void ManageWorkers(IEnumerable<IWorkable> workers)
    {
        foreach (var worker in workers)
        {
            worker.Work();
        }
    }
}

public class CodingService
{
    public void RunCodingTasks(IEnumerable<ICodeable> codeables)
    {
        foreach (var coder in codeables)
        {
            coder.Code();
        }
    }
}
```

## Real-Time Application Example

**Question:** Can you show a real-time application of ISP in a multi-purpose device system?

**Answer:** Let's build a smart home device system that follows ISP:

```csharp
// Segregated interfaces for different device capabilities
public interface IPowerable
{
    void TurnOn();
    void TurnOff();
    bool IsOn { get; }
}

public interface IDimmable
{
    void SetBrightness(int level);
    int GetBrightness();
}

public interface ITemperatureControllable
{
    void SetTemperature(int temperature);
    int GetTemperature();
}

public interface IRecordable
{
    void StartRecording();
    void StopRecording();
    bool IsRecording { get; }
}

public interface IPlayable
{
    void Play();
    void Pause();
    void Stop();
    bool IsPlaying { get; }
}

public interface IVolumeControllable
{
    void SetVolume(int level);
    int GetVolume();
}

// Smart home devices implementing only relevant interfaces
public class SmartLight : IPowerable, IDimmable
{
    private bool _isOn;
    private int _brightness;

    public void TurnOn() { _isOn = true; Console.WriteLine("Light turned on"); }
    public void TurnOff() { _isOn = false; Console.WriteLine("Light turned off"); }
    public bool IsOn => _isOn;

    public void SetBrightness(int level)
    {
        _brightness = Math.Clamp(level, 0, 100);
        Console.WriteLine($"Brightness set to {_brightness}%");
    }

    public int GetBrightness() => _brightness;
}

public class SmartThermostat : IPowerable, ITemperatureControllable
{
    private bool _isOn;
    private int _temperature;

    public void TurnOn() { _isOn = true; Console.WriteLine("Thermostat turned on"); }
    public void TurnOff() { _isOn = false; Console.WriteLine("Thermostat turned off"); }
    public bool IsOn => _isOn;

    public void SetTemperature(int temperature)
    {
        _temperature = Math.Clamp(temperature, 50, 90);
        Console.WriteLine($"Temperature set to {_temperature}°F");
    }

    public int GetTemperature() => _temperature;
}

public class SmartSpeaker : IPowerable, IPlayable, IVolumeControllable
{
    private bool _isOn;
    private bool _isPlaying;
    private int _volume;

    public void TurnOn() { _isOn = true; Console.WriteLine("Speaker turned on"); }
    public void TurnOff() { _isOn = false; Console.WriteLine("Speaker turned off"); }
    public bool IsOn => _isOn;

    public void Play() { _isPlaying = true; Console.WriteLine("Music playing"); }
    public void Pause() { _isPlaying = false; Console.WriteLine("Music paused"); }
    public void Stop() { _isPlaying = false; Console.WriteLine("Music stopped"); }
    public bool IsPlaying => _isPlaying;

    public void SetVolume(int level)
    {
        _volume = Math.Clamp(level, 0, 100);
        Console.WriteLine($"Volume set to {_volume}%");
    }

    public int GetVolume() => _volume;
}

public class SmartCamera : IPowerable, IRecordable
{
    private bool _isOn;
    private bool _isRecording;

    public void TurnOn() { _isOn = true; Console.WriteLine("Camera turned on"); }
    public void TurnOff() { _isOn = false; Console.WriteLine("Camera turned off"); }
    public bool IsOn => _isOn;

    public void StartRecording() { _isRecording = true; Console.WriteLine("Recording started"); }
    public void StopRecording() { _isRecording = false; Console.WriteLine("Recording stopped"); }
    public bool IsRecording => _isRecording;
}

public class SmartTV : IPowerable, IPlayable, IVolumeControllable, IRecordable
{
    private bool _isOn;
    private bool _isPlaying;
    private bool _isRecording;
    private int _volume;

    public void TurnOn() { _isOn = true; Console.WriteLine("TV turned on"); }
    public void TurnOff() { _isOn = false; Console.WriteLine("TV turned off"); }
    public bool IsOn => _isOn;

    public void Play() { _isPlaying = true; Console.WriteLine("TV playing"); }
    public void Pause() { _isPlaying = false; Console.WriteLine("TV paused"); }
    public void Stop() { _isPlaying = false; Console.WriteLine("TV stopped"); }
    public bool IsPlaying => _isPlaying;

    public void SetVolume(int level)
    {
        _volume = Math.Clamp(level, 0, 100);
        Console.WriteLine($"TV volume set to {_volume}%");
    }

    public int GetVolume() => _volume;

    public void StartRecording() { _isRecording = true; Console.WriteLine("TV recording started"); }
    public void StopRecording() { _isRecording = false; Console.WriteLine("TV recording stopped"); }
    public bool IsRecording => _isRecording;
}

// Client code that depends only on specific capabilities
public class HomeAutomationController
{
    public void TurnAllDevicesOn(IEnumerable<IPowerable> devices)
    {
        foreach (var device in devices)
        {
            device.TurnOn();
        }
    }

    public void AdjustLights(IEnumerable<IDimmable> lights)
    {
        foreach (var light in lights)
        {
            light.SetBrightness(75);
        }
    }

    public void ControlTemperature(IEnumerable<ITemperatureControllable> thermostats)
    {
        foreach (var thermostat in thermostats)
        {
            thermostat.SetTemperature(72);
        }
    }

    public void StartRecording(IEnumerable<IRecordable> recorders)
    {
        foreach (var recorder in recorders)
        {
            recorder.StartRecording();
        }
    }

    public void PlayMedia(IEnumerable<IPlayable> players)
    {
        foreach (var player in players)
        {
            player.Play();
        }
    }
}

// Usage example
public class SmartHomeExample
{
    public static void Main()
    {
        // Create devices
        var devices = new List<object>
        {
            new SmartLight(),
            new SmartThermostat(),
            new SmartSpeaker(),
            new SmartCamera(),
            new SmartTV()
        };

        var controller = new HomeAutomationController();

        // Use devices based on their capabilities
        controller.TurnAllDevicesOn(devices.OfType<IPowerable>());
        controller.AdjustLights(devices.OfType<IDimmable>());
        controller.ControlTemperature(devices.OfType<ITemperatureControllable>());
        controller.StartRecording(devices.OfType<IRecordable>());
        controller.PlayMedia(devices.OfType<IPlayable>());
    }
}
```

## Role Interfaces Pattern

**Question:** What is the Role Interfaces pattern and how does it help ISP?

**Answer:** Role Interfaces define interfaces based on roles rather than technical capabilities:

```csharp
// Role-based interfaces
public interface IRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Save(T entity);
    void Delete(int id);
}

public interface IUnitOfWork
{
    void BeginTransaction();
    void Commit();
    void Rollback();
}

public interface IEmailSender
{
    void SendEmail(string to, string subject, string body);
}

public interface ILogger
{
    void LogInformation(string message);
    void LogError(string message);
    void LogWarning(string message);
}

// Classes implement multiple roles as needed
public class UserService : IRepository<User>, IEmailSender, ILogger
{
    public User GetById(int id) { /* implementation */ }
    public IEnumerable<User> GetAll() { /* implementation */ }
    public void Save(User entity) { /* implementation */ }
    public void Delete(int id) { /* implementation */ }

    public void SendEmail(string to, string subject, string body) { /* implementation */ }

    public void LogInformation(string message) { /* implementation */ }
    public void LogError(string message) { /* implementation */ }
    public void LogWarning(string message) { /* implementation */ }
}

// Clients depend on specific roles
public class UserController
{
    private readonly IRepository<User> _userRepository;
    private readonly IEmailSender _emailSender;
    private readonly ILogger _logger;

    public UserController(IRepository<User> userRepository,
                         IEmailSender emailSender,
                         ILogger logger)
    {
        _userRepository = userRepository;
        _emailSender = emailSender;
        _logger = logger;
    }
}
```

## ISP and Dependency Injection

**Question:** How does ISP work with Dependency Injection?

**Answer:** ISP and DI work together to create loosely coupled systems:

```csharp
// Multiple small interfaces
public interface IOrderValidator { bool Validate(Order order); }
public interface IOrderProcessor { ProcessResult Process(Order order); }
public interface IOrderRepository { void Save(Order order); }
public interface IEmailService { void SendOrderConfirmation(Order order); }

// Service that depends on specific interfaces
public class OrderService
{
    private readonly IOrderValidator _validator;
    private readonly IOrderProcessor _processor;
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;

    public OrderService(IOrderValidator validator,
                       IOrderProcessor processor,
                       IOrderRepository repository,
                       IEmailService emailService)
    {
        _validator = validator;
        _processor = processor;
        _repository = repository;
        _emailService = emailService;
    }

    public void PlaceOrder(Order order)
    {
        if (_validator.Validate(order))
        {
            var result = _processor.Process(order);
            _repository.Save(order);
            _emailService.SendOrderConfirmation(order);
        }
    }
}

// DI container configuration
public class DependencyConfig
{
    public static IServiceProvider ConfigureServices()
    {
        var services = new ServiceCollection();

        // Register implementations
        services.AddTransient<IOrderValidator, OrderValidator>();
        services.AddTransient<IOrderProcessor, OrderProcessor>();
        services.AddTransient<IOrderRepository, OrderRepository>();
        services.AddTransient<IEmailService, EmailService>();

        // Register service
        services.AddTransient<OrderService>();

        return services.BuildServiceProvider();
    }
}
```

## Common ISP Violations

**Question:** What are common signs that ISP is being violated?

**Answer:**

1. **NotImplementedException**: Classes throwing this exception
2. **Empty method implementations**: Methods that do nothing
3. **Large interfaces**: Interfaces with many methods
4. **Client classes implementing many interfaces**: Classes implementing interfaces they don't fully use
5. **Methods with multiple parameters**: Methods that take objects with many properties the client doesn't care about

## How to Identify Interface Segregation

**Question:** How do you identify when to segregate interfaces?

**Answer:**

1. **Client analysis**: Look at how different clients use the interface
2. **Method cohesion**: Group methods that are used together
3. **Change patterns**: Methods that change together should be in same interface
4. **Role stereotypes**: Define interfaces based on roles (Repository, Service, etc.)
5. **Dependency analysis**: See what clients actually depend on

## ISP and Other SOLID Principles

**Question:** How does ISP relate to other SOLID principles?

**Answer:**

- **SRP**: Smaller interfaces help classes have single responsibility
- **OCP**: Smaller interfaces are easier to extend
- **LSP**: Subtypes can implement smaller interfaces more easily
- **DIP**: Clients depend on specific small interfaces rather than large ones

## Summary

**Question:** What's the key takeaway about ISP?

**Answer:** The Interface Segregation Principle is about **creating focused, client-specific interfaces**. Don't force clients to depend on methods they don't use. Break down large interfaces into smaller, more specific ones that match client needs.

**Key Points:**
- Clients should not depend on unused methods
- Segregate interfaces based on client needs
- Use Role Interfaces pattern
- Smaller interfaces are easier to implement and maintain
- ISP enables better dependency management