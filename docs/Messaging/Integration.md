---
title: Integration
parent: Messaging
layout: default
nav_order: 4.2
---

# NServiceBus Integration

In this lesson, you will integrate NServiceBus into your solution to handle DataFile processing. When a health insurer uploads a DataFile, the API will validate it, split it into parts, and process each part as a command using NServiceBus.

## Step 1: Add a New Database for Messaging

In the Aspire AppHost, create a new database for messages and add a reference to your API project.

- Find the `nservicebus.sql` script in the `docs` folder.
- Add `nservicebus.sql` to the AppHost project and ensure it is copied to the output directory.
- Add the following code to `program.cs` in the AppHost:

```csharp
var sqlScript = await File.ReadAllTextAsync("nservicebus.sql");
var messages = sqlServer
    .AddDatabase("MessagesDb")
    .WithCreationScript(sqlScript);
```

- Add the messages database `WithReference` to the API project and use `WaitFor` to ensure it wait for it to be ready.


## Step 2: Create a Shared Class Library

Create a new class library project for shared commands between projects.

- Name it something like `InsuranceDetails.Messages`.
- Add the NServiceBus package: `dotnet add package NServiceBus`
- Add a `Commands` folder and in the folder
- Add a `UpdateBasicHealthInsuranceCommand` class
```csharp
public class UpdateBasicHealthInsuranceCommand : ICommand
{
    public required string Bsn { get; init; }
    public required int HealthInsuranceId { get; init; }
    public required DateTime AsFromDate { get; init; }
    public required DateTime TillDate { get; init; }
}
```
- Add a `UpdateSupplementaryHealthInsuranceCommand` class
```csharp
public class UpdateSupplementaryHealthInsuranceCommand : ICommand
{
    public required string Bsn { get; init; }
    public required int HealthInsuranceId { get; init; }
    public required DateTime AsFromDate { get; init; }
    public required DateTime TillDate { get; init; }
    public required string WhatIsCovered { get; init; }
    public required int PercentageCovered { get; init; }
    public required int MaxAmount { get; init; }
}
``` 

- Reference this library from both the API and processor projects.

## Step 2: Add NServiceBus Packages

Add the latest NuGet packages to your API project:

```bash
dotnet add package NServiceBus
dotnet add package NServiceBus.Transport.SqlServer
dotnet add package NServiceBus.Extensions.Hosting
```

## Step 3: Configure NServiceBus in the API

Add the following NServiceBus configuration to your API project and make sure the method is called:

```csharp
static void AddNServiceBus(WebApplicationBuilder builder)
{
    var connectionString = builder.Configuration.GetConnectionString("MessagesDb") ?? 
                           throw new InvalidOperationException("No connection string configured");
    
    var endpointConfiguration = new EndpointConfiguration("DataFileUploadEndpoint");
    endpointConfiguration.UseSerialization<SystemJsonSerializer>();
    endpointConfiguration.SendOnly();

    var transport = endpointConfiguration.UseTransport<SqlServerTransport>()
        .ConnectionString(connectionString)
        .DefaultSchema("dbo")
        .Transactions(TransportTransactionMode.SendsAtomicWithReceive);

    var delayedDelivery = transport.NativeDelayedDelivery();
    delayedDelivery.TableSuffix("Delayed");

    var routing = transport.Routing();
    routing.RouteToEndpoint(typeof(UpdateSupplementaryHealthInsuranceCommand), "DataFileProcessorEndpoint");
    routing.RouteToEndpoint(typeof(UpdateBasicHealthInsuranceCommand), "DataFileProcessorEndpoint");

    builder.UseNServiceBus(endpointConfiguration);
}
```

## Step 5: Update the processing of the DataFile
We need to create an new implementation of the `IDataFileService` so that the messages are send in commands and not directly to the datbase.

- Add a new class to `InsuranceDetails.Api.DataFiles` names: `NServiceBusDataFileService`

```csharp
using FluentValidation;
using InsuranceDetails.Messages.Commands;

namespace InsuranceDetails.Api.DataFiles;

public class NServiceBusDataFileService : IDataFileService
{
    private readonly IValidator<DataFile> _validator;
    private readonly IMessageSession _messageSession;

    public NServiceBusDataFileService(IValidator<DataFile> validator, IMessageSession messageSession)
    {
        _validator = validator;
        _messageSession = messageSession;
    }

    public async Task<bool> ProcessDataFileAsync(DataFile dataFile, int healthInsurerId)
    {
        var validationResult = await _validator.ValidateAsync(dataFile);
        if (!validationResult.IsValid)
        {
            return false;
        }

        foreach (var citizenDto in dataFile.Citizens)
        {
            if (citizenDto.BasicHealthInsurance is not null)
            {
                var command = new UpdateBasicHealthInsuranceCommand
                {
                    Bsn = citizenDto.Bsn,
                    HealthInsuranceId = healthInsurerId,
                    AsFromDate = citizenDto.BasicHealthInsurance.AsFromDate,
                    TillDate = citizenDto.BasicHealthInsurance.TillDate
                };
                await _messageSession.Send(command);
            }

            foreach (var supplementaryInsurance in citizenDto.SupplementaryHealthInsurances)
            {
                var command = new UpdateSupplementaryHealthInsuranceCommand
                {
                    Bsn = citizenDto.Bsn,
                    HealthInsuranceId = healthInsurerId,
                    AsFromDate = supplementaryInsurance.AsFromDate,
                    TillDate = supplementaryInsurance.TillDate,
                    WhatIsCovered = supplementaryInsurance.WhatIsCovered,
                    PercentageCovered = supplementaryInsurance.PercentageCovered,
                    MaxAmount = supplementaryInsurance.MaxAmount
                };
                await _messageSession.Send(command);
            }
        }

        return true;
    }
}
```

- Register the new `NServiceBusDataFileService` to the Dependency Injector to file `DataFileServiceExtensions`.

## Step 5: Lets test the application
Let's review what we've accomplished so far with NServiceBus:

- We set up a dedicated messages database and configured NServiceBus to use it.
- We created shared command classes for processing health insurance data.
- We updated the API to send commands to NServiceBus when a DataFile is uploaded.
- We implemented a service that validates and splits the DataFile, sending each part as a command for processing.

Now it's time to test the integration:

1. **Start the Application**  
   Make sure all your services (API, processor, database) are running.

2. **Authenticate with the API**  
   Use Postman to log in to the service. If you don't have an account yet, register a new one.

3. **Create a Health Insurer**  
   Use the appropriate API endpoint in Postman to create a new HealthInsurer.

4. **Upload a DataFile**  
   Use the DataFile upload endpoint to send a file.  
   The API will validate the file and, if valid, send commands to NServiceBus for processing.

If everything is set up correctly, your uploaded DataFile will be processed asynchronously using NServiceBus, and you can monitor the results in your database and logs.

In order to see the data execute the following SQL query:

```sql
SELECT *
FROM [MessagesDb].[dbo].[DataFileProcessorEndpoint] WITH (NOLOCK)
```

## Next steps
The commands are now being send to `NServiceBus` in the next section a processor is made to handle the messages.