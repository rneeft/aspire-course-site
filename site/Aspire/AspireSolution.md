---
title: Connect the solution
parent: .NET Aspire
layout: default
nav_order: 2.2
---

# Connect the solution
In the previous step, the Aspire projects were created. You will now add the rest of the application to the AppHost.
To start, let's add a reference to all the projects in the Aspire AppHost. 

In the `Aspire.AppHost` add a reference to all other projects, except for the shared library `Data`.

In `AppHost.cs` you can now reference a project by typing `Projects.` followed by the name of the project.

## Database Migrations

Let's first connect the database migration project to Aspire. Add the following lines below SQL but before the `builder.Build().Run();` line.

```csharp
var migration = builder.AddProject<Projects.Data_MigrationService>("Migrations")
    .WithReference(db)
    .WaitFor(db);
```

The above code will add the project to Aspire and adds a reference from the database to the migration service. In practice, this will add the connection string of the database as an environment variable to the project. 

Let's run the Aspire application to discover what changes.

After startup the migration will run and finish. In the console logging you will find more information about what it did. In the project information you will also find that Aspire added the environment variable `ConnectionStrings__OnlineToestemmingDb` with the information needed to connect.

Via SQL Management Studio you can verify that the database is created. 

## The rest of the application
Before we add the rest of the application to Aspire we first need to define a signing secret for the Identity API. We can use the same approach as for the SQL password, but since this is something we do not need to see, we will let Aspire generate one for us. Add the following code to `AppHost.cs`:

```csharp
var jwtSigningKey = new GenerateParameterDefault { MinLength = 44 }.GetDefaultValue();

var secret = builder.AddParameter("mysecret", jwtSigningKey, secret: true);
```

Then add the rest of the application:

```csharp
var identity = builder.AddProject<Projects.IdentityApi>("IdentityApi")
    .WithReference(db)
    .WithReference(migration)
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

var pseudoniem = builder.AddProject<Projects.PseudoniemApi>("PseudoniemApi")
    .WithReference(db)
    .WithReference(migration)
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

var dossier = builder.AddProject<Projects.DossierApi>("DossierApi")
    .WithReference(db)
    .WithReference(identity)
    .WithReference(pseudoniem)
    .WithReference(migration)
    .WaitFor(migration)
    .WithEnvironment("JwtSettings__SecretSigningKey", jwtSigningKey);

builder.AddProject<Projects.PatientWebsite>("PatientWebsite")
    .WithExternalHttpEndpoints()
    .WithReference(db)
    .WithReference(identity)
    .WithReference(migration)
    .WaitFor(migration)
    .WaitFor(identity);
```

Before we can run the application, we first need to add the Service Defaults project to the applications. 

## Service Defaults
In the `Aspire.ServiceDefaults` project, code is provided to easily integrate with Aspire, for example [OpenTelemetry](https://opentelemetry.io/). 

To all applications, add a reference to the `Aspire.ServiceDefaults` project.

In each project, add the following line to the `Program.cs` file just under `var builder = WebApplication.CreateBuilder(args);` line

```csharp
builder.AddServiceDefaults();
```

This method will add opinionated extensions to the application. 

After this is done, let's start the application.

## Testing

Just like with the Docker setup, we can test the setup with Bruno.