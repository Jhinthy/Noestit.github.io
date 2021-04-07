---
layout: single
title: "Pattern matching"
permalink: /codeconventions/pattern-matching

sidebar:
  nav: "codeconventions"
---

## Pattern matching

### Is-expression

[C# 7](csharp7)

An is-expression can now have a pattern, instead of just a type.

The `is` and switch statements support pattern matching. The `is` keyword supports the following patterns:

- `type` pattern: tests if an expression can be converted to a specified type. If so it casts to a variable of that type.

```csharp
if(obj is int i)
  Console.Writeline("obj is an integer with value {i}");

// Cast to int via as keyword
if(obj is int){
    int i = obj as int;
    Console.Writeline("obj is an integer with value {i}");
}
```

- `constant` pattern: tests if an expression equals a specified constant. This can be a value, a declared const variable or an enumeration constant. Also checking for null is perfomed via constant pattern.

```csharp
const int threeDiceHighRoll = 18;

if(diceRoll is threeDiceHighRoll ){
  Console.WriteLine("To the max!");
}

if(diceRoll is 3){
    Console.WriteLine("Triple one");
}

if(diceRoll is null){
    Console.WriteLine("Did the die land on its corner?");
}
```

- `var` pattern: a match that always succeeds and binds the value of an expression to a new local variable. When the expresion evaluates to null, the is-expression produces true and assigns null to the variable.

An example: let's check if the square sides are a multiple of 2. Since patterns are constant expressions, the modulo calculation is not possible.

```csharp
if (shape is Square { Side: var side } square && side % 2 == 0)
{
    // `sqare` is a Square whose Side is a multiple of 2
}
```

**Note:** the `var` pattern will always match.

### Switch statement

[C# 7](csharp7)

```csharp
switch(obj){
  case string text:
      Console.Writeline("obj is a string with value {text}");
  case int number:
      Console.Writeline("obj is an integer with value {number}");
  default:
      throw new ArgumentException("Invalid data");
}
```

`When` pattern allows you to have additional conditions on the switch case.

```csharp
switch(number)
{
  case int value when value <= 0:
      Console.WriteLine("Less than or equal to 0");
      break;
  case int value when value > 0 && value <= 10:
      Console.WriteLine("More than 0 but less than or equal to 10");
      break;
  default:
      Console.WriteLine("More than 10");
      break;
}
```

```csharp
switch(shape)
{
  case Circle c:
      Console.Writeline($"circle with radius {c.Radius}");
      break;
  case Rectangle s when (s.Length == s.Height): // This one needs to be above the regular rectangle case, else it will never be hit.
      Console.Writeline($"Square with sides of lenght {s.Length}");
      break;
  case Rectangle r:
      Console.Writeline($"Rectangle with length {r.Length} and height{r.Height}");
      break;
  default:
      Console.Writeline("unknown shape");
      break;
  case null:
      throw new ArgumentNullException(nameof(shape));
}
```

**Note:** the order of case labels is important. Especially with the `with` statements.

### Switch expression

[C# 8](csharp8)

There are fewer repetitive keywords, and fewer curly braces needed to write a switch statement.

```csharp
// classic switch statement
switch (color)
{
    case Color.Red:
        return "The color is red";
    case Color.Blue:
        return "The color is blue";
    default:
        return "The color is unknown.";
}

// C# 8 switch expression
return color switch
{
    Color.Red => "The color is red",
    Color.Blue => "The color is blue",
    _ => "The color is unknown.",
};
```

**Note:** Consider readability when using switch expressions.

### Property pattern

[C# 8](csharp8)
The `property pattern` allows you to match on properties of the examined object.

```csharp
public static bool isNetDeveloper(Developer developer){
    return developer is { Language: ".NET" };
}
```

You will think you can achieve the same by chaining the properties. `if(developer.Language == ".NET")`. Indeed but there will be certain cases where this comes in handy. What if we also want to check the age of the person?

Consider this switch expression. First we use a type pattern to cast it to a `Point` (we did it twice) and with the when expression we added additional conditions.

```csharp
static string Display(object o)
    => o switch
    {
        Point p when p.X == 0 && p.Y == 0 => "origin",
        Point p                           => $"({p.X}, {p.Y})",
        _                                 => "unknown"
    };

static string Display(object o)
    => o switch
    {
        Point { X: 0, Y: 0 }         p => "origin",
        Point { X: var x, Y: var y } p => $"({x}, {y})",
        _                              => "unknown"
    };
```

We can even change the 'empty' property to a compact not-null pattern. The fallback case can be replaced as follows.

```csharp
{} => o.ToString(), // checks non-null objects
null => "null" // checks for null object
```

### Positional patterns

[C# 8](csharp8)

Background information.
Object deconstructing makes it possible to extract properties without accessing the properties. You can discard properties you don't need via `_`.

```csharp
public record Person
{
    public string Name { get; init; }
    public string City { get; init; }

    public void Deconstruct(out string name, out int age)
        => (name, age) = (Name, Age);
}

// Usage
var person = new Person { Name = "John Smith" , Age = 25 };
var (string name, int age) = person;
Console.WriteLine($"Hello {name}, looks like you're {age} years old");
```

If an object provides a deconstruct method, we can pattern match the object to match on the properties. It makes it possible to check the type and cast it to the type AND deconstruct it.

```csharp
if (person is Person(string name, int age))
{
    Console.WriteLine($"Person with name {name} is {age} years old");
}

if(person is (_, 65)){
    Console.WriteLine("You used to call me on the landline");
}
```

### Tuple patterns

[C# 8](csharp8)

The tuple patterns are a special case of positional patterns. They can be used when we want to match on multiple input values at the same time.

```csharp
public static string RockPaperScissors(string firstHand, string secondHand)
=> (firstHand, secondHand) switch
{
    ("rock", "paper") => "rock is covered by paper. Paper wins.",
    ("rock", "scissors") => "rock breaks scissors. Rock wins.",
    ("paper", "rock") => "paper covers rock. Paper wins.",
    ("paper", "scissors") => "paper is cut by scissors. Scissors wins.",
    ("scissors", "rock") => "scissors is broken by rock. Rock wins.",
    ("scissors", "paper") => "scissors cuts paper. Scissors wins.",
    (_, _) => "tie"
};
```

### Pattern matching enhancements C# 9

#### Logical patterns

[C# 9](csharp9)

The logical patterns now can make code unreadable when you start combining multiple checks. In c# 9 we can use the spoken equivalent of these operators. `||` becomes `or`, `&&` becomes `and` and `!` equals `not`;

Via the `not` keyword, the negated pattern helps to clarify, among other things, null checks.

```csharp

if(!(language is null)){
    // Do something
}

// becomes
if(language is not null){
    // Do something
}
```

`or` and `and` keywords

```csharp
int age = 15;
if ((age >= 0 && age <= 18) || (age >= 65 && age <= 9999))
{
    Console.WriteLine("Working age");
}
// This will also match the type
object age = 25;
if (age is (>= 0 and <= 18) or (>= 65 and <= 9999))
{
    Console.WriteLine("Working age");
}
```

**Note:** `and` and `or` pattern are not equivalent to eachother. They cannot be used in the same situations. Below are the variations between both.

- `and` cannot be placed between two type patterns (unless they are targeting interfaces)
- `or` can be placed between two type patterns but it doesn’t support capturing (type pattern matching)
- `and` cannot be placed in a property pattern without a relational one
- `or` can be placed in a property pattern without a relational one and supports capturing
- `or` cannot be used between two properties of the same object
- `and` cannot be used between two properties of the same object, but it is implicit

Clarified via code examples:

```csharp
shape is Square and Circle // this will not compile
shape is Square or Circle // OK!
shape is Square or Circle smt // this will not compile
shape is Square { Side: 0 and 1 } // this will not compile
shape is Square { Side: 0 or 1 } sq // OK!
shape is Rectangle { Height: 0 or Length: 0 } // this will not compile
shape is Rectangle { Height: 0 } or Rectangle { Length: 0 } // OK!
shape is Rectangle { Height: 0 and Length: 0 } // this will not compile
shape is Rectangle { Height: 0, Length: 0 } re // OK! equivalent to the pattern above
```

#### Relational patterns

[C# 9](csharp9)

Relational patterns enables the use of <, >, >=, <= in patterns.

```csharp
case int age when age > 0 && age <= 10:

// becomes
case > 0 and <= 10:

// switch expression
public static LifeStage LifeStageAtAge(int age)
    => age switch
    {
        < 0 =>  LifeStage.Prenatal,
        < 2 =>  LifeStage.Infant,
        < 4 =>  LifeStage.Toddler,
        < 6 =>  LifeStage.EarlyChild,
        < 12 => LifeStage.MiddleChild,
        < 20 => LifeStage.Adolescent,
        < 40 => LifeStage.EarlyAdult,
        < 65 => LifeStage.MiddleAdult,
        _ =>    LifeStage.LateAdult,
    };
```

### Wrap it up

```csharp
// C# 7 and earlier
if (person.Parent.DateOfBirth.Year > 1970 && person.Parent.DateOfBirth.Year < 1990 && person.Parent.LastName == "Smith"){}

// C# 8.0
if (person is { Parent: { DateOfBirth: { Year: var year }, LastName: "Smith" } } && year > 1970 && year < 1990){}

// C# 9 (relational pattern)
if (person is { Parent: { DateOfBirth: { Year: > 1970 and < 1990 }, LastName: "Smith" } }){}
```

Used links:
[Do more with patterns in #8](https://devblogs.microsoft.com/dotnet/do-more-with-patterns-in-c-8-0/)
[Pattern matching in C# 9](https://renatogolia.com/2020/10/31/pattern-matching-in-c-sharp-9/)
