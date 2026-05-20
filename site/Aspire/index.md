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

In this section of the workshop we will introduce Aspire into the solution. 