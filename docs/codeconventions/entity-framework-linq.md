---
layout: single
title: 'EF & Linq'
permalink: /codeconventions/entity-framework-linq

sidebar:
  nav: "codeconventions"

toc: true
toc_sticky: true
---

## Introduction

This article describes some best practices and performance improving principles when using Entity Framework. In most of our applications, LINQ syntax is used to retrieve data from different sources. The built-in LINQ engine translates these queries into raw SQL queries. There are a few conventions and best practices that ensure a much better application performance when communicating with the database.

## LINQ syntax

There are two ways to write LINQ queries

* LINQ Query Expression
* LINQ Extension Methods (method based queries)

At Sweet Mustard, we aim to always use **LINQ Extension Methods**. Method based queries are very readable and allow you to use custom extension methods that will still read fine. Since not all operators are available when using LINQ Query Expressions, you quickly end up with a mixed syntax.

### LINQ Query example

```csharp
var query = (from p in dbContext.Products.AsNoTracking()
             where p.Name.Contains("foo")
             orderby c.Name
             select p).Into("MyTable");
```

### LINQ Exentsion Method example

```csharp
var query = dbContext.Products
    .AsNoTracking()
    .Where(p => p.Name.Contains("foo"))
    .OrderBy(p => p.Name)
    .Into("MyTable");
```

## Eager Loading vs Lazy Loading vs Explicit Loading

Entity Framework supports multiple ways to load related entities (that is, when a relation is defined in the models).

### When to use Eager Loading

* In one side of a one-to-many relationship where you always need the related entity, f.e. the category of a product.
* Generally when relations are not too heavy, eager loading will reduce the amount of queries sent to the server.

#### Example

You have a `User` table and a `UserDetails` table, the LINQ query below will fetch a single `User` entity including the related `UserDetails` entity/entities at once.

```csharp
var user = dbContext.Users
    .AsNoTracking()
    .Include(a => a.UserDetails)
    .FirstOrDefault(a => a.UserId == userId);
```

### When to use Lazy Loading

* Almost on every 'collection side' of one-to-many relations, f.e. products of a category.
* You know exactly you will not need a property.

**Example**

The LINQ query below does not return the `UserDetails` entity/entities along with the `User` unless the property gets explicitly read.

```csharp
var user = dbContext.Users
    .AsNoTracking()
    .FirstOrDefault(a => a.UserId == userId);
```

`UserDetails` will only be loaded when you explicit call for it:

```csharp
UserDetails userDetails = user.UserDetails;
```

Lazy loading can cause weird side effects and slow down you application as multiple database queries get executed behind the scenes. Therefore we do not recommend using lazy loading.

### When to use Explicit Loading

After disabling lazy loading in Entity Framework, there's still a way to load related entities by explicitly calling the `Load` method. There are two ways to use the `Load` method: Reference (load a single navigation property) and Collection (to load collection).

#### Example

```csharp
var user = dbContext.Users
    .AsNoTracking()
    .FirstOrDefault(a => a.UserId == userId);

// Reference (when there's only one UserDetails entity per user)
dbContext.Entry(user).Reference(u => u.UserDetails).Load();

// Collection (when there are many UserDetails entryies per user)
dbContext.Entry(user).Reference(u => u.UserDetails).Load();
```

## Entity Identity Management

Only query the data you need is the golden rule to make performant queries.

F.e. when you need a list of all first- and lastnames for all users, you want to limit the data that needs to be transferred by only getting the `Id`, `FirstName` and `LastName` column data:

```csharp
// Good
var users = dbContext.User
    .AsNoTracking()
    .Select(user => new
    {
        Id = user.Id,
        FirstName = user.FirstName,
        LastName = user.LastName
    })
    .ToArray();

// Bad
var users = dbContext.User
    .AsNoTracking()
    .ToArray();
```

## Filter data

You can filter the data to be retrieved using `DataLoadOptions.AssociateWith` so that only the required data is returned. Following example shows how `DataLoadOptions` can be used:

```csharp
using (var dataContext = new MyDbContext())
{
    var dataLoadOptions = new DataLoadOptions();
    dataLoadOptions.AssociateWith<Customer>(customer =>
        customer.Address.Where<Address>(address => address.PinCode == 500016));
    dataContext.LoadOptions = dataLoadOptions;
}
```

## Disable Object Tracking

You can disable object tracking for the entire `DbContext` by setting `dbContext.ObjectTrackingEnabled = false`, but when using DI, it's better to explicitly disable tracking on the queries itself using `AsNoTracking` as demonstrated in the examples above.

## Disable Optimistic Concurrency

Concurrency handling enables you to detect and resolve conflicts that arise out of concurrent requests to the same resource. You should turn it off when not needed, you can use the `UpdateCheck` property of the `ColumnAttribute` to turn it off:

```csharp
[Column(Storage="_Address", UpdateCheck=UpdateCheck.Never)]
```

## Analyze LINQ queries

Monitor the generated SQL and understand how your LINQ has been translated. You can turn on the `Log` property of the `DbContext` to see the generated SQL or use [LINQPad](https://www.linqpad.net/) to tune the query.

```csharp
dbContext.Log = Console.Out;
```

## Code First

### Data Annotations vs Fluent API

Everything you can configure with DataAnnotations (attributes) is also possible with Fluent API, but the reverse is not true. The limitations of DataAnnotations are quickly reached (except for extremely simple entity models). We aim to use Fluent API for most applications.

Because of Fluent API:

* Model classes aren't buffed with annotations
* Navigation properties don't need to be declared virtual
* Implementation logic stays in your `DbContext`

To write Fluent API configurations, just override the `OnModelCreating()` method of the `DbContext` in the application specific context class.

#### Example from our ASP.NET MVC: Repository Pattern demo application:

 ```csharp
namespace Store.Data
{
    public class StoreEntities : DbContext
    {
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder.Configurations.Add(new GadgetConfiguration());
            modelBuilder.Configurations.Add(new CategoryConfiguration());
        }

        public StoreEntities() : base("StoreEntities")
        {
        }

        public virtual DbSet<Gadget> Gadgets { get; set; }
        public virtual DbSet<Category> Categories { get; set; }

        public virtual void Commit()
        {
            base.SaveChanges();
        }
    }
}
```

### Entity Type Configurations

In the code snippet above, we see `EntityTypeConfiguration` classes added to the `modelBuilder`. In these configuration classes, we define the configuration for a specific entity.

```csharp
public class CategoryConfiguration : EntityTypeConfiguration<Category>
{
    public CategoryConfiguration()
    {
        ToTable("Categories");
        Property(c => c.Name).IsRequired().HasMaxLength(50);
    }
}
```
