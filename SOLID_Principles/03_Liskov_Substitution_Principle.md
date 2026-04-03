# SOLID Principles - Liskov Substitution Principle (LSP)

## What is the Liskov Substitution Principle (LSP)?

**Question:** What does the Liskov Substitution Principle state?

**Answer:** The Liskov Substitution Principle states that **objects of a superclass should be replaceable with objects of its subclasses without affecting the correctness of the program**. In other words, if S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program.

## Why is LSP Important?

**Question:** Why should we follow the Liskov Substitution Principle?

**Answer:** LSP is important because:
- **Correctness**: Subtypes behave as expected by clients
- **Polymorphism**: Enables proper object-oriented design
- **Maintainability**: Changes to subtypes don't break existing code
- **Testability**: Subtypes can be tested in place of base types
- **Extensibility**: New subtypes can be added safely

## Real-World Analogy

**Question:** Can you give a real-world analogy for LSP?

**Answer:** Think of electrical outlets. A three-prong plug (subtype) should work in any outlet that accepts two-prong plugs (supertype), but not vice versa. The subtype extends the interface but maintains compatibility.

## Violation Example

**Question:** What does code violating LSP look like?

**Answer:** Here's a classic LSP violation:

```csharp
// VIOLATION: Square inheriting from Rectangle breaks LSP
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    private int _side;

    public override int Width
    {
        get => _side;
        set
        {
            _side = value;
            // PROBLEM: Setting width also changes height
            base.Height = value;
        }
    }

    public override int Height
    {
        get => _side;
        set
        {
            _side = value;
            // PROBLEM: Setting height also changes width
            base.Width = value;
        }
    }
}

// Client code that expects LSP
public class AreaCalculator
{
    public int CalculateTotalArea(List<Rectangle> rectangles)
    {
        int total = 0;
        foreach (var rect in rectangles)
        {
            // This works for Rectangle but fails for Square
            rect.Width = 4;
            rect.Height = 5;
            total += rect.CalculateArea(); // Expected: 20, but Square gives 25!
        }
        return total;
    }
}
```

**Why this violates LSP:**
- Square changes the behavior of Rectangle
- Setting Width also changes Height (and vice versa)
- Client code breaks when using Square instead of Rectangle
- The subtype doesn't fulfill the contract of the supertype

## Correct Implementation

**Question:** How should we fix the Rectangle/Square inheritance to follow LSP?

**Answer:** We need to redesign the inheritance hierarchy:

```csharp
// Solution 1: Use composition instead of inheritance
public interface IShape
{
    int CalculateArea();
    int CalculatePerimeter();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public int CalculateArea()
    {
        return Width * Height;
    }

    public int CalculatePerimeter()
    {
        return 2 * (Width + Height);
    }
}

public class Square : IShape
{
    public int Side { get; set; }

    public int CalculateArea()
    {
        return Side * Side;
    }

    public int CalculatePerimeter()
    {
        return 4 * Side;
    }
}

// Solution 2: If inheritance is needed, make Square NOT inherit from Rectangle
public abstract class Shape
{
    public abstract int CalculateArea();
    public abstract int CalculatePerimeter();
}

public class RectangleShape : Shape
{
    public int Width { get; set; }
    public int Height { get; set; }

    public override int CalculateArea()
    {
        return Width * Height;
    }

    public override int CalculatePerimeter()
    {
        return 2 * (Width + Height);
    }
}

public class SquareShape : Shape
{
    public int Side { get; set; }

    public override int CalculateArea()
    {
        return Side * Side;
    }

    public override int CalculatePerimeter()
    {
        return 4 * Side;
    }
}
```

## Real-Time Application Example

**Question:** Can you show a real-time application of LSP in a payment processing system?

**Answer:** Let's build a payment system that properly follows LSP:

