---
title: Processing
parent: Messaging
layout: default
nav_order: 4.3
---

## Processing


## Step 1: Add a Console Application for Processing

- Create a new console application
- Add a reference to the `InsuranceDetails.Messages` class library

- Add the following packages to the console application
```bash
dotnet add package Microsoft.Extensions.Hosting
dotnet add package NServiceBus
dotnet add package NServiceBus.Transport.SqlServer
dotnet add package NServiceBus.Extensions.Hosting
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```
- Replace `program.cs` with:

```csharp
using InsuranceDetails.Processor.Database;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using NServiceBus.Features;

var builder = Host.CreateApplicationBuilder(args);

builder.Configuration.AddEnvironmentVariables();

var insuranceDetailsDbConnectionString = builder.Configuration.GetConnectionString("InsuranceDetailsDb") ?? 
    throw new InvalidOperationException("No connection string configured");

builder.Services.AddDbContext<AppDbContext>(options => options.UseSqlServer(insuranceDetailsDbConnectionString));

var messagesDbConnectionString = builder.Configuration.GetConnectionString("MessagesDb") ??
    throw new InvalidOperationException("No connection string configured");

var endpointConfiguration = new EndpointConfiguration("DataFileProcessorEndpoint");
endpointConfiguration.UseSerialization<SystemJsonSerializer>();
endpointConfiguration.DisableFeature<AutoSubscribe>();
endpointConfiguration.DisableFeature<Audit>();
endpointConfiguration.SendFailedMessagesTo("Error");

// for demo purposes, we limit concurrency to 1
endpointConfiguration.LimitMessageProcessingConcurrencyTo(1);

var transport = endpointConfiguration.UseTransport<SqlServerTransport>()
    .QueuePeekerOptions(delay: TimeSpan.FromSeconds(10), peekBatchSize: 1) // let's slow this down for demo purposes
    .ConnectionString(messagesDbConnectionString)
    .DefaultSchema("dbo")
    .Transactions(TransportTransactionMode.SendsAtomicWithReceive);

var delayedDelivery = transport.NativeDelayedDelivery();
delayedDelivery.TableSuffix("Delayed");

builder.UseNServiceBus(endpointConfiguration);

await builder.Build().RunAsync();
```

The errors visible will be fixed later.

## Step 2: The Database
Lets copy some files to handle the database actions in the processor

- Create a folder `Database`
- Copy the `AppDbContext`, `Citizen`, `BasicHealthInsurance`  and `SupplementaryHealthInsurance` classes from Api to the Processor
- Update the namespaces to `InsuranceDetails.Processor.Database` in the copied classes
- Remove the `HealthInsurer` navigation properties from `BasicHealthInsurance` and `SupplementaryHealthInsurance`
- In `AppDbContext` remove the `HealthInsurer` lines and `Log` lines


## Step 3: Add a Command Handlers

In the processor project, add two new command handlers.

`UpdateBasicHealthInsuranceCommandHandler`
```csharp
using InsuranceDetails.Messages.Commands;
using InsuranceDetails.Processor.Database;
using Microsoft.EntityFrameworkCore;

namespace InsuranceDetails.Processor;

public class UpdateBasicHealthInsuranceCommandHandler : IHandleMessages<UpdateBasicHealthInsuranceCommand>
{
    private readonly AppDbContext _dbContext;
    private readonly Random _random = new();

    public UpdateBasicHealthInsuranceCommandHandler(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task Handle(UpdateBasicHealthInsuranceCommand message, IMessageHandlerContext context)
    {
        var number = _random.Next(1,11);
        if (number == 1)
        {
            throw new InvalidOperationException("Something goes wrong. It happens.");
        }
        
        await Task.Delay(1_000, context.CancellationToken); // Simulate processing time
        
        var citizen = await FindCitizenAsync(message.Bsn);
        var basic = new BasicHealthInsurance
        {
            AsFromDate = message.AsFromDate,
            TillDate = message.TillDate,
            Citizen = citizen,
            HealthInsurerId = message.HealthInsuranceId
        };
        
        _dbContext.BasicHealthInsurances.Add(basic);
        await _dbContext.SaveChangesAsync(context.CancellationToken);
    }

    private async Task<Citizen> FindCitizenAsync(string bsn)
    {
        var citizen = await _dbContext.Citizens
            .FirstOrDefaultAsync(c => c.Bsn == bsn);
        
        if (citizen is null)
        {
            citizen = new Citizen
            {
                Bsn = bsn
            };
                
            _dbContext.Citizens.Add(citizen);
            await _dbContext.SaveChangesAsync();
        }

        return citizen;
    }
}
```

