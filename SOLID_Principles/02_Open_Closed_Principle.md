# SOLID Principles - Open-Closed Principle (OCP)

## What is the Open-Closed Principle (OCP)?

**Question:** What does the Open-Closed Principle state?

**Answer:** The Open-Closed Principle states that **software entities (classes, modules, functions, etc.) should be open for extension but closed for modification**. This means you should be able to add new functionality without changing existing code.

## Why is OCP Important?

**Question:** Why should we follow the Open-Closed Principle?

**Answer:** OCP is important because:
- **Stability**: Existing code remains unchanged and tested
- **Extensibility**: New features can be added easily
- **Maintainability**: Less risk of introducing bugs in existing code
- **Flexibility**: System can evolve without major rewrites
- **Testing**: Existing functionality doesn't need re-testing

## Real-World Analogy

**Question:** Can you give a real-world analogy for OCP?

**Answer:** Think of a USB port. The USB standard is "closed for modification" - you can't change how USB works. But it's "open for extension" - you can plug in new devices (keyboards, mice, hard drives) without changing the USB port itself.

## Violation Example

**Question:** What does code violating OCP look like?

**Answer:** Here's a class that violates OCP:

```csharp
// VIOLATION: Modifying existing code for new requirements
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType, decimal amount)
    {
        if (paymentType == "CreditCard")
        {
            // Credit card processing logic
            Console.WriteLine($"Processing credit card payment: ${amount}");
        }
        else if (paymentType == "PayPal")
        {
            // PayPal processing logic
            Console.WriteLine($"Processing PayPal payment: ${amount}");
        }
        else if (paymentType == "BankTransfer")
        {
            // Bank transfer processing logic
            Console.WriteLine($"Processing bank transfer: ${amount}");
        }
        // PROBLEM: To add a new payment type, we must modify this class
    }
}
```

**Why this violates OCP:**
- Every new payment type requires modifying the existing method
- Risk of breaking existing payment types
- Method becomes longer and more complex
- Harder to test and maintain

## Correct Implementation

**Question:** How should we refactor the above code to follow OCP?

**Answer:** We need to use abstraction and polymorphism:

```csharp
// Abstraction for payment processing
public interface IPaymentMethod
{
    void ProcessPayment(decimal amount);
    bool IsValidPaymentType(string type);
}

// Concrete implementations
public class CreditCardPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        // Credit card specific logic
        Console.WriteLine($"Processing credit card payment: ${amount}");
        // Validate card number, expiry, CVV, etc.
    }

    public bool IsValidPaymentType(string type) => type == "CreditCard";
}

public class PayPalPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        // PayPal specific logic
        Console.WriteLine($"Processing PayPal payment: ${amount}");
        // PayPal API integration
    }

    public bool IsValidPaymentType(string type) => type == "PayPal";
}

public class BankTransferPayment : IPaymentMethod
{
    public void ProcessPayment(decimal amount)
    {
        // Bank transfer specific logic
        Console.WriteLine($"Processing bank transfer: ${amount}");
        // Bank API integration
    }

    public bool IsValidPaymentType(string type) => type == "BankTransfer";
}

// OCP-compliant payment processor
public class PaymentProcessor
{
    private readonly List<IPaymentMethod> _paymentMethods;

    public PaymentProcessor()
    {
        _paymentMethods = new List<IPaymentMethod>
        {
            new CreditCardPayment(),
            new PayPalPayment(),
            new BankTransferPayment()
        };
    }

    public void ProcessPayment(string paymentType, decimal amount)
    {
        var paymentMethod = _paymentMethods.FirstOrDefault(p => p.IsValidPaymentType(paymentType));

        if (paymentMethod == null)
        {
            throw new InvalidOperationException($"Unsupported payment type: {paymentType}");
        }

        paymentMethod.ProcessPayment(amount);
    }

    // EXTENSION POINT: Add new payment methods without modifying existing code
    public void AddPaymentMethod(IPaymentMethod paymentMethod)
    {
        _paymentMethods.Add(paymentMethod);
    }
}
```

## Real-Time Application Example

**Question:** Can you show a real-time application of OCP in a notification system?

**Answer:** Let's build a notification system that can send different types of notifications:

