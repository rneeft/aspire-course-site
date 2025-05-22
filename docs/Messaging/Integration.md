---
title: Integration
parent: Messaging
layout: default
nav_order: 4.1
---

# Integration

In this lesson, you will integrate NServiceBus into your solution to handle DataFile processing. When a health insurer uploads a DataFile, the API will validate it, split it into parts, and process each part as a command using NServiceBus.

---

## Step 1: Add a New Database for Messaging

In the Aspire AppHost, create a new database for messages and add a reference to your API project.

- Find the `nservicebus.sql` script in the `docs` folder.
- Add `nservicebus.sql` to the AppHost project and ensure it is copied to the output directory.
- Add the following code to `program.cs` in the AppHost:

```csharp
var sqlScript = await File.ReadAllTextAsync("nservicebus.sql");
var messages = sqlServer
    .AddDatabase("messages")
    .WithCreationScript(sqlScript);
```

- Add the messages database as a reference to the API project and use `WaitFor` to ensure it is ready.

---

## Step 2: Add NServiceBus Packages

Add the latest NuGet packages to your API project:

```bash
dotnet add package NServiceBus
dotnet add package NServiceBus.SqlServer
dotnet add package NServiceBus.Extensions.Hosting
```

---

## Step 3: Create a Shared Class Library

Create a new class library project for shared commands between projects.

- Name it something like `Messaging.Commands`.
- Add a `ProcessDataFile` command class to this library.
- Reference this library from both the API and processor projects.

---

## Step 4: Configure NServiceBus in the API

Add the following NServiceBus configuration to your API project:

```csharp
var endpointConfiguration = new EndpointConfiguration("DataFileUploadEndpoint");
endpointConfiguration.UseSerialization<SystemJsonSerializer>();
endpointConfiguration.DisableFeature<AutoSubscribe>();
endpointConfiguration.DisableFeature<Audit>();
endpointConfiguration.SendFailedMessagesTo("Error");

var transport = endpointConfiguration.UseTransport<SqlServerTransport>()
    .ConnectionString(connectionString)
    .DefaultSchema("dbo")
    .Transactions(TransportTransactionMode.SendsAtomicWithReceive);

var delayedDelivery = transport.NativeDelayedDelivery();
delayedDelivery.TableSuffix("Delayed");

var routing = transport.Routing();
routing.RouteToEndpoint(typeof(ProcessDataFile), "DataFileProcessorEndpoint");

builder.UseNServiceBus(endpointConfiguration);
```

---

## Step 5: Add a Console Application for Processing

Create a new console application to process the commands.

- Add the NServiceBus NuGet package to the console app.
- Replace `program.cs` with:

```csharp
var builder = Host.CreateApplicationBuilder(args);

var endpointConfiguration = NServiceBusConfig.Configuration("DataFileProcessorEndpoint");
builder.UseNServiceBus(endpointConfiguration);

await builder.Build().RunAsync();
```

- Copy any necessary database setup files and ensure the processor can access the messages database.

---

## Step 6: Add a Command Handler

In the processor project, add a new command handler for the `ProcessDataFile` command.

- Implement the handler by creating a class that implements `IHandleMessages<ProcessDataFile>`.
- Add your processing logic inside the `Handle` method.

---

## Next Steps

You have now set up NServiceBus for distributed DataFile processing. In the next lesson, you will test the integration and monitor message flow using the Aspire dashboard.