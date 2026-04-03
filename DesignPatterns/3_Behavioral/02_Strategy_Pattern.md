# Strategy Pattern

## Definition
The Strategy Pattern defines a family of **interchangeable algorithms**, encapsulates each one, and makes them **interchangeable at runtime**. It lets the algorithm vary independently from clients that use it.

## Core Concept
- **Encapsulation**: Each algorithm in its own class
- **Interchangeability**: Swap algorithms at runtime
- **Context Independence**: Client doesn't know about specific algorithms
- **Family of Algorithms**: Related algorithms grouped together
- **Runtime Selection**: Choose algorithm based on runtime conditions

## Problem It Solves
- Multiple similar algorithms with different implementations
- Conditional logic with many if-else branches
- Algorithm selection at runtime
- Avoiding tight coupling to specific implementations
- Need to extend algorithms without modifying existing code

## Generic Implementation

### Basic Strategy Pattern

```csharp
// Strategy Interface
public interface IPaymentStrategy
{
    bool Pay(decimal amount);
    string GetPaymentMethod();
}

// Concrete Strategies
public class CreditCardPaymentStrategy : IPaymentStrategy
{
    private string _cardNumber;

    public CreditCardPaymentStrategy(string cardNumber)
    {
        _cardNumber = cardNumber;
    }

    public bool Pay(decimal amount)
    {
        Console.WriteLine($"Processing credit card payment of ${amount}");
        Console.WriteLine($"Card: {_cardNumber}");
        return true;
    }

    public string GetPaymentMethod() => "Credit Card";
}

public class PayPalPaymentStrategy : IPaymentStrategy
{
    private string _email;

    public PayPalPaymentStrategy(string email)
    {
        _email = email;
    }

    public bool Pay(decimal amount)
    {
        Console.WriteLine($"Processing PayPal payment of ${amount}");
        Console.WriteLine($"PayPal Account: {_email}");
        return true;
    }

    public string GetPaymentMethod() => "PayPal";
}

public class CryptoCurrencyPaymentStrategy : IPaymentStrategy
{
    private string _walletAddress;

    public CryptoCurrencyPaymentStrategy(string walletAddress)
    {
        _walletAddress = walletAddress;
    }

    public bool Pay(decimal amount)
    {
        Console.WriteLine($"Processing cryptocurrency payment of ${amount}");
        Console.WriteLine($"Wallet: {_walletAddress}");
        return true;
    }

    public string GetPaymentMethod() => "Cryptocurrency";
}

// Context
public class ShoppingCart
{
    private decimal _total;
    private IPaymentStrategy _paymentStrategy;

    public ShoppingCart(decimal total)
    {
        _total = total;
    }

    public void SetPaymentStrategy(IPaymentStrategy strategy)
    {
        _paymentStrategy = strategy;
    }

    public void Checkout()
    {
        if (_paymentStrategy == null)
            throw new InvalidOperationException("Payment strategy not set");

        Console.WriteLine($"Total Amount: ${_total}");
        Console.WriteLine($"Payment Method: {_paymentStrategy.GetPaymentMethod()}");
        
        if (_paymentStrategy.Pay(_total))
            Console.WriteLine("Payment successful!");
        else
            Console.WriteLine("Payment failed!");
    }
}

// Usage
var cart = new ShoppingCart(99.99m);

// Strategy 1: Credit Card
cart.SetPaymentStrategy(new CreditCardPaymentStrategy("1234-5678-9012-3456"));
cart.Checkout();

Console.WriteLine();

// Strategy 2: PayPal
cart.SetPaymentStrategy(new PayPalPaymentStrategy("user@example.com"));
cart.Checkout();

Console.WriteLine();

// Strategy 3: Cryptocurrency
cart.SetPaymentStrategy(new CryptoCurrencyPaymentStrategy("1A1z7agoat4..."));
cart.Checkout();
```

### Sorting Strategy

