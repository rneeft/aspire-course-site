---
title: .NET Aspire
layout: default
nav_order: 2
has_children: true
permalink: Aspire/
---

# .NET Aspire

So far you have run the entire platform with Docker Compose. Docker Compose is great for production-like environments, but during development you want:

- **Automatic service discovery** — no hardcoded hostnames or ports
- **A live dashboard** — see all services, their logs, traces, and metrics in one place
- **Dependency ordering** — wait for SQL Server to be ready before starting APIs, without writing health-check scripts

.NET Aspire provides all of this for **local development**.

{: .note }
> .NET Aspire is a development-time tool. It does not deploy to production. The Docker Compose setup remains the production/demo configuration.

## The two Aspire projects

The solution already contains the Aspire projects under `src/Aspire/`:

| Project | Purpose |
|---------|---------|
| `AspireAppHost` | Orchestrates all services for local development. Edit `AppHost.cs` to wire up resources. |
| `AspireServiceDefaults` | Shared defaults (health checks, telemetry, service discovery). Referenced by every API project. |

You will not modify `AspireServiceDefaults` — it is already configured.

## Install the Aspire workload

Check whether the Aspire workload is installed:

```bash
dotnet workload list
```

If `aspire` is not in the list, install it:

```bash
dotnet workload install aspire
```

## Add ServiceDefaults to each service

Every service needs two lines added to its `Program.cs` so Aspire can register health checks, telemetry, and service discovery.

In each of the five services (`IdentityApi`, `PseudoniemApi`, `DossierApi`, `PatientWebsite`, `Data.MigrationService`), open `Program.cs` and add:

```csharp
// immediately after: var builder = WebApplication.CreateBuilder(args);
builder.AddServiceDefaults();
```

And after `var app = builder.Build()`:

```csharp
app.MapDefaultEndpoints();
```

{: .note }
> `Data.MigrationService` is a `IHostBuilder` worker, not a web app. Add only `builder.AddServiceDefaults()` — there is no `app.MapDefaultEndpoints()` call.

When that is done, continue to the [AppHost setup](dependencies).