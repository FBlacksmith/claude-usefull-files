---
name: dotnet-clean-architecture
description: Scaffolds a production-ready .NET 10.0 LTS Clean Architecture solution with CPM, locked baseline packages, NsDepCop rules, and .editorconfig code styling.
argument-hint: [SolutionName]
disable-model-invocation: true
user-invocable: true
---

# .NET Clean Architecture Scaffolding Blueprint

This blueprint automates the creation of a production-ready, domain-driven, enterprise-grade .NET solution. It utilizes the **dotnet CLI**, **.NET 10.0 (LTS)**, **Central Package Management (CPM)**, **.NET Aspire**, **NsDepCop** for architecture linting, and **.editorconfig** for coding styles and standards formatting.

---

## 0. Interaction Rules & Token Economy
When executing scaffolding or coding tasks, strictly adhere to these behavioral rules:
*   **Surgery Mode:** When updating files, output ONLY the specific lines or blocks being changed. Do not recreate files unless instructed.
*   **No Prose:** Eliminate conversational filler, pleasantries, and explanations. Output only code, terminal commands, and minimal structural summaries.
*   **Explicit File Pathing:** Always provide the full, exact relative file path for any file being created, read, or modified.

## 1. Solution Setup & Scaffolding
Run the following commands using the `dotnet` CLI to initialize the solution structure cleanly under the current workspace:

```bash
# 1. Create the Solution file in the root
dotnet new sln -n $0

# 2. Create the clean projects (Zero boilerplate, targeting .NET 10.0 LTS)
dotnet new classlib -o src/$0.Core -f net10.0
dotnet new classlib -o src/$0.UseCases -f net10.0
dotnet new classlib -o src/$0.Infrastructure -f net10.0
dotnet new web -o src/$0.Web -f net10.0

# 3. Create the AppHost (Aspire Orchestrator) and Test project
dotnet new aspire-apphost -o src/$0.AppHost -f net10.0
dotnet new xunit -o tests/$0.Tests -f net10.0

# 4. Add all projects to the Solution
dotnet sln add src/$0.Core
dotnet sln add src/$0.UseCases
dotnet sln add src/$0.Infrastructure
dotnet sln add src/$0.Web
dotnet sln add src/$0.AppHost
dotnet sln add tests/$0.Tests
```

---

