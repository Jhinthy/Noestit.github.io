---
layout: single
title: 'Logging'
permalink: /codeconventions/logging

sidebar:
  nav: "codeconventions"
---

## Introduction

Logging is a critical part in any application. When the end-user faces an error or unexpected behavior in the application, you want to have a trace or at least some pointer at what went wrong. Logging exceptions with a stacktrace is very helpful in that regard, while for some complex operations, you might decide to log additional side information to better understand the program flow. To distinguish trace data from error data, any decent logging framework also logs a level with each message and allows you to decide which level of messages you actually want to store.

A lot of logging frameworks exist but two of them are quite popular in the .NET world:

* log4net [NuGet](https://www.nuget.org/packages/log4net/)
* Serilog [NuGet](https://www.nuget.org/packages/serilog/)
* Microsoft.Extensions.Logging [Microsoft docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0)

Compared to log4net, Serilog has additional support for [structured logging](https://github.com/serilog/serilog/wiki/Structured-Data) so could be the better choice for more complex applications.

Both of these frameworks have a lot of different output methods: they can write to a file, write to a console, write to Windows Event log, write as XML, write as plain-text, write to a database, write to telnet, and so on. Log4Net calls them `Appenders` while Serilog calls them `Sinks`. See [log4net features](https://logging.apache.org/log4net/release/features.html) or [Serilog sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks).

## Logging and unit-testing

In both frameworks, you can choose to create different loggers for different context (f.e. different classes) or create one logger for the entire application. In any case, when running unit-tests, you don't want to log anything to disk or any other output, so it's advisable to inject a logging interface instead of directly calling a static `Log` class from the code.

When a context logger per class is used, you can use this method:

```csharp
// With log4net:
class UserService
{
    private readonly ILog _logger;

    public UserService(/* other required dependencies */, ILog logger = null)
    {
        _logger = logger ?? LogManager.GetLogger(typeof(UserService));
    }

    public void SomeMethod()
    {
        _logger.Warn("Some warning");
    }
}

// With Serilog:
class UserService
{
    private readonly ILogger _logger;

    public UserService(/* other required dependencies */, ILogger logger = null)
    {
        _logger = logger ?? Log.ForContext<UserService>();
    }

    public void SomeMethod()
    {
        _logger.Warning("Some warning");
    }
}

// With Microsoft.Extensions.Logger
class UserService
{
    private readonly ILogger _logger;

    public UserService(/* other required dependencies */, ILogger<UserService> logger = null)
    {
        _logger = logger ?? Log.ForContext<UserService>();
    }

    public void SomeMethod()
    {
        _logger.LogWarning("Some warning");
    }
}
```

This way, at runtime, dependency injection will not inject a logger and a context specific logger will be created in the constructor. While in unit-tests you can pass a mock logger and even verify certain logging behavior.

### SeriLog

SeriLog is easy to set up, has a clean `API` and can be used in all recent .NET platforms.

**Configure SeriLog & Sinks**

Create the logger using a `LoggerConfiguration` object.

```csharp
Log.Logger = new LoggerConfiguration().CreateLogger();
Log.Information("Who will read this?");
```

The above code example creates a logger that doesn't log any event anywhere. To log events, a sink must be configured.
Sinks are configures using the `WriteTo` configuration object.

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();

//Log event
Log.Information("Now you can read this in the console!");
```

The example above will write information logs to the console.
You can configure multiple sinks, by chaining the `WriteTo` blocks. In the example below, a minimum log event level is passed to the configuration.

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimulLevel.Debug() //Override for logger
    .WriteTo.Console(restrictedToMinimumLevel: LogEventLevel.Information) //Override per sink
    .WriteTo.File("log.txt")
    .CreateLogger();
```

**Creating a custom sink***

A sink is nothing more than a `class` that implements `ILogEventSink`. The example below creates a `sink` that renders every message to the console, regardless of the log level.

```csharp
public class MyCustomSink: ILogEventSink
{
    private readonly IFormatProvider _formatProvider;

    public MyCustomSink(IFormatProvider formatProvider)
    {
        _formatProvider = formatProvider;
    }

    public void Emit(LogEvent logEvent)
    {
        var message = logEvent.RenderMessage(_formatProvider);
        Console.WriteLine($"{DateTimeOffSet.Now.ToString()} {message}");
    }
}
```

**Extensions for configuration**

When configuring a sink, a extension method can be provided.

```csharp
public static class MyCustomSinkExtensions
{
    public static LoggerConfiguration MyCustomSink(
              this LoggerSinkConfiguration loggerConfiguration,
              IFormatProvider formatProvider = null)
    {
        return loggerConfiguration.Sink(new MyCustomSink(formatProvider));
    }
}
```

You can now use sink in your `LoggerConfiguration`:

```csharp
var log = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.MyCustomSink()
    .CreateLogger();
```

## Additional resources

* [Ultimate log4net Tutorial for .NET Logging](https://stackify.com/log4net-guide-dotnet-logging/)
* [Serilog tutorial](https://blog.getseq.net/serilog-tutorial/)
