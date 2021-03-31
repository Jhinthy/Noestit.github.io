---
layout: single
title: "Best practice"
permalink: /codeconventions/best-practices

sidebar:
  nav: "codeconventions"
---

## Startup class and Services

### ServiceCollection Extension Pattern

.NET Core has a built-in DI (IoC) container. In the `ConfigureServices()` method in the `startup.cs` file in the host `ASP.NET Core` project, services are added to the DI container.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ....
    services.AddTransient<IUserService, UserService>();
    services.AddScoped<ISweetMustardService, SweetMustardService>();
    services.AddScoped<IProductRepository, ProductRepository>();
    services.AddScoped<IMustardRepository, MustardRepository>();
    services.TryAddSingleton<GraphQLQuery>();
    ...

	services.AddMvc();
}
```

`ConfigureServices` should be clean and readable. We can extract chunks of code from the startup class to extension methods.

**Example**

The `Data` layer in your solution holds repositories, and we need to add these to the DI container. We can create our own extension method that registers all the repositories. This method accepts parameter `this IServiceCollection` so we can bind our variables and return the `IServiceCollection` instance with the services registered.

```csharp
public static class ServicesConfiguration
{
	public static void AddRepositories(this IServiceCollection services)
	{
		services.AddTransient<IUserRepository, UserRepository>();
        services.AddTransient<IProductRepository, ProductRepository>();
        ...
	}
}
```

After referencing the `Data` project to your `ASP.NET Core` host project, you can add the services to the container in the `ConfigureServices()` method in the `startup.cs` file in a nice and clean way:

```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddMvc();
	services.AddRepositories();
}
```

> `IServiceCollection` is not part of the .NET Standard.

### Middleware

In `ASP.NET` we had modules, handlers a `web.config` file. In `ASP.NET Core` there is middleware, which is also set up in the startup class.

Middleware controls how the application responds `HTTP` requests, and we define these pieces in the `Configure()` method. We do this by using the `IApplicationBuilder` instance, and there are 4 methods to interact with a request.

**Use** adds a middleware to the pipeline and it can pass the request to the next delegate, or end it.
**Map** is used to connect a request path with another middleware.
**MapWhen** is almost the same as **Map**, but we can specify the detailed condition by using a `HttpContext` object.
**Run** adds a terminal middleware delegate to the application's request pipeline.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.Map("/route", HandleRouteMethod);

    app.Use(async delegate (HttpContext context, Func<Task> next)
    {
        await next.Invoke();
    });
}

private static void HandleRouteMethod(IApplicationBuilder app)
{
    app.Run(async context =>
    {
        await context.Response.WriteAsync("Sweetness from new route");
    });
}
```

#### Creating middleware

Basically, you move the code in the `Use` method to a separate class. To be able to use this we will have to make a extension method.

Example implementation of global error handling middleware:

```csharp
public class CustomExceptionMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<SampleMiddleware> _logger;

    public SampleMiddleware(RequestDelegate next, ILogger<SampleMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        try
        {
            await _next(httpContext);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception ...");
            await HandleExceptionAsync(httpContext, ex);
        }
    }
}
```

Extension method:

```csharp
public static class CustomExceptionMiddlewareExtension
{
    public static IApplicationBuilder UseCustomExceptionMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CustomExceptionMiddleware>();
    }
}
```

Just like the services in the `Service Collection Extension Pattern` section above, we can now add this middleware in 1 line to our startup class, in the `Configure()` method.

```csharp
public void Configure(IApplicationBuilder app)
{
    ...
    app.UseCustomExceptionMiddleware();

    app.Use(async delegate (HttpContext context, Func<Task> next)
    {
        await next.Invoke();
    });
}
```

## Validation

### Use data annotations for server-side validation

Use the `System.ComponentModel.DataAnnotations` namespace to access annotations for server side validation.

**Note:** server-side validation is mandatory.

In our example we demonstrate server side vaidation through `DataAnnotations` when creating a new user.

```csharp
public class CreateUserViewModel
{
    [Required(ErrorMessage="Username is mandatory")]
    [MaxLength(100)]
    public string Name { get; set; }
}
```

### Modelstate

In the controller, we can check if the `modelstate` is valid. `ModelState.IsValid` determines wether the submitted values statisfy all `DataAnnotation` validation attributes applied to the model properties.

```csharp
[HttpPost]
public ActionResult CreateUser(CreateUserViewModel createUserForm)
{
    if(!ModelState.IsValid)
    {
        //Code to handle invalid form
    }

    //Code to create user
}
```
