---
layout: single
title: "Folder structure"
permalink: /codeguidelines/folder-structure

sidebar:
  nav: "codeguidelines"
---

Logical folder structure on disk differs from the folder structure inside the solution because their purpose is different.

On disk, we want to keep unit-test projects separated from runtime projects whereas in the solution, we want to have unit-test projects as close as possible to their corresponding runtime projects.

## Logical folder structure

```markdown
<repository-root>
├ src/
| ├ Project1/
| | ├ ...
| | └ Project1.csproj
| ├ Project2/
| | ├ ...
| | └ Project2.csproj
| └ ...
├ tests/
| ├ Project1.Tests/
| | ├ ...
| | └ Project1.Tests.csproj
| ├ Project2.Tests/
| | ├ ...
| |  Project2.Tests.csproj
| └ ...
├ .gitignore
├ LICENSE (not needed for internal projects)
├ README.md
└ SolutionA.sln
```

## Solution structure

Depending on the type of project (simple service, 3-tier, CQRS/ES, ...) you might group multiple library projects in solution folders.

### Repository Pattern solution structure (3-tier sample)

A typicall 3-tier application can be arranged like the example below:

```markdown
ProjectName (solution folder)
├ DataLayer (Project or Folder)
| ├ Repositories (folder)
| | ├ ProductRepository.cs
| | └ IProductRepository.cs
| ├ Models (folder)
| | ├ Product.cs
| | └ User.cs
| └ DataAcces (folder)
| ├ UnitOfWork.cs
| └ IUnitOfWork.cs
├ ServiceLayer (Folder)
| ├ Services/
| | ├ ProductService.cs
| | └ UserService.cs
| └ Utils/
| ├ ViewModelFactory.cs
| └ ProductLogic.cs
├ PresentationLayer (Web.UI Project)
| ├ Views/
| | ├ Index.cshtml
| | └ \_userPartial.cshtml
| ├ Scripts/
| | ├ JavaScript.js
| | └ jQuery.min.js
| ├ Content/
| | ├ Site.css
| | └ Bootstrap.css
└ ...
```

### High-level solution structure (CQRS/ES sample)

A CQRS/ES application contains a command-side, a query-side and a shared library that contains the messages. Depending on the size of the project and size of the teams the application will consist of one or more solutions.

In the example below, command- and query-side has been put into a single solution:

```markdown
Solution
├ Command/ (solution folder)
├ Query/ (solution folder)
├ Solution Items/ (solution folder)
└ SomeNamespace.Messages (project)
```

### Command/query structure (CQRS/ES sample)

```markdown
Command/ (solution folder)
├ SomeNamespace.Command.Domain (project)
| ├ CommandHandlers/
| | ├ ProductCommandHandlers.cs
| | └ UserCommandHandlers.cs
| ├ Entities/
| | ├ Product.cs
| | └ User.cs
| └ Exceptions/
| ├ ProductExceptions.cs
| └ UserExceptions.cs
├ SomeNamespace.Command.Domain.Tests (project)
| ├ CommandHandlers/
| | ├ ProductCommandHandlersTests.cs
| | └ UserCommandHandlersTests.cs
| └ Entities/
| ├ ProductTests.cs
| └ UserTests.cs
├ SomeNamespace.Command.Host (servicehost or webhost project)
├ SomeNamespace.Command.Host.Tests (project)
└ ...
```

```markdown
Query/ (solution folder)
├ SomeNamespace.Query.Domain (project)
├ SomeNamespace.Query.Domain.Tests (project)
├ SomeNamespace.Query.Host (API project)
├ SomeNamespace.Query.Host.Tests (project)
└ ...
```

### T4 Templates (CQRS/ES sample)

```markdown
Solution Items/ (solution folder)
└ Templates/ (solution folder)
├ Base.tt
├ Header.tt
└ TypeDefinition.tt
```

### Messages project (CQRS/ES sample)

This project contains automatically generated command and event messages that are shared between command and query side.

```markdown
SomeNamespace.Messages (project)
├ Definitions/
| ├ Product.Commands.def
| ├ Product.Events.def
| ├ User.Commands.def
| └ User.Events.def
├ Commands.tt
| └ Commands.cs (generated)
└ Events.tt
└ Events.cs (generated)
```