```csharp
// Strategy Interface
public interface ISortingStrategy
{
    void Sort(int[] array);
}

// Concrete Strategies
public class BubbleSortStrategy : ISortingStrategy
{
    public void Sort(int[] array)
    {
        Console.WriteLine("Sorting with Bubble Sort");
        for (int i = 0; i < array.Length; i++)
        {
            for (int j = 0; j < array.Length - 1 - i; j++)
            {
                if (array[j] > array[j + 1])
                {
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
}

public class QuickSortStrategy : ISortingStrategy
{
    public void Sort(int[] array)
    {
        Console.WriteLine("Sorting with Quick Sort");
        QuickSort(array, 0, array.Length - 1);
    }

    private void QuickSort(int[] array, int low, int high)
    {
        if (low < high)
        {
            int pivot = Partition(array, low, high);
            QuickSort(array, low, pivot - 1);
            QuickSort(array, pivot + 1, high);
        }
    }

    private int Partition(int[] array, int low, int high)
    {
        int pivot = array[high];
        int i = low - 1;
        for (int j = low; j < high; j++)
        {
            if (array[j] < pivot)
            {
                i++;
                int temp = array[i];
                array[i] = array[j];
                array[j] = temp;
            }
        }
        int temp2 = array[i + 1];
        array[i + 1] = array[high];
        array[high] = temp2;
        return i + 1;
    }
}

public class MergeSortStrategy : ISortingStrategy
{
    public void Sort(int[] array)
    {
        Console.WriteLine("Sorting with Merge Sort");
        MergeSort(array, 0, array.Length - 1);
    }

    private void MergeSort(int[] array, int left, int right)
    {
        if (left < right)
        {
            int mid = left + (right - left) / 2;
            MergeSort(array, left, mid);
            MergeSort(array, mid + 1, right);
            Merge(array, left, mid, right);
        }
    }

    private void Merge(int[] array, int left, int mid, int right)
    {
        // Merge implementation
    }
}

// Context
public class DataSorter
{
    private ISortingStrategy _strategy;

    public void SetStrategy(ISortingStrategy strategy)
    {
        _strategy = strategy;
    }

    public void SortData(int[] data)
    {
        if (_strategy == null)
            throw new InvalidOperationException("Strategy not set");

        _strategy.Sort(data);
        Console.WriteLine($"Sorted: {string.Join(", ", data)}");
    }
}

// Usage
var sorter = new DataSorter();
int[] data = { 5, 2, 8, 1, 9 };

sorter.SetStrategy(new BubbleSortStrategy());
sorter.SortData(data);

data = new int[] { 5, 2, 8, 1, 9 };
sorter.SetStrategy(new QuickSortStrategy());
sorter.SortData(data);
```

## Real-World Examples

### Compression Strategy

```csharp
public interface ICompressionStrategy
{
    void Compress(string filePath);
    string GetFormat();
}

public class ZipCompressionStrategy : ICompressionStrategy
{
    public void Compress(string filePath)
    {
        Console.WriteLine($"Compressing {filePath} using ZIP format");
    }

    public string GetFormat() => "ZIP";
}

public class RarCompressionStrategy : ICompressionStrategy
{
    public void Compress(string filePath)
    {
        Console.WriteLine($"Compressing {filePath} using RAR format");
    }

    public string GetFormat() => "RAR";
}

public class GzipCompressionStrategy : ICompressionStrategy
{
    public void Compress(string filePath)
    {
        Console.WriteLine($"Compressing {filePath} using GZIP format");
    }

    public string GetFormat() => "GZIP";
}

public class FileCompressor
{
    private ICompressionStrategy _strategy;

    public void SetStrategy(ICompressionStrategy strategy)
    {
        _strategy = strategy;
    }

    public void CompressFile(string filePath)
    {
        _strategy?.Compress(filePath);
    }
}

// Usage
var compressor = new FileCompressor();
compressor.SetStrategy(new ZipCompressionStrategy());
compressor.CompressFile("document.pdf");

compressor.SetStrategy(new RarCompressionStrategy());
compressor.CompressFile("video.mp4");
```

### Export Strategy

