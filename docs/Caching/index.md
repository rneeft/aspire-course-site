---
title: Caching
layout: default
nav_order: 5
---

# Speed up with Cache

Caching can significantly improve the performance and scalability of your application by reducing the load on databases and speeding up data retrieval. One popular caching solution is **Redis**, an in-memory data store.

## Implementing Redis Cache in .NET Aspire

### 1. Add Redis Dependency to AppHost

First, add the Redis dependency to your Aspire AppHost project. Install the NuGet package:

```bash
dotnet add package Aspire.Hosting.Redis --version 9.3.0
```

Then, update your `program.cs` in the AppHost project to include Redis:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var redis = builder.AddRedis("redis");

builder
    .AddProject<Projects.InsuranceDetails_Api>("api")
    .WithReference(redis);

builder.Build().Run();
```

This configuration will spin up a Redis container and make its connection string available to your API project.

### 2. Add Redis Client to Your API Project

Install the Redis client library in your API project:

```bash
dotnet add package Microsoft.Extensions.Caching.StackExchangeRedis
```

### 3. Configure Redis Cache in `Program.cs` of API Project

Add the Redis cache service in your API's `Program.cs`:

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("redis");
});
```

### 4. Use Redis Cache in Your Application

You can now inject and use `IDistributedCache` in your controllers or services:

```csharp
public class MyController : ControllerBase
{
    private readonly IDistributedCache _cache;

    public MyController(IDistributedCache cache)
    {
        _cache = cache;
    }

    [HttpGet("cache-example")]
    public async Task<IActionResult> Get()
    {
        var value = await _cache.GetStringAsync("my-key");
        if (value == null)
        {
            value = "cached-value";
            await _cache.SetStringAsync("my-key", value, new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            });
        }
        return Ok(value);
    }
}
```

With these steps, your .NET Aspire solution will use Redis for caching, improving performance and scalability.