## 2. Central Package Management (CPM) Setup
Create a **`Directory.Packages.props`** file in the root directory of your solution. This manages all package versions centrally, ensuring strict governance, deterministic builds, and avoiding version mismatches.

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <!-- [Core] Layers Packages -->
    <PackageVersion Include="Ardalis.GuardClauses" Version="4.6.0" />
    <PackageVersion Include="Ardalis.Result" Version="10.1.0" />
    <PackageVersion Include="Ardalis.SharedKernel" Version="2.1.1" />
    <PackageVersion Include="Ardalis.SmartEnum" Version="8.1.0" />
    <PackageVersion Include="Ardalis.Specification" Version="8.0.0" />
    <PackageVersion Include="NimblePros.SharedKernel" Version="1.0.0" />
    <PackageVersion Include="Mediator.Abstractions" Version="2.1.7" />
    <PackageVersion Include="Vogen" Version="4.0.22" />

    <!-- [UseCases] Layers Packages -->
    <PackageVersion Include="FluentValidation" Version="11.9.0" />
    <PackageVersion Include="FluentValidation.DependencyInjectionExtensions" Version="11.9.0" />
    <PackageVersion Include="Mediator.SourceGenerator" Version="2.1.7" />
    <PackageVersion Include="MessagePack" Version="2.5.140" />

    <!-- [Infrastructure] Layers Packages -->
    <PackageVersion Include="Ardalis.Specification.EntityFrameworkCore" Version="8.0.0" />
    <PackageVersion Include="Azure.Identity" Version="1.11.4" />
    <PackageVersion Include="MailKit" Version="4.5.0" />
    <PackageVersion Include="MimeKit" Version="4.5.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.SqlServer" Version="10.0.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.Sqlite" Version="10.0.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.Relational" Version="10.0.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.Tools" Version="10.0.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.Design" Version="10.0.0" />
    <PackageVersion Include="Microsoft.EntityFrameworkCore.InMemory" Version="10.0.0" />

    <!-- [Web] Layers Packages -->
    <PackageVersion Include="Ardalis.ListStartupServices" Version="4.1.0" />
    <PackageVersion Include="Ardalis.Result.AspNetCore" Version="10.1.0" />
    <PackageVersion Include="AspNetCore.Localizer.Json" Version="8.0.0" />
    <PackageVersion Include="FastEndpoints" Version="5.24.0" />
    <PackageVersion Include="FastEndpoints.ApiExplorer" Version="5.24.0" />
    <PackageVersion Include="FastEndpoints.Swagger" Version="5.24.0" />
    <PackageVersion Include="FastEndpoints.Swagger.Swashbuckle" Version="5.24.0" />
    <PackageVersion Include="Microsoft.Extensions.Http.Resilience" Version="10.0.0" />
    <PackageVersion Include="Microsoft.Extensions.ServiceDiscovery" Version="10.0.0" />
    <PackageVersion Include="OpenTelemetry.Exporter.OpenTelemetryProtocol" Version="1.10.0" />
    <PackageVersion Include="OpenTelemetry.Extensions.Hosting" Version="1.10.0" />
    <PackageVersion Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.10.0" />
    <PackageVersion Include="OpenTelemetry.Instrumentation.Http" Version="1.10.0" />
    <PackageVersion Include="OpenTelemetry.Instrumentation.Runtime" Version="1.10.0" />
    <PackageVersion Include="Scalar.AspNetCore" Version="1.2.0" />
    <PackageVersion Include="Serilog.AspNetCore" Version="8.0.1" />
    <PackageVersion Include="Serilog.Sinks.ApplicationInsights" Version="4.0.0" />
    <PackageVersion Include="Serilog.Sinks.OpenTelemetry" Version="4.1.0" />

    <!-- [AppHost] Layers Packages -->
    <PackageVersion Include="Aspire.Hosting.AppHost" Version="10.0.0" />
    <PackageVersion Include="Aspire.Hosting.SqlServer" Version="10.0.0" />

    <!-- [Tests] Layers Packages -->
    <PackageVersion Include="Ardalis.HttpClientTestExtensions" Version="4.2.0" />
    <PackageVersion Include="Aspire.Hosting.Testing" Version="10.0.0" />
    <PackageVersion Include="coverlet.collector" Version="6.0.2" />
    <PackageVersion Include="Microsoft.AspNetCore.Mvc.Testing" Version="10.0.0" />
    <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
    <PackageVersion Include="NSubstitute" Version="5.1.0" />
    <PackageVersion Include="ReportGenerator" Version="5.2.4" />
    <PackageVersion Include="Shouldly" Version="4.2.1" />
    <PackageVersion Include="Testcontainers" Version="3.8.0" />
    <PackageVersion Include="Testcontainers.MsSql" Version="3.8.0" />
    <PackageVersion Include="xunit.v3" Version="0.1.1-pre.358" />
    <PackageVersion Include="xunit.runner.visualstudio" Version="3.0.0-pre.30" />

    <!-- [Code Analysis] Layers Packages -->
    <PackageVersion Include="NimblePros.Metronome" Version="1.1.3" />
    <PackageVersion Include="NsDepCop" Version="2.3.0" />
  </ItemGroup>
</Project>
```

---

## 3. Configuration Files Generation
In order to enforce enterprise style rules, formatting consistency, and package boundaries directly inside the newly created workspace, generate the following `.editorconfig` and `config.xml` files.

### 3.1. Generate `.editorconfig` (Code Styling & Standards)
Create a `.editorconfig` file in the root directory to enforce .NET 10 development style rules and code clean formatting standards:

```ini
# .editorconfig file for enterprise C# development
root = true

# All files
[*]
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8

# XML/JSON and configs
[*.{xml,json,props,targets,config}]
indent_size = 2

# Markdown
[*.md]
trim_trailing_whitespace = false

# C# Files styling rules
[*.cs]
# Organize Directives (Usings)
dotnet_sort_system_directives_first = true:suggestion
dotnet_separate_import_directive_groups = true:suggestion

# Var preferences
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = true:suggestion