```csharp
public interface IExportStrategy
{
    void Export(List<string> data, string fileName);
}

public class PdfExportStrategy : IExportStrategy
{
    public void Export(List<string> data, string fileName)
    {
        Console.WriteLine($"Exporting to PDF: {fileName}");
        foreach (var item in data)
            Console.WriteLine($"  - {item}");
    }
}

public class CsvExportStrategy : IExportStrategy
{
    public void Export(List<string> data, string fileName)
    {
        Console.WriteLine($"Exporting to CSV: {fileName}");
        Console.WriteLine(string.Join(",", data));
    }
}

public class ExcelExportStrategy : IExportStrategy
{
    public void Export(List<string> data, string fileName)
    {
        Console.WriteLine($"Exporting to Excel: {fileName}");
        foreach (var item in data)
            Console.WriteLine($"  [{item}]");
    }
}

public class DataExporter
{
    private IExportStrategy _strategy;

    public void SetStrategy(IExportStrategy strategy)
    {
        _strategy = strategy;
    }

    public void ExportData(List<string> data, string fileName)
    {
        _strategy?.Export(data, fileName);
    }
}

// Usage
var exporter = new DataExporter();
var data = new List<string> { "John", "Jane", "Bob" };

exporter.SetStrategy(new PdfExportStrategy());
exporter.ExportData(data, "users.pdf");

exporter.SetStrategy(new CsvExportStrategy());
exporter.ExportData(data, "users.csv");
```

### Authentication Strategy

```csharp
public interface IAuthenticationStrategy
{
    bool Authenticate(string username, string credential);
}

public class PasswordAuthenticationStrategy : IAuthenticationStrategy
{
    public bool Authenticate(string username, string password)
    {
        Console.WriteLine($"Authenticating {username} with password");
        return password == "secret123";
    }
}

public class BiometricAuthenticationStrategy : IAuthenticationStrategy
{
    public bool Authenticate(string username, string biometricData)
    {
        Console.WriteLine($"Authenticating {username} with biometric data");
        return true; // Simplified
    }
}

public class TokenAuthenticationStrategy : IAuthenticationStrategy
{
    public bool Authenticate(string username, string token)
    {
        Console.WriteLine($"Authenticating {username} with token");
        return token.StartsWith("TOKEN_");
    }
}

public class AuthenticationManager
{
    private IAuthenticationStrategy _strategy;

    public void SetStrategy(IAuthenticationStrategy strategy)
    {
        _strategy = strategy;
    }

    public bool Login(string username, string credential)
    {
        return _strategy?.Authenticate(username, credential) ?? false;
    }
}

// Usage
var auth = new AuthenticationManager();

auth.SetStrategy(new PasswordAuthenticationStrategy());
Console.WriteLine(auth.Login("john", "secret123")); // true

auth.SetStrategy(new TokenAuthenticationStrategy());
Console.WriteLine(auth.Login("jane", "TOKEN_xyz123")); // true
```

## Strategy vs Other Patterns

| Pattern | Purpose | Selection |
|---------|---------|-----------|
| **Strategy** | Encapsulate algorithms | Runtime |
| **State** | Object behavior based on state | State change |
| **Template Method** | Algorithm outline in base class | Design time |
| **Decorator** | Add features dynamically | Runtime |

## Advantages

✅ **Runtime Selection**: Choose algorithm at runtime  
✅ **Encapsulation**: Each algorithm independent  
✅ **Easy to Add**: New algorithms don't modify existing code  
✅ **Testable**: Each strategy easily testable in isolation  
✅ **Flexible**: Swap implementations seamlessly  

## Disadvantages

❌ **Extra Classes**: One class per strategy  
❌ **Overhead**: Multiple small objects  
❌ **Complexity**: More classes to understand  
❌ **Overkill**: For simple algorithms  

## When to Use

- Multiple algorithms for same task
- Algorithm selected at runtime
- Frequent algorithm changes/additions
- Want to avoid conditional logic
- Algorithm should be independent of client

## When NOT to Use

- Single algorithm (no variation)
- Algorithm never changes
- Algorithm selection is static
- Performance critical with many strategies

## Best Practices

1. **Keep Strategies Stateless**: Reuse instances across clients
2. **Document Each Strategy**: Explain implementation details
3. **Use Default Strategy**: Provide sensible default
4. **Strategy Factories**: Consider factory for creating strategies
5. **Consistent Interfaces**: All strategies have same signature

## Summary

The Strategy Pattern is ideal when you have multiple algorithms and need to choose between them at runtime. It eliminates conditional logic and makes code more maintainable by encapsulating each algorithm in its own class. Perfect for flexible, extensible systems.