```csharp
// Core notification abstraction
public interface INotificationService
{
    void Send(NotificationMessage message);
    NotificationType SupportedType { get; }
}

public class NotificationMessage
{
    public string To { get; set; }
    public string Subject { get; set; }
    public string Body { get; set; }
    public NotificationType Type { get; set; }
    public Dictionary<string, string> Metadata { get; set; } = new();
}

public enum NotificationType
{
    Email,
    SMS,
    Push,
    Slack
}

// Email notification implementation
public class EmailNotificationService : INotificationService
{
    public NotificationType SupportedType => NotificationType.Email;

    public void Send(NotificationMessage message)
    {
        // Email-specific implementation
        Console.WriteLine($"Sending EMAIL to {message.To}");
        Console.WriteLine($"Subject: {message.Subject}");
        Console.WriteLine($"Body: {message.Body}");

        // Simulate email sending
        using (var smtpClient = new SmtpClient("smtp.example.com"))
        {
            // Email sending logic
        }
    }
}

// SMS notification implementation
public class SmsNotificationService : INotificationService
{
    public NotificationType SupportedType => NotificationType.SMS;

    public void Send(NotificationMessage message)
    {
        // SMS-specific implementation
        Console.WriteLine($"Sending SMS to {message.To}");
        Console.WriteLine($"Message: {message.Body}");

        // Simulate SMS sending via Twilio or similar
        var smsApi = new SmsApiClient();
        smsApi.SendSms(message.To, message.Body);
    }
}

// Push notification implementation
public class PushNotificationService : INotificationService
{
    public NotificationType SupportedType => NotificationType.Push;

    public void Send(NotificationMessage message)
    {
        // Push notification specific implementation
        Console.WriteLine($"Sending PUSH notification to {message.To}");
        Console.WriteLine($"Title: {message.Subject}");
        Console.WriteLine($"Message: {message.Body}");

        // Simulate push notification via Firebase/APNs
        var pushClient = new PushNotificationClient();
        pushClient.SendPush(message.To, message.Subject, message.Body);
    }
}

// Slack notification implementation
public class SlackNotificationService : INotificationService
{
    public NotificationType SupportedType => NotificationType.Slack;

    public void Send(NotificationMessage message)
    {
        // Slack-specific implementation
        string channel = message.Metadata.GetValueOrDefault("channel", "#general");
        Console.WriteLine($"Sending SLACK message to {channel}");
        Console.WriteLine($"Message: {message.Body}");

        // Simulate Slack API call
        var slackClient = new SlackApiClient();
        slackClient.PostMessage(channel, message.Body);
    }
}

// OCP-compliant notification manager
public class NotificationManager
{
    private readonly Dictionary<NotificationType, INotificationService> _services = new();

    public NotificationManager()
    {
        // Register built-in services
        RegisterService(new EmailNotificationService());
        RegisterService(new SmsNotificationService());
        RegisterService(new PushNotificationService());
        RegisterService(new SlackNotificationService());
    }

    // EXTENSION POINT: Add new notification types without modifying existing code
    public void RegisterService(INotificationService service)
    {
        _services[service.SupportedType] = service;
    }

    public void SendNotification(NotificationMessage message)
    {
        if (_services.TryGetValue(message.Type, out var service))
        {
            service.Send(message);
        }
        else
        {
            throw new InvalidOperationException($"No service registered for notification type: {message.Type}");
        }
    }

    public void SendBulkNotifications(IEnumerable<NotificationMessage> messages)
    {
        foreach (var message in messages)
        {
            SendNotification(message);
        }
    }
}

// Usage example
public class NotificationExample
{
    public static void Main()
    {
        var notificationManager = new NotificationManager();

        // Send different types of notifications
        var messages = new List<NotificationMessage>
        {
            new NotificationMessage
            {
                To = "user@example.com",
                Subject = "Welcome!",
                Body = "Welcome to our platform",
                Type = NotificationType.Email
            },
            new NotificationMessage
            {
                To = "+1234567890",
                Body = "Your order has been shipped",
                Type = NotificationType.SMS
            },
            new NotificationMessage
            {
                To = "device_token_123",
                Subject = "New Message",
                Body = "You have a new message",
                Type = NotificationType.Push
            },
            new NotificationMessage
            {
                To = "#orders",
                Body = "New order received",
                Type = NotificationType.Slack,
                Metadata = new Dictionary<string, string> { ["channel"] = "#orders" }
            }
        };

        foreach (var message in messages)
        {
            notificationManager.SendNotification(message);
        }

        // EXTENSION: Add a new notification type (e.g., WhatsApp) without modifying existing code
        notificationManager.RegisterService(new WhatsAppNotificationService());

        var whatsappMessage = new NotificationMessage
        {
            To = "+1234567890",
            Body = "Hello from WhatsApp!",
            Type = NotificationType.WhatsApp
        };

        notificationManager.SendNotification(whatsappMessage);
    }
}

// Hypothetical new notification service (can be added without changing existing code)
public class WhatsAppNotificationService : INotificationService
{
    public NotificationType SupportedType => NotificationType.WhatsApp;

    public void Send(NotificationMessage message)
    {
        Console.WriteLine($"Sending WhatsApp message to {message.To}: {message.Body}");
        // WhatsApp Business API integration
    }
}
```

## Strategy Pattern for OCP

**Question:** How does the Strategy Pattern help implement OCP?

**Answer:** The Strategy Pattern is a common way to implement OCP:

