# SOLID Principles - Dependency Inversion Principle (DIP)

## What is the Dependency Inversion Principle (DIP)?

**Question:** What does the Dependency Inversion Principle state?

**Answer:** The Dependency Inversion Principle states that:
1. **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
2. **Abstractions should not depend on details. Details should depend on abstractions.**

In simpler terms: **Depend on abstractions, not concretions.**

## Why is DIP Important?

**Question:** Why should we follow the Dependency Inversion Principle?

**Answer:** DIP is important because:
- **Decoupling**: High-level and low-level modules are loosely coupled
- **Testability**: Easy to test with mock implementations
- **Flexibility**: Can change implementations without affecting high-level code
- **Maintainability**: Changes to low-level modules don't break high-level code
- **Reusability**: Abstractions can be reused across different implementations

## Real-World Analogy

**Question:** Can you give a real-world analogy for DIP?

**Answer:** Think of electrical plugs and sockets. The socket (high-level module) doesn't depend on specific devices (low-level modules). Both depend on the electrical standard (abstraction). You can plug in lamps, TVs, or computers without changing the socket.

## Violation Example

**Question:** What does code violating DIP look like?

**Answer:** Here's a classic DIP violation:

```csharp
// VIOLATION: High-level module depends on low-level module
public class OrderService
{
    private readonly EmailService _emailService; // Direct dependency on concrete class

    public OrderService()
    {
        _emailService = new EmailService(); // Tight coupling
    }

    public void ProcessOrder(Order order)
    {
        // Process order logic...

        // High-level module directly uses low-level module
        _emailService.SendOrderConfirmation(order.CustomerEmail, "Order Confirmed", "Your order has been processed");
    }
}

// Low-level module
public class EmailService
{
    public void SendOrderConfirmation(string to, string subject, string body)
    {
        // Email sending implementation
        Console.WriteLine($"Sending email to {to}: {subject}");
    }
}
```

**Why this violates DIP:**
- OrderService (high-level) depends directly on EmailService (low-level)
- Can't test OrderService without EmailService
- Can't change email implementation without modifying OrderService
- Tight coupling makes the system rigid

## Correct Implementation

**Question:** How should we refactor the above code to follow DIP?

**Answer:** We need to introduce abstractions and dependency injection:

```csharp
// Abstraction that both high-level and low-level modules depend on
public interface IEmailService
{
    void SendOrderConfirmation(string to, string subject, string body);
    void SendShippingNotification(string to, string subject, string body);
}

// High-level module depends on abstraction
public class OrderService
{
    private readonly IEmailService _emailService;

    // Dependency injection through constructor
    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void ProcessOrder(Order order)
    {
        // Process order logic...

        // Use abstraction, not concrete implementation
        _emailService.SendOrderConfirmation(
            order.CustomerEmail,
            "Order Confirmed",
            "Your order has been processed"
        );
    }
}

// Low-level module implements abstraction
public class SmtpEmailService : IEmailService
{
    public void SendOrderConfirmation(string to, string subject, string body)
    {
        // SMTP implementation
        Console.WriteLine($"SMTP: Sending email to {to}: {subject}");
    }

    public void SendShippingNotification(string to, string subject, string body)
    {
        // SMTP implementation
        Console.WriteLine($"SMTP: Sending shipping notification to {to}: {subject}");
    }
}

// Another implementation
public class MockEmailService : IEmailService
{
    public void SendOrderConfirmation(string to, string subject, string body)
    {
        // Mock implementation for testing
        Console.WriteLine($"MOCK: Would send email to {to}: {subject}");
    }

    public void SendShippingNotification(string to, string subject, string body)
    {
        // Mock implementation for testing
        Console.WriteLine($"MOCK: Would send shipping notification to {to}: {subject}");
    }
}
```

## Real-Time Application Example

**Question:** Can you show a real-time application of DIP in an e-commerce system?

**Answer:** Let's build a complete e-commerce system following DIP:

