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

Regions are a way to structure large blocks of code. However, since we believe in small methods and classes, we will not allow the usage of regions.

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

// Conditional expression
IsValid ? expression : second_expression;
```

### Input parameters in lambda expressions

For simple and short lambda expressions, we allow the use of `x` as input parameter name. For more complex expressions, use full names.

```csharp
// Good
var enabledUsers = users.Where(x => x.IsEnabled);
var enabledUsers = users.Where(user => user.IsEnabled);

// Bad
var enabledUsers = users.Where(w => w.IsEnabled);
```