```csharp
// Base payment method contract
public abstract class PaymentMethod
{
    public string Name { get; protected set; }
    public abstract bool IsValid();
    public abstract PaymentResult ProcessPayment(decimal amount);
    public abstract void CancelPayment(string transactionId);
}

// Payment result
public class PaymentResult
{
    public bool Success { get; set; }
    public string TransactionId { get; set; }
    public string ErrorMessage { get; set; }
    public decimal Amount { get; set; }
}

// Credit card payment - follows LSP
public class CreditCardPayment : PaymentMethod
{
    public string CardNumber { get; set; }
    public string ExpiryDate { get; set; }
    public string CVV { get; set; }

    public CreditCardPayment()
    {
        Name = "Credit Card";
    }

    public override bool IsValid()
    {
        // Validate card number, expiry, CVV
        return !string.IsNullOrEmpty(CardNumber) &&
               CardNumber.Length == 16 &&
               !string.IsNullOrEmpty(ExpiryDate) &&
               !string.IsNullOrEmpty(CVV);
    }

    public override PaymentResult ProcessPayment(decimal amount)
    {
        if (!IsValid())
        {
            return new PaymentResult
            {
                Success = false,
                ErrorMessage = "Invalid credit card details"
            };
        }

        // Process credit card payment
        string transactionId = Guid.NewGuid().ToString();
        Console.WriteLine($"Processing credit card payment: ${amount}, Transaction: {transactionId}");

        return new PaymentResult
        {
            Success = true,
            TransactionId = transactionId,
            Amount = amount
        };
    }

    public override void CancelPayment(string transactionId)
    {
        Console.WriteLine($"Cancelling credit card payment: {transactionId}");
        // Credit card refund logic
    }
}

// PayPal payment - follows LSP
public class PayPalPayment : PaymentMethod
{
    public string Email { get; set; }
    public string Password { get; set; }

    public PayPalPayment()
    {
        Name = "PayPal";
    }

    public override bool IsValid()
    {
        return !string.IsNullOrEmpty(Email) &&
               Email.Contains("@") &&
               !string.IsNullOrEmpty(Password);
    }

    public override PaymentResult ProcessPayment(decimal amount)
    {
        if (!IsValid())
        {
            return new PaymentResult
            {
                Success = false,
                ErrorMessage = "Invalid PayPal credentials"
            };
        }

        // Process PayPal payment
        string transactionId = Guid.NewGuid().ToString();
        Console.WriteLine($"Processing PayPal payment: ${amount}, Transaction: {transactionId}");

        return new PaymentResult
        {
            Success = true,
            TransactionId = transactionId,
            Amount = amount
        };
    }

    public override void CancelPayment(string transactionId)
    {
        Console.WriteLine($"Cancelling PayPal payment: {transactionId}");
        // PayPal refund logic
    }
}

// Bank transfer payment - follows LSP
public class BankTransferPayment : PaymentMethod
{
    public string AccountNumber { get; set; }
    public string RoutingNumber { get; set; }

    public BankTransferPayment()
    {
        Name = "Bank Transfer";
    }

    public override bool IsValid()
    {
        return !string.IsNullOrEmpty(AccountNumber) &&
               AccountNumber.Length >= 8 &&
               !string.IsNullOrEmpty(RoutingNumber) &&
               RoutingNumber.Length == 9;
    }

    public override PaymentResult ProcessPayment(decimal amount)
    {
        if (!IsValid())
        {
            return new PaymentResult
            {
                Success = false,
                ErrorMessage = "Invalid bank account details"
            };
        }

        // Process bank transfer (this might be asynchronous)
        string transactionId = Guid.NewGuid().ToString();
        Console.WriteLine($"Processing bank transfer: ${amount}, Transaction: {transactionId}");

        return new PaymentResult
        {
            Success = true,
            TransactionId = transactionId,
            Amount = amount
        };
    }

    public override void CancelPayment(string transactionId)
    {
        Console.WriteLine($"Cancelling bank transfer: {transactionId}");
        // Bank transfer cancellation logic
    }
}

// LSP-compliant payment processor
public class PaymentProcessor
{
    public PaymentResult ProcessPayment(PaymentMethod paymentMethod, decimal amount)
    {
        // Client code can use any PaymentMethod subtype interchangeably
        if (!paymentMethod.IsValid())
        {
            return new PaymentResult
            {
                Success = false,
                ErrorMessage = $"{paymentMethod.Name} validation failed"
            };
        }

        return paymentMethod.ProcessPayment(amount);
    }

    public void CancelPayment(PaymentMethod paymentMethod, string transactionId)
    {
        // All payment methods support cancellation
        paymentMethod.CancelPayment(transactionId);
    }
}

// Usage example
public class PaymentExample
{
    public static void Main()
    {
        var processor = new PaymentProcessor();
        decimal amount = 99.99m;

        // Create different payment methods
        var payments = new List<PaymentMethod>
        {
            new CreditCardPayment
            {
                CardNumber = "1234567890123456",
                ExpiryDate = "12/25",
                CVV = "123"
            },
            new PayPalPayment
            {
                Email = "user@example.com",
                Password = "password123"
            },
            new BankTransferPayment
            {
                AccountNumber = "123456789",
                RoutingNumber = "021000021"
            }
        };

        // Process payments - LSP allows us to treat all as PaymentMethod
        foreach (var payment in payments)
        {
            var result = processor.ProcessPayment(payment, amount);
            if (result.Success)
            {
                Console.WriteLine($"{payment.Name} payment successful: {result.TransactionId}");

                // Demonstrate cancellation works for all types
                processor.CancelPayment(payment, result.TransactionId);
            }
            else
            {
                Console.WriteLine($"{payment.Name} payment failed: {result.ErrorMessage}");
            }
        }
    }
}
```

## Behavioral Subtyping Rules

**Question:** What are the behavioral subtyping rules for LSP?

**Answer:** For proper LSP compliance, subtypes must follow these rules:

### 1. **Contravariance of Method Arguments**
```csharp
public class Animal
{
    public virtual void Eat(Food food) { /* ... */ }
}

public class Dog : Animal
{
    // CORRECT: Dog can eat more specific food types
    public override void Eat(Meat food) { /* ... */ }
}
```

