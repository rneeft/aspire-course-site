---
title: Aspire Dashboard
parent: .NET Aspire
layout: default
nav_order: 2.2
---

# Aspire Dashboard

When the AppHost starts, a browser window opens automatically with the Aspire Dashboard. Let's explore each section.

## Resources

The **Resources** tab lists every service, container, and executable managed by Aspire. For each resource you can see:

- Its current **state** (running, stopped, waiting)
- **Endpoint URLs** — click to open the service directly in your browser
- **Environment variables** injected by Aspire

This is where you would find the dynamically assigned port for IdentityApi, PseudoniemApi, and DossierApi — no need to hardcode them in Bruno.

{: .tip }
> Notice that SQL Server appears as a **container** resource and all .NET projects appear as **project** resources. Only SQL Server runs in Docker; everything else runs as a native process.

## Console logs

The **Console** tab streams the standard output of every service in one place. Select a resource from the dropdown to filter. This replaces having to `docker logs` each container individually.

## Structured logs

The **Structured Logs** tab shows logs emitted via `ILogger` in a searchable, filterable table. You can filter by:

- Service name
- Log level (Information, Warning, Error)
- Time range
- Free-text search

## Tracing

The **Tracing** tab shows distributed traces. Because all services use `AddServiceDefaults()`, OpenTelemetry is automatically configured and traces flow from PatientWebsite → IdentityApi, and from DossierApi → IdentityApi → PseudoniemApi.

Click a trace to see the full span tree. This makes it easy to measure latency and spot where time is spent across service boundaries — try creating a dossier and then inspect the trace to see the internal token fetch and pseudoniem lookup as child spans.

## Metrics

The **Metrics** tab shows runtime metrics such as:

- HTTP request rates and durations
- Error rates
- Process memory and CPU usage

These metrics are provided automatically by ASP.NET Core's built-in OpenTelemetry instrumentation enabled by `AddServiceDefaults()`.

---

Congratulations — you have replaced Docker Compose with .NET Aspire for local development. The same microservices that previously required manually managed environment variables and healthcheck scripts now start in the right order with full observability.
