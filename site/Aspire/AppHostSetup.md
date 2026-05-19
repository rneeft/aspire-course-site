---
title: AppHost Setup
parent: .NET Aspire
layout: default
nav_order: 2.1
---

# AppHost Setup

The `AspireAppHost` project (`src/Aspire/AspireAppHost/AppHost.cs`) is where you wire everything together. Open it now.

## Step 1 — SQL Server

Add the SQL Server NuGet package to the AppHost project:

```bash
dotnet add package Aspire.Hosting.SqlServer
```

Then declare the SQL Server resource. The `WithDataVolume` call persists data between restarts; `WithLifetime(Persistent)` keeps the container alive when the AppHost stops.

```csharp
var sqlPassword = builder.AddParameter("SqlPassword");

var sql = builder
    .AddSqlServer("sql", port: 2026, password: sqlPassword)
    .WithDataVolume("WorkshopAspire")
    .WithLifetime(ContainerLifetime.Persistent);

var db = sql.AddDatabase("OnlineToestemmingDb", databaseName: "OnlineToestemming");
```

Aspire will inject the connection string as `ConnectionStrings__OnlineToestemmingDb` into every service that references `db`.

## Step 2 — Migration Service

The MigrationService must run first and finish before any API starts:

```csharp
var migration = builder.AddProject<Projects.Data_MigrationService>("Migrations")
    .WithReference(db)
    .WaitFor(db);
```

`WaitFor(db)` means the MigrationService waits for SQL Server to be healthy. All other services will then `.WaitFor(migration)` so they start only after migrations complete.

## Step 3 — Shared JWT signing key

All three API services and the website share one JWT signing key. Generate it once and inject it everywhere:

```csharp
var jwtSigningKey = builder.Configuration["Parameters:mysecret"]
    ?? new GenerateParameterDefault { MinLength = 44 }.GetDefaultValue();

var secret = builder.AddParameter("mysecret", jwtSigningKey, secret: true);
```

## Step 4 — Wire up the services

Add each service and connect its dependencies. Notice how Aspire **service discovery** replaces the hardcoded Docker hostnames from `docker-compose.yml`:

```csharp
var identity = builder.AddProject<Projects.IdentityApi>("IdentityApi")
    .WithReference(db)
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

var pseudoniem = builder.AddProject<Projects.PseudoniemApi>("PseudoniemApi")
    .WithReference(db)
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

var dossier = builder.AddProject<Projects.DossierApi>("DossierApi")
    .WithReference(db)
    .WithReference(identity)    // injects services__IdentityApi__http__0
    .WithReference(pseudoniem)  // injects services__PseudoniemApi__http__0
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

builder.AddProject<Projects.PatientWebsite>("PatientWebsite")
    .WithExternalHttpEndpoints()
    .WithReference(db)
    .WithReference(identity)
    .WaitFor(migration)
    .WaitFor(identity);

builder.Build().Run();
```

`WithReference(identity)` tells Aspire to inject the IdentityApi base URL as an environment variable using the [Aspire service discovery format](https://learn.microsoft.com/en-us/dotnet/aspire/service-discovery/overview). No hardcoded hostnames or ports needed.

## Step 5 — Run the AppHost

Set `AspireAppHost` as the startup project and run it (F5 or `dotnet run`). The Aspire Dashboard will open automatically.

Aspire will:
1. Pull and start the SQL Server Docker container
2. Run the MigrationService and wait for it to complete
3. Start all four services in the correct order

Verify with:

```bash
docker ps
```

You should see the SQL Server container running. The .NET projects run as processes (not containers) in Aspire by default.

Continue to the [Dashboard](AspireDashboard).