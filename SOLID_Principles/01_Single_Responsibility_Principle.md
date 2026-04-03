# SOLID Principles - Single Responsibility Principle (SRP)

## What is the Single Responsibility Principle (SRP)?

**Question:** What does the Single Responsibility Principle state?

**Answer:** The Single Responsibility Principle states that **a class should have only one reason to change**. This means a class should have only one job or responsibility. In other words, a class should be responsible for only one part of the software's functionality.

## Why is SRP Important?

**Question:** Why should we follow the Single Responsibility Principle?

**Answer:** SRP is important because:
- **Maintainability**: Changes to one responsibility don't affect others
- **Testability**: Easier to test single responsibilities
- **Readability**: Code is clearer and more focused
- **Flexibility**: Easier to modify and extend
- **Reduced Coupling**: Less dependencies between different concerns

## Real-World Analogy

**Question:** Can you give a real-world analogy for SRP?

**Answer:** Think of a restaurant kitchen. A chef shouldn't also be the waiter, cashier, and dishwasher. Each person has one clear responsibility:
- Chef: Cooks food
- Waiter: Takes orders and serves food
- Cashier: Handles payments
- Dishwasher: Cleans dishes

If one person tries to do everything, mistakes in one area affect all others.

## Violation Example

**Question:** What does a class violating SRP look like?

**Answer:** Here's a class that violates SRP:

```csharp
// VIOLATION: Multiple responsibilities in one class
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }

    // Responsibility 1: Employee data management
    public void SaveToDatabase()
    {
        // Database operations
        Console.WriteLine($"Saving {Name} to database");
    }

    // Responsibility 2: Email sending
    public void SendEmail(string message)
    {
        // Email operations
        Console.WriteLine($"Sending email to {Name}: {message}");
    }

    // Responsibility 3: Salary calculation
    public decimal CalculateTax()
    {
        // Tax calculation logic
        return Salary * 0.2m;
    }

    // Responsibility 4: Report generation
    public string GenerateReport()
    {
        // Report formatting
        return $"Employee Report: {Name}, Salary: {Salary}";
    }
}
```

**Why this violates SRP:**
- Database operations
- Email sending
- Tax calculation
- Report generation

**Problems:**
- If email logic changes, we modify Employee class
- If database schema changes, we modify Employee class
- Hard to test individual functionalities
- Class becomes bloated and hard to maintain

## Correct Implementation

**Question:** How should we refactor the above code to follow SRP?

**Answer:** We need to separate each responsibility into its own class:

```csharp
// Responsibility 1: Employee data (Data Transfer Object)
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
}

// Responsibility 2: Employee data persistence
public class EmployeeRepository
{
    public void Save(Employee employee)
    {
        // Database operations only
        Console.WriteLine($"Saving {employee.Name} to database");
    }

    public Employee GetById(int id)
    {
        // Database retrieval only
        return new Employee { Name = "John Doe", Salary = 50000 };
    }
}

// Responsibility 3: Email notifications
public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        // Email operations only
        Console.WriteLine($"Sending email to {to}: {subject} - {body}");
    }
}

// Responsibility 4: Tax calculations
public class TaxCalculator
{
    public decimal CalculateTax(decimal salary)
    {
        // Tax calculation logic only
        return salary * 0.2m;
    }
}

// Responsibility 5: Report generation
public class ReportGenerator
{
    public string GenerateEmployeeReport(Employee employee)
    {
        // Report formatting only
        return $"Employee Report: {employee.Name}, Salary: {employee.Salary}";
    }
}
```

## Real-Time Application Example

**Question:** Can you show a real-time application of SRP in an e-commerce system?

**Answer:** Let's build an e-commerce order processing system following SRP:

```csharp
// 1. Order data (DTO)
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

// 2. Order validation (Single responsibility: validation)
public class OrderValidator
{
    public ValidationResult Validate(Order order)
    {
        var result = new ValidationResult();

        if (string.IsNullOrEmpty(order.CustomerEmail))
            result.Errors.Add("Customer email is required");

        if (order.Items == null || !order.Items.Any())
            result.Errors.Add("Order must have at least one item");

        if (order.TotalAmount <= 0)
            result.Errors.Add("Total amount must be greater than zero");

        foreach (var item in order.Items)
        {
            if (item.Quantity <= 0)
                result.Errors.Add($"Invalid quantity for {item.ProductName}");
        }

        return result;
    }
}

public class ValidationResult
{
    public List<string> Errors { get; set; } = new();
    public bool IsValid => !Errors.Any();
}

// 3. Order processing (Single responsibility: business logic)
public class OrderProcessor
{
    private readonly OrderValidator _validator;

    public OrderProcessor(OrderValidator validator)
    {
        _validator = validator;
    }

    public ProcessResult ProcessOrder(Order order)
    {
        // Validate order
        var validation = _validator.Validate(order);
        if (!validation.IsValid)
            return new ProcessResult { Success = false, Errors = validation.Errors };

        // Calculate total
        order.TotalAmount = order.Items.Sum(item => item.Quantity * item.UnitPrice);

        // Set status
        order.Status = OrderStatus.Confirmed;

        return new ProcessResult { Success = true, Order = order };
    }
}

public class ProcessResult
{
    public bool Success { get; set; }
    public Order Order { get; set; }
    public List<string> Errors { get; set; } = new();
}

// 4. Order persistence (Single responsibility: data storage)
public class OrderRepository
{
    private readonly List<Order> _orders = new();
    private int _nextId = 1;

    public Order Save(Order order)
    {
        if (order.Id == 0)
        {
            order.Id = _nextId++;
        }
        _orders.Add(order);
        Console.WriteLine($"Order {order.Id} saved to database");
        return order;
    }

    public Order GetById(int id)
    {
        return _orders.FirstOrDefault(o => o.Id == id);
    }
}

// 5. Email notifications (Single responsibility: communication)
public class EmailService
{
    public void SendOrderConfirmation(Order order)
    {
        string subject = $"Order Confirmation #{order.Id}";
        string body = $"Your order totaling ${order.TotalAmount} has been confirmed.";
        Console.WriteLine($"Email sent to {order.CustomerEmail}: {subject}");
    }

    public void SendShippingNotification(Order order)
    {
        string subject = $"Order Shipped #{order.Id}";
        string body = $"Your order has been shipped.";
        Console.WriteLine($"Email sent to {order.CustomerEmail}: {subject}");
    }
}

// 6. Payment processing (Single responsibility: payments)
public class PaymentService
{
    public PaymentResult ProcessPayment(Order order, string paymentMethod)
    {
        // Payment processing logic
        Console.WriteLine($"Processing payment of ${order.TotalAmount} via {paymentMethod}");

        // Simulate payment processing
        bool success = order.TotalAmount < 1000; // Simple validation

        return new PaymentResult
        {
            Success = success,
            TransactionId = success ? Guid.NewGuid().ToString() : null
        };
    }
}

public class PaymentResult
{
    public bool Success { get; set; }
    public string TransactionId { get; set; }
    public string ErrorMessage { get; set; }
}

// 7. Order service (Orchestrates all responsibilities)
public class OrderService
{
    private readonly OrderValidator _validator;
    private readonly OrderProcessor _processor;
    private readonly OrderRepository _repository;
    private readonly EmailService _emailService;
    private readonly PaymentService _paymentService;

    public OrderService()
    {
        _validator = new OrderValidator();
        _processor = new OrderProcessor(_validator);
        _repository = new OrderRepository();
        _emailService = new EmailService();
        _paymentService = new PaymentService();
    }

    public OrderResult PlaceOrder(Order order, string paymentMethod)
    {
        try
        {
            // Process the order
            var processResult = _processor.ProcessOrder(order);
            if (!processResult.Success)
            {
                return new OrderResult
                {
                    Success = false,
                    Errors = processResult.Errors
                };
            }

            // Process payment
            var paymentResult = _paymentService.ProcessPayment(order, paymentMethod);
            if (!paymentResult.Success)
            {
                return new OrderResult
                {
                    Success = false,
                    Errors = new List<string> { "Payment failed" }
                };
            }

            // Save order
            var savedOrder = _repository.Save(order);

            // Send confirmation email
            _emailService.SendOrderConfirmation(savedOrder);

            return new OrderResult
            {
                Success = true,
                Order = savedOrder,
                TransactionId = paymentResult.TransactionId
            };
        }
        catch (Exception ex)
        {
            return new OrderResult
            {
                Success = false,
                Errors = new List<string> { $"System error: {ex.Message}" }
            };
        }
    }
}

public class OrderResult
{
    public bool Success { get; set; }
    public Order Order { get; set; }
    public string TransactionId { get; set; }
    public List<string> Errors { get; set; } = new();
}

// Usage Example
public class Program
{
    public static void Main()
    {
        var orderService = new OrderService();

        var order = new Order
        {
            CustomerEmail = "customer@example.com",
            Items = new List<OrderItem>
            {
                new OrderItem { ProductId = 1, ProductName = "Laptop", Quantity = 1, UnitPrice = 999.99m },
                new OrderItem { ProductId = 2, ProductName = "Mouse", Quantity = 2, UnitPrice = 25.00m }
            }
        };

        var result = orderService.PlaceOrder(order, "CreditCard");

        if (result.Success)
        {
            Console.WriteLine($"Order placed successfully! Order ID: {result.Order.Id}, Transaction: {result.TransactionId}");
        }
        else
        {
            Console.WriteLine($"Order failed: {string.Join(", ", result.Errors)}");
        }
    }
}
```