```csharp
// Domain entities
public class Order
{
    public int Id { get; set; }
    public string CustomerEmail { get; set; }
    public List<OrderItem> Items { get; set; } = new();
    public decimal TotalAmount { get; set; }
    public OrderStatus Status { get; set; }
}

public class OrderItem
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public enum OrderStatus { Pending, Confirmed, Shipped, Delivered }

// Abstractions (interfaces)
public interface IOrderRepository
{
    Order GetById(int id);
    void Save(Order order);
    IEnumerable<Order> GetPendingOrders();
}

public interface IPaymentService
{
    PaymentResult ProcessPayment(Order order);
    void RefundPayment(string transactionId);
}

public interface IEmailService
{
    void SendOrderConfirmation(Order order);
    void SendShippingNotification(Order order);
}

public interface IInventoryService
{
    bool CheckStock(int productId, int quantity);
    void ReserveStock(int productId, int quantity);
    void ReleaseStock(int productId, int quantity);
}

public interface ILogger
{
    void LogInformation(string message);
    void LogError(string message, Exception ex = null);
}

// Payment result
public class PaymentResult
{
    public bool Success { get; set; }
    public string TransactionId { get; set; }
    public string ErrorMessage { get; set; }
}

// High-level module: Order processing service
public class OrderProcessingService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;
    private readonly IEmailService _emailService;
    private readonly IInventoryService _inventoryService;
    private readonly ILogger _logger;

    public OrderProcessingService(
        IOrderRepository orderRepository,
        IPaymentService paymentService,
        IEmailService emailService,
        IInventoryService inventoryService,
        ILogger logger)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
        _emailService = emailService;
        _inventoryService = inventoryService;
        _logger = logger;
    }

    public OrderResult ProcessOrder(Order order)
    {
        try
        {
            _logger.LogInformation($"Processing order {order.Id}");

            // Validate inventory
            foreach (var item in order.Items)
            {
                if (!_inventoryService.CheckStock(item.ProductId, item.Quantity))
                {
                    return new OrderResult
                    {
                        Success = false,
                        ErrorMessage = $"Insufficient stock for {item.ProductName}"
                    };
                }
            }

            // Reserve inventory
            foreach (var item in order.Items)
            {
                _inventoryService.ReserveStock(item.ProductId, item.Quantity);
            }

            // Process payment
            var paymentResult = _paymentService.ProcessPayment(order);
            if (!paymentResult.Success)
            {
                // Release reserved stock
                foreach (var item in order.Items)
                {
                    _inventoryService.ReleaseStock(item.ProductId, item.Quantity);
                }

                return new OrderResult
                {
                    Success = false,
                    ErrorMessage = paymentResult.ErrorMessage
                };
            }

            // Update order status
            order.Status = OrderStatus.Confirmed;
            order.TotalAmount = order.Items.Sum(item => item.Quantity * item.UnitPrice);

            // Save order
            _orderRepository.Save(order);

            // Send confirmation email
            _emailService.SendOrderConfirmation(order);

            _logger.LogInformation($"Order {order.Id} processed successfully");

            return new OrderResult
            {
                Success = true,
                Order = order,
                TransactionId = paymentResult.TransactionId
            };
        }
        catch (Exception ex)
        {
            _logger.LogError($"Error processing order {order.Id}", ex);
            return new OrderResult
            {
                Success = false,
                ErrorMessage = "System error occurred"
            };
        }
    }
}

public class OrderResult
{
    public bool Success { get; set; }
    public Order Order { get; set; }
    public string TransactionId { get; set; }
    public string ErrorMessage { get; set; }
}

// Low-level implementations
public class DatabaseOrderRepository : IOrderRepository
{
    public Order GetById(int id)
    {
        // Database implementation
        Console.WriteLine($"Fetching order {id} from database");
        return new Order { Id = id, Status = OrderStatus.Pending };
    }

    public void Save(Order order)
    {
        // Database implementation
        Console.WriteLine($"Saving order {order.Id} to database");
    }

    public IEnumerable<Order> GetPendingOrders()
    {
        // Database implementation
        Console.WriteLine("Fetching pending orders from database");
        return new List<Order>();
    }
}

public class StripePaymentService : IPaymentService
{
    public PaymentResult ProcessPayment(Order order)
    {
        // Stripe API implementation
        Console.WriteLine($"Processing payment via Stripe: ${order.TotalAmount}");
        return new PaymentResult
        {
            Success = true,
            TransactionId = "stripe_txn_" + Guid.NewGuid().ToString()
        };
    }

    public void RefundPayment(string transactionId)
    {
        // Stripe refund implementation
        Console.WriteLine($"Refunding payment: {transactionId}");
    }
}

public class SendGridEmailService : IEmailService
{
    public void SendOrderConfirmation(Order order)
    {
        // SendGrid implementation
        Console.WriteLine($"SendGrid: Sending order confirmation to {order.CustomerEmail}");
    }

    public void SendShippingNotification(Order order)
    {
        // SendGrid implementation
        Console.WriteLine($"SendGrid: Sending shipping notification to {order.CustomerEmail}");
    }
}

public class WarehouseInventoryService : IInventoryService
{
    public bool CheckStock(int productId, int quantity)
    {
        // Warehouse system implementation
        Console.WriteLine($"Checking stock for product {productId}, quantity {quantity}");
        return true; // Simplified
    }

    public void ReserveStock(int productId, int quantity)
    {
        // Warehouse system implementation
        Console.WriteLine($"Reserving {quantity} units of product {productId}");
    }

    public void ReleaseStock(int productId, int quantity)
    {
        // Warehouse system implementation
        Console.WriteLine($"Releasing {quantity} units of product {productId}");
    }
}

public class ConsoleLogger : ILogger
{
    public void LogInformation(string message)
    {
        Console.WriteLine($"INFO: {message}");
    }

    public void LogError(string message, Exception ex = null)
    {
        Console.WriteLine($"ERROR: {message}");
        if (ex != null)
        {
            Console.WriteLine($"Exception: {ex.Message}");
        }
    }
}

// Dependency injection container (simplified)
public class ServiceLocator
{
    private static readonly Dictionary<Type, object> _services = new();

    static ServiceLocator()
    {
        // Register implementations
        _services[typeof(IOrderRepository)] = new DatabaseOrderRepository();
        _services[typeof(IPaymentService)] = new StripePaymentService();
        _services[typeof(IEmailService)] = new SendGridEmailService();
        _services[typeof(IInventoryService)] = new WarehouseInventoryService();
        _services[typeof(ILogger)] = new ConsoleLogger();
    }

    public static T GetService<T>()
    {
        return (T)_services[typeof(T)];
    }

    public static OrderProcessingService CreateOrderProcessingService()
    {
        return new OrderProcessingService(
            GetService<IOrderRepository>(),
            GetService<IPaymentService>(),
            GetService<IEmailService>(),
            GetService<IInventoryService>(),
            GetService<ILogger>()
        );
    }
}

// Usage example
public class EcommerceExample
{
    public static void Main()
    {
        // Create order
        var order = new Order
        {
            Id = 123,
            CustomerEmail = "customer@example.com",
            Items = new List<OrderItem>
            {
                new OrderItem { ProductId = 1, ProductName = "Laptop", Quantity = 1, UnitPrice = 999.99m },
                new OrderItem { ProductId = 2, ProductName = "Mouse", Quantity = 2, UnitPrice = 25.00m }
            }
        };

        // Process order using DIP-compliant service
        var orderService = ServiceLocator.CreateOrderProcessingService();
        var result = orderService.ProcessOrder(order);

        if (result.Success)
        {
            Console.WriteLine($"Order processed successfully! Transaction: {result.TransactionId}");
        }
        else
        {
            Console.WriteLine($"Order processing failed: {result.ErrorMessage}");
        }
    }
}
```