# Formatting - Braces rules
csharp_new_line_before_open_brace = all:suggestion
csharp_new_line_before_else = true:suggestion
csharp_new_line_before_catch = true:suggestion
csharp_new_line_before_finally = true:suggestion

# Language Rules
csharp_prefer_simple_using_statement = true:suggestion
csharp_style_namespace_declarations = file_scoped:suggestion

# Naming Conventions
dotnet_naming_rule.interface_should_start_with_i.severity = error
dotnet_naming_rule.interface_should_start_with_i.symbols = interface_symbols
dotnet_naming_rule.interface_should_start_with_i.naming_style = interface_style_i

dotnet_naming_symbols.interface_symbols.applicable_kinds = interface
dotnet_naming_style.interface_style_i.required_prefix = I
dotnet_naming_style.interface_style_i.capitalization = pascal_case
```

### 3.2. Generate `config.xml` (NsDepCop Dependency Constraints)
Create an `NsDepCop` configuration file called `config.xml` in the root folder of the solution to actively monitor and block any boundary bypass:

```xml
<NsDepCopConfig InheritanceDepth="3">
  <!-- Default rules for .NET System namespaces -->
  <Allowed From="*" To="System.*" />
  <Allowed From="*" To="Microsoft.Extensions.*" />

  <!-- Core layer dependencies constraint -->
  <Allowed From="*.Core" To="System.*" />
  <Allowed From="*.Core" To="Ardalis.*" />
  <Allowed From="*.Core" To="NimblePros.SharedKernel" />
  <Allowed From="*.Core" To="Mediator.Abstractions" />
  <Allowed From="*.Core" To="Vogen" />
  
  <Forbidden From="*.Core" To="*.UseCases" />
  <Forbidden From="*.Core" To="*.Infrastructure" />
  <Forbidden From="*.Core" To="*.Web" />

  <!-- UseCases layer dependencies constraint -->
  <Allowed From="*.UseCases" To="*.Core" />
  <Allowed From="*.UseCases" To="FluentValidation.*" />
  <Allowed From="*.UseCases" To="MessagePack" />
  
  <Forbidden From="*.UseCases" To="*.Infrastructure" />
  <Forbidden From="*.UseCases" To="*.Web" />

  <!-- Infrastructure layer dependencies constraint -->
  <Allowed From="*.Infrastructure" To="*.UseCases" />
  <Allowed From="*.Infrastructure" To="*.Core" />
  <Allowed From="*.Infrastructure" To="Microsoft.EntityFrameworkCore.*" />
  <Allowed From="*.Infrastructure" To="Azure.Identity" />
  <Allowed From="*.Infrastructure" To="MailKit" />
  <Allowed From="*.Infrastructure" To="MimeKit" />

  <Forbidden From="*.Infrastructure" To="*.Web" />

  <!-- Web (Presentation) layer dependencies constraint -->
  <Allowed From="*.Web" To="*.UseCases" />
  <Allowed From="*.Web" To="*.Infrastructure" />
  <Allowed From="*.Web" To="FastEndpoints.*" />
  <Allowed From="*.Web" To="OpenTelemetry.*" />
  <Allowed From="*.Web" To="Scalar.*" />
  <Allowed From="*.Web" To="Serilog.*" />
</NsDepCopConfig>
```

---

## 4. Install Baseline Packages to Layer Projects
Run the following package install commands sequentially to associate the centrally managed versions to their corresponding architectural projects:

```bash
# Core layer packages
dotnet add src/$0.Core package Ardalis.GuardClauses
dotnet add src/$0.Core package Ardalis.Result
dotnet add src/$0.Core package Ardalis.SharedKernel
dotnet add src/$0.Core package Ardalis.SmartEnum
dotnet add src/$0.Core package Ardalis.Specification
dotnet add src/$0.Core package NimblePros.SharedKernel
dotnet add src/$0.Core package Mediator.Abstractions
dotnet add src/$0.Core package Vogen

# UseCases layer packages
dotnet add src/$0.UseCases package FluentValidation
dotnet add src/$0.UseCases package FluentValidation.DependencyInjectionExtensions
dotnet add src/$0.UseCases package Mediator.SourceGenerator
dotnet add src/$0.UseCases package MessagePack

