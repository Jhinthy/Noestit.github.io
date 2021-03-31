---
layout: single
title: 'TDD unit testing'
permalink: /codeconventions/tdd-unit-testing

sidebar:
  nav: "codeconventions"
---

## Introduction

In order maximize testability of our code, we follow these rules and best practices.

## Best practices

### Write the test first

Write the tests first, then the code. This ensures that you write testable code and that every line of code gets tested.

### Seperate UI code from its behavior using MVC or MPV

This allows the business logic to be tested while parts that can't be unit-tested (UI) is minimized.

### Design classes using dependency injection

When an object creates its own dependencies, you have no control over them. Dependency injection (IoC/DI) allows you to isolate the object under test with mocks/stubs.

### Program against interfaces, not classes

Using interfaces clarifies the relationship between objects and provides a great deal of flexibility to the developer and support for various mock object frameworks. Not using interfaces to inject dependencies to the constructor makes it hard to sometimes impossible to isolate certain implementations for unit-testing.

### Implement continuous integration

Build your software and run your full suite of tests on every check in, pull-request, merge, etc.

## Naming convention

A consistent naming convention helps to avoid comments and increases maintainability and identifiability. When a test fails you know exactly what functionality has been broken.

We try to follow this convention:

```
<Method Under Test>_<Given>_<Expected>
```

## Characteristics of a good unit test

* Fast: It's common for big projects to have thousands of unit tests. They should take little time to run. Milliseconds.
* Isolated: Unit tests are stand alone, can be run in isolation, and have no dependencies on any outside factors.
* Repeatable: Running a unit test should be consistent with its results. Always return same result if nothing has changed between runs.
* Self-Checking: The test should detect if it has passed or failed without any human interaction.
* Timely: A unit test should not take a disproportionally long time to write compared to the code being tested.

## Frameworks and tools

### Frameworks

There are several unit-testing frameworks available for .NET, including:

* MSTest/Visual Studio (basic)
* NUnit
* xUnit.NET
* MbUnit

[See this post](https://stackoverflow.com/questions/276829/what-can-i-use-for-good-quality-code-coverage-for-c-net "Stack overflow") for more information on different frameworks.

We prefer to use the combination of `xUnit`, `Moq` and `FluentAssertions` in our projects.

More info about the `FluentAssertions` framework [here](https://fluentassertions.com/).

### Mocking with Moq

Mock objects allow you to mimic the behaviour of classes and interfaces, so you can let your tests interact with the code as if it was real. This isolates the code you are testing and makes sure no other code will make the tests fail.

**The Setup method**

The Setup method is used to set expectations on the mock object.

```csharp
userRepositoryMock.SetUp(x => x.IsDeveloper("Tim")).Returns(true);
```
With this code, we are setting the `IsDeveloper` method on the mocked repository object.
We define that, when the parameter is "Tim", the method should return `true`. In this case we tell the mock exactly what we want returned.

```csharp
userRepositoryMock.SetUp(x => x.IsDeveloper(It.IsAny<Guid>())).Returns(It.IsAny<bool>());
```

With the code example above, we don't tell exactly what we want returned. We state that when a `Guid` is passed as a parameter, a `bool` must be returned.


### Tools and extensions

Tools used to check unit testing / code coverage. Most of these tools require a purchase, and/or have a free trial period.

* Ncrunch
* NCover
* dotCover

 [See this post](https://stackoverflow.com/questions/261139/nunit-vs-mbunit-vs-mstest-vs-xunit-net "Stack overflow") for more information on different tools.

Preferred scenario is to use NUnit as unit-testing framework with ReSharper for integration in VS, though ReSharper is very expensive.

When building with a CI tool like Jenkins and you don't need code coverage inside Visual Studio, you can use the free Jenkins [Cobertura plugin](https://wiki.jenkins.io/display/JENKINS/Cobertura+Plugin) to visualize coverage reports, use the free [`OpenCover`](https://www.nuget.org/packages/OpenCover/) package to generate a coverage report while running `NUnit` after which it should run the free tool [`OpenCoverToCoberturaConverter`](https://www.nuget.org/packages/OpenCoverToCoberturaConverter/) to convert the OpenCover report to a report the Jenkins plugin understands. Also see [this blogpost](https://www.swtestacademy.com/jenkins-dotnet-integration/).