## DIP and Dependency Injection Patterns

**Question:** What are different ways to implement DIP?

**Answer:**

### 1. Constructor Injection (Most Common)
```csharp
public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

### 2. Property Injection
```csharp
public class OrderService
{
    public IEmailService EmailService { get; set; }

    public void ProcessOrder(Order order)
    {
        EmailService?.SendOrderConfirmation(order);
    }
}
```

### 3. Method Injection
```csharp
public class OrderService
{
    public void ProcessOrder(Order order, IEmailService emailService)
    {
        emailService.SendOrderConfirmation(order);
    }
}
```

### 4. Service Locator (Anti-pattern if overused)
```csharp
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        var emailService = ServiceLocator.GetService<IEmailService>();
        emailService.SendOrderConfirmation(order);
    }
}
```

## DIP and IoC Containers

**Question:** How do Inversion of Control containers help with DIP?

**Answer:** IoC containers automate dependency injection:

```csharp
// Using Microsoft.Extensions.DependencyInjection
public class Startup
{
    public IServiceProvider ConfigureServices()
    {
        var services = new ServiceCollection();

        // Register abstractions with implementations
        services.AddTransient<IOrderRepository, DatabaseOrderRepository>();
        services.AddTransient<IPaymentService, StripePaymentService>();
        services.AddTransient<IEmailService, SendGridEmailService>();
        services.AddTransient<IInventoryService, WarehouseInventoryService>();
        services.AddTransient<ILogger, ConsoleLogger>();

        // Register high-level service
        services.AddTransient<OrderProcessingService>();

        return services.BuildServiceProvider();
    }
}

