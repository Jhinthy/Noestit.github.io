---
layout: single
title: "Noest coding guidelines"
permalink: /codeguidelines/coding-style

sidebar:
  nav: "codeguidelines"
---

## Coding Style

### Tabs vs Spaces

We use the default indentation of 4 spaces, no tabs.

### Declare all member variables at the top of a class, with static/const members at the very top

In general, we follow this order:

- `private const`
- `private static readonly`
- `private readonly`
- `private`
- `public properties`

```csharp
public class Developer
{
    private const string Prefix = "NST";
    private static readonly string Seed = Cryptography.GenerateSeed(32);

    private readonly string _language;
    private int _age;

    public string Number { get; set; }
    public decimal Wage { get; set; }

    // Constructor
    public Developer()
    {
        // ...
    }
}
```

### Curly brackets below method/function/class definition

```csharp
// Good
public void SomeMethod()
{
    // ...
}

// Bad
public int CalculateHours() {
    // ...
}

// Exception: property declarations
public int Id { get; set; }

// Exception: if-statements with small one-line body
if (bookingService == null)
    throw new ArgumentNullException(nameof(bookingService));
```

### Order of namespaces

Order alphabetically but keep `System` directives on top.

### Summarize methods

Only summarize public methods that are exposed by the project.

This also includes ASP.NET Controller methods.

```csharp
/// <summary>
/// Description of what the method does.
/// </summary>
/// <param name="userId">An optional user id.</param>
/// <returns>Does the method return a view? A result value?</returns>
public IActionResult Index(int userId = null)
{
    // ...
}
```

Make sure to keep summaries up-to-date!

### Commenting

Writing and maintaining comments is expensive and it's the first thing that gets outdated during refactoring or bugfixes.

> If your feel your code is too complex to understand without comments, your code is probably just bad -- Sammy Larbi

