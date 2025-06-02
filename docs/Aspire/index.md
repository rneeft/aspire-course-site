---
title: .NET Aspire
layout: default
nav_order: 2
---

# Introduction .NET Aspire

.NET Aspire is a cloud-native development stack for building distributed applications with .NET. It provides tools, templates, and best practices to help developers create, configure, and deploy microservices and cloud-based solutions efficiently. Aspire focuses on simplifying service discovery, configuration, health checks, telemetry, for **local development**, making it easier to build scalable and maintainable applications for the cloud.

{: .note }
> .NET Aspire will only run on your local development machine. It won't be going to production.

Before we start with .NET Aspire make sure the templates are available on your machine

```bash
dotnet workload list
``` 

If aspire is not listed, install the workload using the following command:

```bash
dotnet workload install aspire
``` 

When everything is setup, lets create the .NET Aspire projects. Navigate to the source directory `src` and create two projects. If you create them in the command line, do not forget to add them to the solution:

```bash
dotnet new aspire-apphost -o InsuranceDetails.AppHost --framework net9.0
dotnet new aspire-servicedefaults -o InsuranceDetails.ServiceDefaults --framework net9.0
```

or create them in your preferred IDE. 

## InsuranceDetails.AppHost
The `InsuranceDetails.AppHost` project in .NET Aspire acts as the entry point and orchestrator for your distributed application during local development. It coordinates the startup and configuration of all your services, dependencies, and resources defined in your Aspire solution. The AppHost manages service lifecycles, wiring, and environment setup, making it easier to run and debug your entire microservices system locally.

## InsuranceDetails.ServiceDefaults
The `InsuranceDetails.ServiceDefaults` project in .NET Aspire provides a central place to define and share common configuration, dependencies, and best practices across all your microservices. It helps ensure consistency by applying default settings for things like logging, health checks, telemetry, and service discovery, so each service in your solution automatically inherits these defaults without duplicating configuration code. This streamlines setup and maintenance for distributed applications.

We will not touch the `InsuranceDetails.ServiceDefaults` in this workshop.

## Integration in the .API project
In the `InsuranceDetails.Api` project add a reference to the `InsuranceDetails.ServiceDefaults` and in the `InsuranceDetails.AppHost` project add a reference to the `InsuranceDetails.Api` project.

In the `program.cs` file of the api project add the following line of code just under the `var builder = ...` line:

```csharp
builder.AddServiceDefaults();
```

And the following line under the `var app = ...` line:

```csharp
app.MapDefaultEndpoints();
```

The `InsuranceDetails.Api` project is now ready to be included in the Aspire's `AppHost`. Replace the following code in the `InsuranceDetails.AppHost` `AppHost.cs`:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

builder.AddProject<Projects.InsuranceDetails_Api>("api");

builder.Build().Run();
``` 

With this change we are almost ready. The project also need a SQL Server and a SQL Database. Lets add that dependency in the [next section](dependencies).