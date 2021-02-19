---
layout: single
title: 'Noest coding conventions'
permalink: /codeconventions/index

sidebar:
  nav: "codeconventions"
---

# Welcome to the Noest code conventions

In this section you can find our coding guidelines, best practices and conventions for software development in .NET. Click on a link in the sidebar to navigate through our modules. This is **live documentation** meaning it will be edited and extended on a frequent basis.



## EditorConfig

A lot of the code style rules can be enforced by the use of an `.editorconfig` file added to the solution.

An `.editorconfig` file for .NET projects can be found in our public [editorconfig repository](https://github.com/team-noest/editorconfig/tree/master/dotnet).

### Using an .editorconfig file in Visual Studio

To use this file, you need to install the `EditorConfig Language Service` extension in Visual Studio. While VS2017 should include native support for `.editorconfig` files, it appears not to work without this extension. Also, changes made to the `.editorconfig` file seems to only apply after restarting Visual Studio. Since Visual Studio 2019 this issue has been resolved.

## Naming conventions

### Use PascalCasing for event-, property, namespace-, class- and methodnames

```csharp
namespace Noest.UserManagement
{
    public class UserAccount
    {
        public string Name { get; set; }

        public void UpdateName(string name)
        {
            this.Name = name;
        }
    }
}
```

### Use camelCasing for method parameters and local variables

```csharp
public class Noest
{
    public void CreateUserAccount(string name, string position)
    {
        var userAccount = new UserAccount(name, position);
    }
}
```

### Use predefined type aliases instead of system type names

```csharp
// Good
string firstName;
int lastIndex;
bool isDeleted;

// Bad
String firstName;
Int32 lastIndex;
Boolean isDeleted;
```

### Interface names always start with capital letter I

The name should be a noun or adjective.

```csharp
public interface IGadgetRepository
{
}

public interface IGadgetService
{
}

public interface IGroupable
{
}
```

### Avoid using abbreviations.

Exceptions do exist such as `Id`, `Ip`, `Ftp`, `Uri` ...

```csharp
// Good
UserGroup userGroup;
Assignment taskAssignment;

// Bad
UserGroup userGrp;
Assignment tskAsgnmnt;

// Exceptions
CustomerId customerId;
XmlDocument xmlDocument;
UriPart uriPart;
```

### DO NOT use include parent class name within a property name.

```csharp
// Good
Customer.Name

// Bad
Customer.CustomerName
```

### DO NOT use Hungarian notation for any other type identification in identifiers

```csharp
// Good
int counter;
string name;

// Bad
int iCounter;
string strName;
```

### DO NOT use all-CAPS for constant or static readonly fields

```csharp
// Good
public const string DeveloperType = "BackEnd";
public static readonly string DeveloperLevel = "Intermediate";

// Bad
public const string DEVELOPERTYPE = "BackEnd";
public static readonly string DEVELOPER_LEVEL = "Intermediate";
```

### DO NOT use underscores in identifiers.

Exception: private (readonly) variables.

```csharp
// Good
public DateTime clientAppointment;
public TimeSpan timeLeft;

// Bad
public DateTime client_appointment;
public TimeSpan time_left;

// Exception: private (readonly) variables must start with an underscore
private DateTime _registrationDate;
private readonly TimeSpan _cookieLifetime = TimeSpan.FromDays(7);
```

### DO NOT omit acces modifiers

Declare all identifiers with the appropriate acces modifier instead of allowing the default
``` csharp
// Bad
void MethodName(string parameter);

// Good
private void MethodName(string parameter);
public void MethodName(string parameter);
```

### DO NOT suffix enum names with Enum

```csharp
// Good
public enum MustardBrand
{
    DevosLemmens,
    Tierenteyn,
    Wostyn,
    Bister
}

// Bad
public enum MustardBrandEnum
{
    DevosLemmens,
    Tierenteyn,
    Wostyn,
    Bister
}
```

### Data Transfer Objects

If you use DTO's in your application, we recommend that you should have one DTO class for each entity/domain object, where the name can be suffixed with `Dto`.

```csharp
// <EntityName> + <Dto>
AccountDto
LocationDto
```

### Async Methods

Don't suffix Async methods with `Async`.

```csharp
// Good
Task<User> GetUserById(int userId);
Task<User[]> GetAllUsers();

// Bad
Task<User> GetUserByIdAsync(int userId);
Task<User[]> GetAllUsersAsync();
```