### 2. **Covariance of Return Types**
```csharp
public class Animal
{
    public virtual Food GetFavoriteFood() { return new Food(); }
}

public class Dog : Animal
{
    // CORRECT: Dog returns more specific food type
    public override Meat GetFavoriteFood() { return new Meat(); }
}
```

### 3. **No Strengthening Preconditions**
```csharp
public class Animal
{
    public virtual void Move(int distance)
    {
        // Can move any positive distance
        if (distance <= 0) throw new ArgumentException();
    }
}

public class Dog : Animal
{
    // VIOLATION: Dog requires distance > 10, strengthening precondition
    public override void Move(int distance)
    {
        if (distance <= 10) throw new ArgumentException(); // LSP violation!
    }
}
```

### 4. **No Weakening Postconditions**
```csharp
public class Animal
{
    public virtual bool TryMove(int distance)
    {
        // Guarantees: returns true if movement successful
        return distance > 0;
    }
}

public class Dog : Animal
{
    // VIOLATION: Dog might return false even for valid distances
    public override bool TryMove(int distance)
    {
        // Sometimes fails randomly - weakens postcondition
        return distance > 0 && new Random().Next(2) == 0; // LSP violation!
    }
}
```

### 5. **Exception Rules**
- Subtypes can throw fewer or more specific exceptions
- Cannot throw new exception types not thrown by supertype

## Common LSP Violations

**Question:** What are common signs that LSP is being violated?

**Answer:**

1. **Type checking in client code**: `if (obj is SpecificType)` 
2. **Downcasting**: `(SpecificType)obj` operations
3. **Empty method overrides**: Methods that do nothing or throw NotImplementedException
4. **Overriding with different behavior**: Subtype behaves differently than expected
5. **Strengthening preconditions**: Subtype requires more than supertype
6. **Weakening postconditions**: Subtype guarantees less than supertype

## LSP and Design Patterns

**Question:** How do design patterns help implement LSP?

**Answer:**

### Strategy Pattern
```csharp
public interface ISortingStrategy
{
    void Sort(int[] array);
}

public class QuickSort : ISortingStrategy
{
    public void Sort(int[] array) { /* QuickSort implementation */ }
}

public class MergeSort : ISortingStrategy
{
    public void Sort(int[] array) { /* MergeSort implementation */ }
}

// Client can use any ISortingStrategy interchangeably
```

### Factory Pattern with LSP
```csharp
public abstract class Document
{
    public abstract void Open();
    public abstract void Save();
}

public class PdfDocument : Document
{
    public override void Open() { /* PDF open logic */ }
    public override void Save() { /* PDF save logic */ }
}

public class WordDocument : Document
{
    public override void Open() { /* Word open logic */ }
    public override void Save() { /* Word save logic */ }
}

// Factory returns substitutable objects
public class DocumentFactory
{
    public static Document CreateDocument(string type)
    {
        return type switch
        {
            "pdf" => new PdfDocument(),
            "word" => new WordDocument(),
            _ => throw new ArgumentException()
        };
    }
}
```

## Testing for LSP Compliance

**Question:** How do you test that your code follows LSP?

**Answer:** Use the **LSP Testing Pattern**:

```csharp
[TestFixture]
public class LspTests
{
    [Test]
    public void AllShapes_Should_Calculate_Area_Correctly()
    {
        // Arrange
        var shapes = new List<IShape>
        {
            new Rectangle { Width = 4, Height = 5 },
            new Square { Side = 4 }
        };

        // Act & Assert
        foreach (var shape in shapes)
        {
            // All shapes should behave consistently
            var area = shape.CalculateArea();
            Assert.Greater(area, 0);
        }
    }

    [Test]
    public void PaymentMethods_Should_Process_Payments_Consistently()
    {
        // Arrange
        var paymentMethods = new List<PaymentMethod>
        {
            new CreditCardPayment { /* valid data */ },
            new PayPalPayment { /* valid data */ }
        };

        // Act & Assert
        foreach (var method in paymentMethods)
        {
            var result = method.ProcessPayment(100.00m);
            Assert.IsTrue(result.Success);
            Assert.IsNotNull(result.TransactionId);
        }
    }
}
```

## LSP and Other SOLID Principles

**Question:** How does LSP relate to other SOLID principles?

**Answer:**

- **SRP**: Subtypes should have single responsibility like their parents
- **OCP**: LSP enables OCP by ensuring subtypes can extend without breaking
- **ISP**: Proper LSP requires focused interfaces
- **DIP**: LSP supports DIP by enabling dependency on abstractions

## Summary

**Question:** What's the key takeaway about LSP?

**Answer:** The Liskov Substitution Principle ensures that **inheritance hierarchies are correct**. Subtypes must be completely substitutable for their base types. When LSP is violated, polymorphism breaks down and client code becomes fragile.

**Key Points:**
- Subtypes must honor the contract of their supertypes
- Behavioral subtyping rules must be followed
- Inheritance should model "is-a" relationships correctly
- Test subtypes by substituting them in client code
- LSP is the foundation of proper polymorphism