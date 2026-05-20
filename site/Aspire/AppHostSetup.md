---
title: AppHost Setup
parent: .NET Aspire
layout: default
nav_order: 2.1
---

# AppHost Setup

Start with create a folder `Aspire` to keep things seperated. Inside the folder create two new projects and make sure they use the latests versions:

1. `Aspire.ServiceDefaults`
2. `Aspire.AppHost`

After the two projects are created make sure they use the latest nuget versions. The `Aspire.AppHost` project is not using Nuget, in the `.csproj` file edit the SDK version to the latest, 13.3.4. 

Let's start by running the `Aspire.AppHost` and see what we get out of the box.

If you get an error about using a unsecured transport, add the following line in the `launchSettings.json` file under `environmentVariables`:

```json
"ASPIRE_ALLOW_UNSECURED_TRANSPORT": "true"
```

# Adding the SQL Server
Aspire makes it really easy to setup your projects but also external dependencies, lets start by adding a SQL server to the resources. Add the following NuGet package to the project:

```
Aspire.Hosting.SqlServer
```

Next, replace the code in AppHost.cs with the following:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var sqlPassword = builder.AddParameter("SqlPassword");

var sql = builder
    .AddSqlServer("sql", port: 2026, password: sqlPassword)
    .WithDataVolume("AspireDataVolume");

var db = sql.AddDatabase("OnlineToestemmingDb", databaseName: "OnlineToestemming");


builder.Build().Run();
```
Let run the application and discover what Aspire did.

First it lets the developer create a password. When you start the solution, Aspire will ask for it first so we do not have to specify it. Provide a strong password in the Aspire UI and make sure it is saved to the user secrets. 

Secondly, it sets up the the AddSqlServer on port `2026`, uses a DataVolume names `WorkshopAspire`. Without a data volume all data is removed when you turn of your application.

Create the SQL-server might take some time depending on your machine and internet speed.

Lastly, it create a new database inside the SQL server. 



In docker also a couple of things has happend, run the following commands:

```bash
docker volume ls
```

This will show the volumns created inside docker, you will see you new volume here

```bash
docker ps
```

This command shows you all the containers running. If everything is correct you will have one container funning: `mcr.microsoft.com/mssql/server:2022-latest`. One thing to mention is the port forward: `127.0.0.1:32769->1433/tcp`. The port that is shown is not the same as the port setup in Aspire. This happens because Aspire provides a proxy between you and the docker instance. 

You can sign-in to the SQL server with SQL server management studio. Use the server name `127.0.0.1,2026`, `sa` and username and provide the password you used created to sign into your SQL database server.

In the object explorer you will see the newly created database: `OnlineToestemming`. 

Lets shutdown Aspire and implement some improvements into our setup.

After Aspire is stopped and you were to check docker again you will observe that SQL Server has stopped, also in SQL Magement Studio the server is become unavailable. 

This might become an issue, for example if you tests or other applications depending you the SQL Server. This can be solved by setting a lifetime to SQL Server. 

```csharp
.WithLifetime(ContainerLifetime.Persistent)
```

And while we are making improvements, lets disable the proxy support so that in docker we have the same outbound port:

```csharp
.WithEndpointProxySupport(proxyEnabled: false)
```

After you maked the changes restart the Aspire AppHost application, wait for the server to become available and execute the `docker ps` command again in the terminal.

You can see that the port will now be `2026`, `127.0.0.1:2026->1433/tcp` and when you stop the application it will keep running (until you shutdowns your machine)

The next step is connecting the solution to the Aspire app host.