public class OrderController
{
    private readonly OrderProcessingService _orderService;

    public OrderController(OrderProcessingService orderService)
    {
        _orderService = orderService;
        // All dependencies automatically resolved by container
    }
}
```

## Testing with DIP

**Question:** How does DIP make testing easier?

**Answer:** DIP enables easy mocking for unit tests:

```csharp
[TestFixture]
public class OrderProcessingServiceTests
{
    private Mock<IOrderRepository> _orderRepositoryMock;
    private Mock<IPaymentService> _paymentServiceMock;
    private Mock<IEmailService> _emailServiceMock;
    private Mock<IInventoryService> _inventoryServiceMock;
    private Mock<ILogger> _loggerMock;
    private OrderProcessingService _service;

    [SetUp]
    public void Setup()
    {
        _orderRepositoryMock = new Mock<IOrderRepository>();
        _paymentServiceMock = new Mock<IPaymentService>();
        _emailServiceMock = new Mock<IEmailService>();
        _inventoryServiceMock = new Mock<IInventoryService>();
        _loggerMock = new Mock<ILogger>();

        _service = new OrderProcessingService(
            _orderRepositoryMock.Object,
            _paymentServiceMock.Object,
            _emailServiceMock.Object,
            _inventoryServiceMock.Object,
            _loggerMock.Object
        );
    }

    [Test]
    public void ProcessOrder_ValidOrder_ReturnsSuccess()
    {
        // Arrange
        var order = new Order { Id = 1, CustomerEmail = "test@example.com" };
        _inventoryServiceMock.Setup(x => x.CheckStock(It.IsAny<int>(), It.IsAny<int>())).Returns(true);
        _paymentServiceMock.Setup(x => x.ProcessPayment(order)).Returns(new PaymentResult { Success = true });

        // Act
        var result = _service.ProcessOrder(order);

        // Assert
        Assert.IsTrue(result.Success);
        _emailServiceMock.Verify(x => x.SendOrderConfirmation(order), Times.Once);
    }
}
```

## Common DIP Violations

**Question:** What are common signs that DIP is being violated?

**Answer:**

1. **New keyword in constructors**: `new ConcreteClass()`
2. **Static method calls**: `StaticClass.Method()`
3. **Concrete class dependencies**: Classes depending on other concrete classes
4. **Tight coupling**: Hard to change implementations
5. **Difficult testing**: Can't easily mock dependencies

## DIP and Other SOLID Principles

**Question:** How does DIP relate to other SOLID principles?

**Answer:**

- **SRP**: DIP helps achieve SRP by allowing classes to focus on their responsibilities
- **OCP**: DIP enables OCP by allowing extension through new implementations
- **LSP**: DIP works with LSP by depending on abstractions that subtypes can implement
- **ISP**: DIP benefits from small, focused interfaces

## Summary

**Question:** What's the key takeaway about DIP?

**Answer:** The Dependency Inversion Principle is about **depending on abstractions rather than concretions**. High-level modules should not depend on low-level modules; both should depend on abstractions. This leads to loosely coupled, testable, and maintainable code.

**Key Points:**
- Depend on abstractions, not concrete classes
- Use dependency injection to invert dependencies
- Abstractions should be defined by the client
- DIP enables better testing and flexibility
- Foundation for modern dependency injection frameworks