Also read [coding without comments](https://blog.codinghorror.com/coding-without-comments/).

### Spacing between functions, methods and field declarations

Try to keep your code files structured. In short: have the amount of whitespace that is easy on the eye. **Think paragraph-like**.

```csharp
// Good
public void Method(int number)
{
    var variable = new Variable();
    variable.Property += number;

    if (statement)
    {
        variable.Boolean = true;
        return variable;
    }

    return variable;
}

public void AnotherMethod()
{
    // Do Stuff
}
```

```csharp
// Bad
public void Method(int number)
{
    var variable = new Variable();
    variable.Property += number;
    if(statement)
        return variable;
    return variable;
}
public void AnotherMethod()
{
    // Do Stuff
}
```

### Usage of regions

Regions are a way to structure large blocks of code. However, since we believe in small methods and classes, we will **not** allow the usage of regions.

### Working with lots of arguments

When a constructor or method takes more than 3 arguments, sort them under each other instead of creating a long unwrapped list.

```csharp
// Good
public AccountController(
    IUnitOfWork unitOfWork,
    IObjectService objectService,
    ILogger logger,
    IHelper helper)
{
    _unitOfWork = unitOfWork;
    // ...
}

// Bad
public AccountController(IUnitOfWork unitOfWork, IObjectService objectService, ILogger logger, IHelper helper)
{
    _unitOfWork = unitOfWork;
    // ...
}
```

Also, when calling constructors or methods with more than 1 argument, use the argument names in the call. This makes the code safe against refactoring.

```csharp
// Good
accountController.AddAccount(username: command.Username, email: command.Email);

// Bad
accountController.AddAccount(command.Username, command.Email);
```

### Value tuples

A tuple is a data structure that contains a sequence of elements of different data types. It can be used where you want to have a data structure to hold an object with properties, but you don't want to create a separate type for it. [source](https://www.c-sharpcorner.com/blogs/tuples-in-c-sharp)

Use the named members notation.

```csharp
// BAD
 (string, int) person1 = ("John Doe", 31);
 Console.WriteLine($"Tuple with elements {person1.Item1} and {person1.Item2}.");

// Good
(string Name, int Age) person2 = ("John Doe", 31);
Console.WriteLine($"Sum of {person2.Name} elements is {person2.Age}.");
```

Use cases:

- Instead of out parameters
- When you need to return two classes this could come in handy, you don't want to generate a composed class or a class that's only used for one method.
- If you only wish to return two values for a method and the class would only be used for that method, having a class might seem like too much.

```csharp
// Using out:
GetPerson(out string name, out int age);

object person = new {
    Name = name,
    Age = age
};

// Using Tuple class:
(string Name, int Age) person = GetPerson();
```

```csharp
(Customer Customer, Basket Basket) customerWithBasket = GetCustomerWithBasket(1);
```

**Note:** Be cautious when using `ValueTuples`

The use of C# 7 `ValueTuple` is only allowed in private methods. Any public method should not take or return a `ValueTuple`.

If a public method needs to take or return a combination of multiple fields, create a result or context class with the required fields.
The problem with tuples is that you can easily swap values without the compiler (or you) noticing. Consider the examples below:

```csharp
// Bad: tuple elements get mistakenly swapped in calling code
public (string FirstName, string LastName) GetPersonNameInfo()
{
    return (FirstName: "Bob", LastName: "Dylan");
}

(string LastName, string FirstName) result = GetPersonNameInfo();

// Bad: tuple elements get mistakenly swapped in method itself
public (string Firstname, string LastName) GetPersonNameInfo()
{
    return ("Dylan", "Bob");
}

// Good
public PersonNameInfo GetPersonNameInfo()
{
    return new PersonNameInfo
    {
        FirstName = "Bob",
        Lastname = "Dylan"
    };
}
```

### Usage of the ? operator

You can express calculations that otherwise need an `if-else` construction more concisely by using the conditional operator.

```csharp
bool IsValid;

// If-Else construction
if (isValid)
{
    expression;
}
else
{
    second_expression;
}

// Ternary conditional expression (?:)
IsValid ? expression : second_expression;
```

You can simplify nested null checks with the conditional operator.

```csharp
if (something != null)
{
    if (something.Other != null)
    {
        return something.Other.Child;
    }
}

// Conditional (?.) operator
return something?.Other?.Child;
```

The null-coalescing operator returns the left-side of the check is it isn't null. Else it returns the right side.

```csharp
if (something != null)
{
    return string.empty;
}

// Null-coalescing (??) operator
return something ?? string.empty;



int? number = null;
var n = number.HasValue ? number : 0;

// Null coalescing (??) operator
var n = number ?? 0;

// Alternative for numbers : GetValueOrDefault() if you want to return the default value of the underlaying value type.
var n = number.GetValueOrDefault();
```

We don't force one way or the other, just use what is the most readable / understandable for the given situation.

### Input parameters in lambda expressions

For simple and short lambda expressions, we allow the use of `x` as input parameter name. For more complex expressions, use full names.

```csharp
// Good
var enabledUsers = users.Where(x => x.IsEnabled);
var enabledUsers = users.Where(user => user.IsEnabled);

// Bad
var enabledUsers = users.Where(w => w.IsEnabled);
```

### Concatenating strings

Avoid using '+' to concatenate text into a new string.

```csharp
string name = "John";
string greetings = "Hello " + name + "!";

//Good: string.Format()
string greetingFormat = string.Format("Hello {0}!", name);

//Better: string interpolation (${})
string greeting = $"Hello, {name}!";
```

When you need to execute multiple changes to a string, creating a new String object will cause overhead. If you find yourself in this situation it is best to use `System.Text.StringBuilder` class.

If you want to use the [StringBuilder](https://docs.microsoft.com/en-us/dotnet/standard/base-types/stringbuilder) object you must pass it to a string object. This can easily be done via `.ToString()`.

```csharp
using System.Text;

StringBuilder stringBuilder = new StringBuilder("Hello! ");
stringBuilder.Append("What a beautiful day. ");
stringBuilder.AppendLine("Let's count numbers");

for (int i = 0; i < 4; i++)
{
    stringBuilder.AppendLine($"{i}.");
}

Console.WriteLine(stringBuilder.ToString());

// Hello! What a beautiful day. Let's count numbers
// 0.
// 1.
// 2.
// 3.
```
