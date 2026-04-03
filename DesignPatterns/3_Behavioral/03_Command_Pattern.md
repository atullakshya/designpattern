# Command Pattern

## Definition
The Command Pattern encapsulates a **request as an object**, allowing you to **parameterize** clients with different requests, queue requests, log requests, and support undoable operations.

## Core Concept
- **Encapsulation**: Request wrapped in object
- **Parameterization**: Pass commands like data
- **Decoupling**: Sender doesn't know receiver details
- **Undo/Redo**: Easy to implement with command history
- **Queuing**: Commands can be queued and executed later

## Problem It Solves
- Need to decouple request sender from receiver
- Implement undo/redo functionality
- Queue, log, and execute requests at different times
- Pass requests as parameters
- Support transactional operations

## Generic Implementation

### Basic Command Pattern

```csharp
// Command Interface
public interface ICommand
{
    void Execute();
}

// Receiver - The object that performs actual work
public class Light
{
    public void TurnOn() => Console.WriteLine("Light is ON");
    public void TurnOff() => Console.WriteLine("Light is OFF");
}

// Concrete Commands
public class TurnOnLightCommand : ICommand
{
    private Light _light;

    public TurnOnLightCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _light.TurnOn();
    }
}

public class TurnOffLightCommand : ICommand
{
    private Light _light;

    public TurnOffLightCommand(Light light)
    {
        _light = light;
    }

    public void Execute()
    {
        _light.TurnOff();
    }
}

// Invoker - Executes commands
public class RemoteControl
{
    private ICommand _command;

    public void SetCommand(ICommand command)
    {
        _command = command;
    }

    public void PressButton()
    {
        _command?.Execute();
    }
}

// Usage
var light = new Light();
var remote = new RemoteControl();

remote.SetCommand(new TurnOnLightCommand(light));
remote.PressButton(); // Light is ON

remote.SetCommand(new TurnOffLightCommand(light));
remote.PressButton(); // Light is OFF
```

### Undo/Redo Pattern

```csharp
// Enhanced Command with Undo
public interface IUndoableCommand
{
    void Execute();
    void Undo();
    string GetDescription();
}

// Receiver
public class Document
{
    private string _content = "";

    public void Write(string text)
    {
        _content += text;
        Console.WriteLine($"Document: '{_content}'");
    }

    public void Erase(int length)
    {
        if (length <= _content.Length)
        {
            _content = _content.Remove(_content.Length - length);
        }
        Console.WriteLine($"Document: '{_content}'");
    }

    public string GetContent() => _content;
}

// Concrete Commands
public class WriteCommand : IUndoableCommand
{
    private Document _document;
    private string _text;

    public WriteCommand(Document document, string text)
    {
        _document = document;
        _text = text;
    }

    public void Execute() => _document.Write(_text);
    public void Undo() => _document.Erase(_text.Length);
    public string GetDescription() => $"Write '{_text}'";
}

public class EraseCommand : IUndoableCommand
{
    private Document _document;
    private int _length;
    private string _erasedText;

    public EraseCommand(Document document, int length)
    {
        _document = document;
        _length = length;
    }

    public void Execute()
    {
        _erasedText = _document.GetContent();
        _document.Erase(_length);
        _erasedText = _erasedText.Substring(_erasedText.Length - _length);
    }

    public void Undo() => _document.Write(_erasedText);
    public string GetDescription() => $"Erase {_length} characters";
}

// Invoker with Undo/Redo
public class TextEditor
{
    private Stack<IUndoableCommand> _undoStack = new();
    private Stack<IUndoableCommand> _redoStack = new();

    public void ExecuteCommand(IUndoableCommand command)
    {
        command.Execute();
        _undoStack.Push(command);
        _redoStack.Clear(); // Clear redo when new command executed
    }

    public void Undo()
    {
        if (_undoStack.Count == 0) return;

        var command = _undoStack.Pop();
        Console.WriteLine($"Undo: {command.GetDescription()}");
        command.Undo();
        _redoStack.Push(command);
    }

    public void Redo()
    {
        if (_redoStack.Count == 0) return;

        var command = _redoStack.Pop();
        Console.WriteLine($"Redo: {command.GetDescription()}");
        command.Execute();
        _undoStack.Push(command);
    }
}

// Usage
var doc = new Document();
var editor = new TextEditor();

editor.ExecuteCommand(new WriteCommand(doc, "Hello"));
editor.ExecuteCommand(new WriteCommand(doc, " World"));
editor.ExecuteCommand(new EraseCommand(doc, 5));

editor.Undo();
editor.Undo();
editor.Redo();
```

### Command Queue

```csharp
// Request queuing and execution
public class CommandQueue
{
    private Queue<ICommand> _queue = new();

    public void Enqueue(ICommand command)
    {
        _queue.Enqueue(command);
        Console.WriteLine($"Command queued");
    }

    public void ExecuteAll()
    {
        Console.WriteLine("Executing all queued commands...");
        while (_queue.Count > 0)
        {
            var command = _queue.Dequeue();
            command.Execute();
        }
        Console.WriteLine("All commands executed");
    }

    public void Clear()
    {
        _queue.Clear();
        Console.WriteLine("Queue cleared");
    }
}

// Usage
var queue = new CommandQueue();
queue.Enqueue(new TurnOnLightCommand(new Light()));
queue.Enqueue(new TurnOffLightCommand(new Light()));
queue.ExecuteAll();
```