# Infrastructure layer packages
dotnet add src/$0.Infrastructure package Ardalis.Specification.EntityFrameworkCore
dotnet add src/$0.Infrastructure package Azure.Identity
dotnet add src/$0.Infrastructure package MailKit
dotnet add src/$0.Infrastructure package MimeKit
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.Sqlite
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.Relational
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.Tools
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.Design
dotnet add src/$0.Infrastructure package Microsoft.EntityFrameworkCore.InMemory
dotnet add src/$0.Infrastructure package NsDepCop

# Web layer packages
dotnet add src/$0.Web package Ardalis.ListStartupServices
dotnet add src/$0.Web package Ardalis.Result.AspNetCore
dotnet add src/$0.Web package AspNetCore.Localizer.Json
dotnet add src/$0.Web package FastEndpoints
dotnet add src/$0.Web package FastEndpoints.ApiExplorer
dotnet add src/$0.Web package FastEndpoints.Swagger
dotnet add src/$0.Web package FastEndpoints.Swagger.Swashbuckle
dotnet add src/$0.Web package Microsoft.Extensions.Http.Resilience
dotnet add src/$0.Web package Microsoft.Extensions.ServiceDiscovery
dotnet add src/$0.Web package OpenTelemetry.Exporter.OpenTelemetryProtocol
dotnet add src/$0.Web package OpenTelemetry.Extensions.Hosting
dotnet add src/$0.Web package OpenTelemetry.Instrumentation.AspNetCore
dotnet add src/$0.Web package OpenTelemetry.Instrumentation.Http
dotnet add src/$0.Web package OpenTelemetry.Instrumentation.Runtime
dotnet add src/$0.Web package Scalar.AspNetCore
dotnet add src/$0.Web package Serilog.AspNetCore
dotnet add src/$0.Web package Serilog.Sinks.ApplicationInsights
dotnet add src/$0.Web package Serilog.Sinks.OpenTelemetry

# AppHost layer packages
dotnet add src/$0.AppHost package Aspire.Hosting.AppHost
dotnet add src/$0.AppHost package Aspire.Hosting.SqlServer

# Tests layer packages
dotnet add tests/$0.Tests package Ardalis.HttpClientTestExtensions
dotnet add tests/$0.Tests package Aspire.Hosting.Testing
dotnet add tests/$0.Tests package coverlet.collector
dotnet add tests/$0.Tests package Microsoft.AspNetCore.Mvc.Testing
dotnet add tests/$0.Tests package Microsoft.NET.Test.Sdk
dotnet add tests/$0.Tests package NSubstitute
dotnet add tests/$0.Tests package ReportGenerator
dotnet add tests/$0.Tests package Shouldly
dotnet add tests/$0.Tests package Testcontainers
dotnet add tests/$0.Tests package Testcontainers.MsSql
dotnet add tests/$0.Tests package xunit.v3
dotnet add tests/$0.Tests package xunit.runner.visualstudio
```

---

## 5. Configure Boundary Project References
Link the projects together so the dependency flow only points strictly inward toward `Core`:

```bash
# UseCases points only to Core
dotnet add src/$0.UseCases reference src/$0.Core

# Infrastructure points to UseCases (which transitively observes Core)
dotnet add src/$0.Infrastructure reference src/$0.UseCases

# Web acts as the executable Host and configures DI for Core, UseCases, and Infrastructure
dotnet add src/$0.Web reference src/$0.UseCases
dotnet add src/$0.Web reference src/$0.Infrastructure

# AppHost orchestrates Web as a project resource
dotnet add src/$0.AppHost reference src/$0.Web

# Tests targets Web integration flows
dotnet add tests/$0.Tests reference src/$0.Web
```

---

## 6. Code Analysis & Boundaries Validation
To ensure that developers or AI agents do not bypass architecture boundaries, configure `NsDepCop` execution during builds. Add this configuration to your layer projects (Core, UseCases, Infrastructure, Web) to make bounds checking fail the build instantly on violation:

```xml
<ItemGroup>
  <!-- NsDepCop runs locally inside projects to enforce XML boundaries constraints -->
  <PackageReference Include="NsDepCop" PrivateAssets="All" />
</ItemGroup>
```
When you run `dotnet build`, `NsDepCop` will automatically read the solution-root `config.xml` file and ensure strict compliance with Clean Architecture boundaries.