`UpdateSupplementaryHealthInsuranceCommandHandler.cs`
```csharp
public class UpdateSupplementaryHealthInsuranceCommandHandler : IHandleMessages<UpdateSupplementaryHealthInsuranceCommand>
{
    private readonly AppDbContext _dbContext;
    private readonly Random _random = new();

    public UpdateSupplementaryHealthInsuranceCommandHandler(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task Handle(UpdateSupplementaryHealthInsuranceCommand message, IMessageHandlerContext context)
    {
        var number = _random.Next(1,11);
        if (number == 1)
        {
            throw new InvalidOperationException("Something goes wrong. It happens.");
        }
        
        await Task.Delay(1_000, context.CancellationToken); // Simulate processing time
        
        var citizen = await FindCitizenAsync(message.Bsn);
        var newSupplementaryHealthInsurance = new SupplementaryHealthInsurance
        {
            Citizen = citizen,
            HealthInsurerId = message.HealthInsuranceId,
            AsFromDate = message.AsFromDate,
            TillDate = message.TillDate,
            WhatIsCovered = message.WhatIsCovered,
            PercentageCovered = message.PercentageCovered,
            MaxAmount = message.MaxAmount
        };
        
        citizen.SupplementaryHealthInsurances.RemoveAll(x => x.HealthInsurerId == message.HealthInsuranceId && x.WhatIsCovered == message.WhatIsCovered);
        _dbContext.SupplementaryHealthInsurances.Add(newSupplementaryHealthInsurance);
        await _dbContext.SaveChangesAsync(context.CancellationToken);
    }
    
    private async Task<Citizen> FindCitizenAsync(string bsn)
    {
        var citizen = await _dbContext.Citizens
            .FirstOrDefaultAsync(c => c.Bsn == bsn);
        
        if (citizen is null)
        {
            citizen = new Citizen
            {
                Bsn = bsn
            };
                
            _dbContext.Citizens.Add(citizen);
            await _dbContext.SaveChangesAsync();
        }

        return citizen;
    }
}
```

## Step 4: Add project to Aspire

Add a reference to the Processor in the Aspire AppHost
Add the Processor in the Aspire `program.cs`

```csharp
builder
    .AddProject<Projects.InsuranceDetails_Processor>("Processor")
    .WithReference(apiDatabase)
    .WaitFor(apiDatabase)
    .WithReference(messagesDb)
    .WaitFor(messagesDb);
```

## Step 5: Test
Let's review what we've accomplished with the Processor so far:

- We implemented command handlers in the processor project to handle incoming messages.
- We registered the processor project in Aspire and ensured it references the correct databases.

Now it's time to test the full flow:

1. **Start the Application**  
   Make sure all your services (API, processor, and databases) are running. Open the console for the processor application so you can observe the processor's logs.

2. **Upload DataFiles Using Postman**  
   Use Postman to authenticate and upload a DataFile to the API endpoint.  
   Try uploading the DataFile several times.

3. **Observe Message Processing**  
   Watch the processor's console logs. You should see messages being processed as each DataFile is handled. Each command is processed one at a time, and you may notice that processing is slow.

{: .highlight }
> The processing is currently slow because we limited concurrency for demonstration purposes. In the next section, we will update the Aspire configuration to improve performance.

## Step 6: Increase 

Updat the Processor code in Aspire Apphost

```csharp
builder
    .AddProject<Projects.InsuranceDetails_Processor>("Processor")
    .WithReplicas(5)
    .WithReference(apiDatabase)
    .WaitFor(apiDatabase)
    .WithReference(messagesDb)
    .WaitFor(messagesDb);
```

Aspire is now instructed to run the application `5` times, which will increase the message handling. 