## Real-World Examples

### Macro Command (Composite Pattern + Command)

```csharp
// Macro - Command that contains multiple commands
public class MacroCommand : IUndoableCommand
{
    private List<IUndoableCommand> _commands = new();

    public void AddCommand(IUndoableCommand command)
    {
        _commands.Add(command);
    }

    public void Execute()
    {
        foreach (var command in _commands)
        {
            command.Execute();
        }
    }

    public void Undo()
    {
        for (int i = _commands.Count - 1; i >= 0; i--)
        {
            _commands[i].Undo();
        }
    }

    public string GetDescription() => $"Macro with {_commands.Count} commands";
}

// Usage
var macro = new MacroCommand();
macro.AddCommand(new TurnOnLightCommand(new Light()));
macro.AddCommand(new WriteCommand(new Document(), "Starting macro"));
macro.Execute();
```

### Async Command Execution

```csharp
public interface IAsyncCommand
{
    Task ExecuteAsync();
}

public class DownloadFileCommand : IAsyncCommand
{
    private string _url;
    private string _destination;

    public DownloadFileCommand(string url, string destination)
    {
        _url = url;
        _destination = destination;
    }

    public async Task ExecuteAsync()
    {
        Console.WriteLine($"Downloading {_url} to {_destination}");
        await Task.Delay(1000); // Simulate download
        Console.WriteLine("Download complete");
    }
}

public class AsyncCommandExecutor
{
    private Queue<IAsyncCommand> _queue = new();

    public void QueueCommand(IAsyncCommand command)
    {
        _queue.Enqueue(command);
    }

    public async Task ExecuteAllAsync()
    {
        while (_queue.Count > 0)
        {
            var command = _queue.Dequeue();
            await command.ExecuteAsync();
        }
    }
}
```

### Transaction-Like Execution

```csharp
public class Transaction
{
    private List<ICommand> _commands = new();
    private bool _successful = true;

    public void AddCommand(ICommand command)
    {
        _commands.Add(command);
    }

    public void Execute()
    {
        try
        {
            foreach (var command in _commands)
            {
                command.Execute();
            }
            Console.WriteLine("Transaction successful");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Transaction failed: {ex.Message}");
            _successful = false;
        }
    }

    public bool IsSuccessful => _successful;
}
```

### Button Binding

```csharp
public class Button
{
    private ICommand _command;

    public void SetCommand(ICommand command)
    {
        _command = command;
    }

    public void Click()
    {
        _command?.Execute();
    }
}

public class Menu
{
    private Dictionary<string, ICommand> _commands = new();

    public void AddCommand(string name, ICommand command)
    {
        _commands[name] = command;
    }

    public void ExecuteCommand(string name)
    {
        if (_commands.ContainsKey(name))
        {
            _commands[name].Execute();
        }
    }
}

// Usage
var menu = new Menu();
var light = new Light();

menu.AddCommand("LightOn", new TurnOnLightCommand(light));
menu.AddCommand("LightOff", new TurnOffLightCommand(light));

menu.ExecuteCommand("LightOn");
menu.ExecuteCommand("LightOff");
```

## Command vs Strategy

| Aspect | Command | Strategy |
|--------|---------|----------|
| **Purpose** | Encapsulate request as object | Encapsulate algorithm |
| **Execution** | Often invoked later | Invoked immediately |
| **Undo/Redo** | Native support | Needs additional support |
| **Queueing** | Supports naturally | Not typical use case |
| **Decoupling** | Sender from receiver | Context from algorithm |

## Advantages

✅ **Decoupling**: Invoker doesn't need to know receiver  
✅ **Undo/Redo**: Easy to implement with command history  
✅ **Queueing**: Commands can be queued and delayed  
✅ **Logging**: Can log all executed commands  
✅ **Transactions**: Group commands together  

## Disadvantages

❌ **Extra Classes**: One class per command type  
❌ **Complexity**: More objects to manage  
❌ **Memory**: Command objects accumulate  
❌ **Overhead**: Wrapping every action  

## When to Use

- Need undo/redo functionality
- Queue, log, or schedule execution
- Decouple request sender from receiver
- Support transactional operations
- Implement macro/composite operations

## When NOT to Use

- Simple direct method calls sufficient
- No undo/redo needed
- Tight coupling acceptable
- Performance critical

## Best Practices

1. **Keep Commands Simple**: One responsibility per command
2. **Store Necessary State**: Command must remember what it needs
3. **Use Command Objects as Data**: Pass like parameters
4. **Implement Memento for Undo**: Store state for restoration
5. **Document Side Effects**: Note what command modifies

## Summary

The Command Pattern wraps requests as objects, enabling delayed execution, queueing, logging, and undo/redo functionality. It's particularly powerful for implementing complex user interactions, transaction systems, and asynchronous command execution.
