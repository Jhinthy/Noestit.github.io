Where will all this beauty end up in the final version?

## Init-only properties

[C# 9](#csharp9)

Init only setters are used when members of an object need to be immutable. The `init` accessor can be used on properties and indexers. The property can be set on initialization, but is protected from mutation after, thus becomes read-only.

You can declare init only setters for `class` or `struct`.

```csharp
public class Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
}

// Good
var person = new Person { FirstName = "John", LastName = "Smith" };

// Error
var person = new Person { LastName = "Smith" };
person.FirstName = "John";
```

## Records

[C# 9](#csharp9)

If you want your data model to be immutable a record can help you out with that.

A record is still a class, but the `record` keyword extends it with value-like behaviours. Records are defined by the content, by that they lean closer to a struct, but it's still a reference type.

```csharp
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
}

// Shorthand definition
public record Person(string FirstName, string LastName);

// Good
var person = new Person("John", "Smith");

// Error
var person = new Person();
person.FirstName = "John";
```

Inheritance:

```csharp
public sealed record Developer : Person
{
    public string Language;

    public Developer(string FirstName, string LastName, string Language)
        : base(FirstName, LastName) => Language = Language;
}
```

### With expression

By using the `with` expresion, you essentialy make a copy of the record, and creates a new `record` where the properties of the original are copied and specified properties modified, in this case the first name. The original record has not been changed.

Let's say you want to create a family tree starting with your siblings. Your last name is the same, you just want to change the first name.

```csharp
Person sibling1 = person with { FirstName = "Jane" };
Person sibling2 = person with { FirstName = "Marcel" };
```

This operation is also called _non-destructive mutation_. The with-expression works by actually copying the entire state of the old object into a new one and then mutating it according to the object initializer.

### Usefull links

[C#9 on the record](https://devblogs.microsoft.com/dotnet/c-9-0-on-the-record/)

## Is and As keywords

### Is

The `is` operator is used to check whether the result of an expression is compatible with the specified type. If on evaluation the expression can be converted the operator will return `true` if not it will return `false`;

```csharp
if(obj is int){
    Console.WriteLine("The object is of type int");
}
```

### As

The `as` operator converts to the compatible reference type.
Just as with the `is` operator it checks if the object can be cast to the type, if so it returns the object. Otherwise, null will be returned if conversion is not possible.

**Note:** This can only be used with reference types.

```csharp
// Explicit cast
object obj = "Noest";
var word = (string)obj;

SomeClass someObject = (SomeClass)obj;


//As operator
object obj = "Noest";
var word = obj as string;

SomeClass someObject = obj as SomeClass;
```
