---
title: Identity Provider
layout: default
nav_order: 3
---

# Identity Provider

In this section, we will learn how to separate responsibilities in our application by introducing an Identity Provider. In real-world scenarios, authentication is typically handled by an external service, such as [Azure Entra ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id), [AWS IAM](https://aws.amazon.com/iam/), or even a solution like DigiD.

## Step 1: Add a Database for Identity

Let's start by adding a new database to our Aspire application for identity management.  
Add the following code to your Aspire configuration:

```csharp
var identityDatabase = sqlServer
    .AddDatabase("IdentityDb");
```

## Step 2: Connect to the Database

Next, open a database management tool such as Azure Data Studio to inspect your SQL Server instance.

1. In Aspire, locate the resource for your SQL Server.
2. Copy the connection string from the environment setting, specifically the value for `ConnectionStrings__InsuranceDetailsDb`.  
   **Note:** Exclude the `InitialCatalog` part from the connection string.

![Environment-Setting](copy-connectionstring.png)

3. In Azure Data Studio, paste the connection string into the `New connection` dialog and click `Connect`.

![New Connection](new-connection.png)

## Step 3: Verify the Database

Once connected, you should see the databases listed, including the new `IdentityDb`. You can now view and manage tables within this database.

## Step 4: Create a New Web Application

Now, let's separate the identity logic from the main API project by creating a dedicated web application for identity management.

- Create a new WebApplication in your solution and name it `InsuranceDetails.IP`.

{: .highlight }
> This may seem counterintuitive but create the new web application **without authentication**.

## Step 5: Move Identity-Related Code

Move the necessary files from the old API project to the new Identity Provider project.  
This typically includes (but is not limited to):

- NuGet packages like `Microsoft.EntityFrameworkCore` and `Konscious.Security.Cryptography.Argon2`
- JWT generator
- PasswordHasher
- `appsettings` configuration

{: .highlight }
> Only move files related to authentication and identity management.

## Step 6: Clean Up the API Project

After moving the relevant files, remove them from the original API project to avoid duplication and confusion.

## Step 7: Add the project to Aspire and apply Aspire to the project

Register your new Identity Provider project in your Aspire AppHost configuration and make sure it connect to the correct database.

## Step 8: Test the connection

Verify that everything keeps running in Postman. Make sure to use the correct URL when connection to the identity provider.
