---
title: Hello, World
layout: home
nav_order: 0
---

# Welcome to the Microservices Course with .NET Aspire

This website is the central hub for the **Microservices course with .NET Aspire**.

In this course, we build a simplified version of the **COV solution**. COV stands for **Controle op Verzekeringsgegevens** (Insurance Data Check). It is an application where health insurers upload their health insurance data. Healthcare providers use this software to check the health insurance status of their patients.

Explore the materials, follow the workshops, and learn how to design and implement microservices using .NET Aspire!


## Prerequisites

Before you begin, please ensure you have the following installed:

- [JetBrains Rider](https://www.jetbrains.com/rider/), [Visual Studio](https://visualstudio.microsoft.com/), or [Visual Studio Code](https://code.visualstudio.com/) with the [C# Dev Kit extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)
- [.NET 9 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/9.0)
- [.NET Aspire workload](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling)
- [Docker](https://www.docker.com/) or [Rancher Desktop](https://rancherdesktop.io/)
- [Postman](https://www.postman.com/) with account or [Insomnia](https://insomnia.rest/)
- [Azure Data Studio](https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio) or [SQL Server Management Studio](https://aka.ms/ssms)

Before we start with .NET Aspire make sure the templates are available on your machine

```bash
dotnet workload list
``` 

If aspire is not listed, install the workload using the following command:

```bash
dotnet workload install aspire
``` 