```csharp
// Strategy interface
public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal price);
}

// Concrete strategies
public class NoDiscountStrategy : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal price) => price;
}

public class PercentageDiscountStrategy : IDiscountStrategy
{
    private readonly decimal _percentage;

    public PercentageDiscountStrategy(decimal percentage)
    {
        _percentage = percentage;
    }

    public decimal ApplyDiscount(decimal price)
        => price * (1 - _percentage / 100);
}

public class FixedAmountDiscountStrategy : IDiscountStrategy
{
    private readonly decimal _amount;

    public FixedAmountDiscountStrategy(decimal amount)
    {
        _amount = amount;
    }

    public decimal ApplyDiscount(decimal price)
        => Math.Max(0, price - _amount);
}

// OCP-compliant pricing calculator
public class PricingCalculator
{
    private readonly IDiscountStrategy _discountStrategy;

    public PricingCalculator(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculatePrice(decimal basePrice, int quantity)
    {
        decimal subtotal = basePrice * quantity;
        return _discountStrategy.ApplyDiscount(subtotal);
    }

    // Can change discount strategy at runtime without modifying this class
    public void ChangeDiscountStrategy(IDiscountStrategy newStrategy)
    {
        // In a real implementation, this might update a field
        // For demo purposes, we show the concept
    }
}
```

## Template Method Pattern for OCP

**Question:** How does the Template Method Pattern help implement OCP?

**Answer:** Template Method allows subclasses to customize behavior without changing the algorithm structure:

```csharp
// Abstract template
public abstract class DataProcessor
{
    // Template method - defines the algorithm structure
    public void ProcessData(string data)
    {
        // Common steps
        ValidateData(data);
        ParseData(data);
        ProcessParsedData();
        SaveResults();

        // Hook for subclasses
        OnProcessingComplete();
    }

    protected abstract void ValidateData(string data);
    protected abstract void ParseData(string data);
    protected abstract void ProcessParsedData();

    // Common implementation
    protected virtual void SaveResults()
    {
        Console.WriteLine("Saving results to database");
    }

    // Hook method - can be overridden
    protected virtual void OnProcessingComplete()
    {
        // Default: do nothing
    }
}

// Concrete implementations
public class XmlDataProcessor : DataProcessor
{
    protected override void ValidateData(string data)
    {
        Console.WriteLine("Validating XML data");
        // XML validation logic
    }

    protected override void ParseData(string data)
    {
        Console.WriteLine("Parsing XML data");
        // XML parsing logic
    }

    protected override void ProcessParsedData()
    {
        Console.WriteLine("Processing parsed XML data");
        // XML processing logic
    }
}

public class JsonDataProcessor : DataProcessor
{
    protected override void ValidateData(string data)
    {
        Console.WriteLine("Validating JSON data");
        // JSON validation logic
    }

    protected override void ParseData(string data)
    {
        Console.WriteLine("Parsing JSON data");
        // JSON parsing logic
    }

    protected override void ProcessParsedData()
    {
        Console.WriteLine("Processing parsed JSON data");
        // JSON processing logic
    }

    protected override void OnProcessingComplete()
    {
        Console.WriteLine("JSON processing completed - sending notification");
        // Custom completion logic
    }
}
```

## Common OCP Violations

**Question:** What are common signs that OCP is being violated?

**Answer:**

1. **Switch statements**: Adding new cases requires code changes
2. **Type checking with if/else**: `if (obj is TypeA)` patterns
3. **Hard-coded type lists**: Arrays or lists of types that need updating
4. **Inheritance abuse**: Deep inheritance hierarchies that break with new requirements
5. **God classes**: Classes that handle multiple concerns and need frequent changes

## How to Implement OCP

**Question:** What's the process for implementing OCP?

**Answer:**

1. **Identify extension points**: Where will new functionality be added?
2. **Create abstractions**: Interfaces or abstract classes for variable behavior
3. **Implement concrete classes**: One for each variation
4. **Use composition**: Favor composition over inheritance
5. **Apply design patterns**: Strategy, Template Method, Factory, etc.
6. **Test extensibility**: Ensure new implementations work without changing existing code

## OCP and Other SOLID Principles

**Question:** How does OCP relate to other SOLID principles?

**Answer:**

- **SRP**: Classes with single responsibility are easier to extend
- **LSP**: Subtypes must be substitutable for extension to work
- **ISP**: Smaller interfaces make extension easier
- **DIP**: Depend on abstractions, not concretions, for extensibility

## Summary

**Question:** What's the key takeaway about OCP?

**Answer:** The Open-Closed Principle is about **extensibility without modification**. Design your code so that new features can be added through new code rather than changing existing code. This leads to more stable, maintainable, and flexible systems.

**Key Points:**
- Open for extension, closed for modification
- Use abstractions and polymorphism
- Apply design patterns like Strategy and Template Method
- Extension points should be planned in advance
- Testing existing code doesn't need to be repeated