## Benefits in the E-commerce Example

**Question:** What are the benefits of applying SRP in the e-commerce example?

**Answer:**

1. **Easy Testing**: Each class can be tested independently
2. **Easy Modification**: Change email logic without affecting payment logic
3. **Code Reusability**: EmailService can be used for other notifications
4. **Maintainability**: Bug in validation doesn't affect persistence
5. **Parallel Development**: Different developers can work on different responsibilities
6. **Clear Dependencies**: Each class clearly states what it depends on

## Common SRP Violations

**Question:** What are common signs that SRP is being violated?

**Answer:**

1. **Class names with "and"**: `CustomerDataAndEmailService`
2. **Methods that do too many things**: A method that validates, saves, and sends email
3. **Large classes**: Classes with hundreds of lines doing multiple things
4. **Frequent changes**: Class changes for unrelated reasons
5. **Hard to test**: Difficult to test one aspect without others
6. **Multiple reasons to change**: Database changes, UI changes, business logic changes all affect same class

## How to Identify Responsibilities

**Question:** How do you identify what constitutes a single responsibility?

**Answer:**

1. **Business rules**: What the business considers separate concerns
2. **Change reasons**: What would cause this code to change
3. **Dependencies**: What other systems this code interacts with
4. **Cohesion**: How closely related the operations are
5. **Abstraction level**: Operations at the same level of abstraction

## Refactoring to SRP

**Question:** What's the process for refactoring a class to follow SRP?

**Answer:**

1. **Identify responsibilities**: List all things the class does
2. **Group related operations**: Find operations that belong together
3. **Create separate classes**: One class per responsibility
4. **Define interfaces**: Create abstractions for dependencies
5. **Update dependencies**: Use dependency injection
6. **Test each class**: Ensure each responsibility works independently

## SRP and Other SOLID Principles

**Question:** How does SRP relate to other SOLID principles?

**Answer:**

- **OCP**: Easier to extend when responsibilities are separated
- **LSP**: Subclasses can focus on single responsibilities
- **ISP**: Interfaces become smaller and focused
- **DIP**: Dependencies become more specific and testable

## Summary

**Question:** What's the key takeaway about SRP?

**Answer:** The Single Responsibility Principle is about **focus and separation**. Each class should have one clear job and do it well. This leads to more maintainable, testable, and flexible code. When a class has multiple responsibilities, it becomes a maintenance nightmare where changes in one area break others.

**Key Points:**
- One class = One responsibility
- Changes should have one reason
- Separate concerns lead to better design
- Easier testing and maintenance
- Foundation for other SOLID principles