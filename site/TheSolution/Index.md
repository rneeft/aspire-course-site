---
title: The Solution
layout: default
nav_order: 1
has_children: true
permalink: TheSolution/
---

# The Solution

**Online Toestemming** (Online Consent) is a healthcare consent platform. Patients can log in and decide if healthcare companies may share records. Healthcare companies can register themselves and create dossiers for patients.

## The privacy challenge

Dutch law treats a patient's BSN (Burger Service Nummer — citizen service number) as highly sensitive personal data. It must never be stored or passed between services unnecessarily. The platform solves this with **pseudonymisation**: the BSN is converted into a stable, opaque GUID (a *pseudoniem*) at the earliest possible point, and only that GUID flows between services.

## Architecture

```mermaid
graph TB
    subgraph Frontend
        PW[PatientWebsite<br/>Razor Pages]
    end

    subgraph APIs
        IA[IdentityApi]
        PA[PseudoniemApi]
        DA[DossierApi]
    end

    subgraph Data
        DB[(OnlineToestemmingDb<br/>SQL Server)]
        MS[MigrationService]
    end

    MS -->|runs migrations| DB
    IA --> DB
    PA --> DB
    DA --> DB
    PW --> DB

    DA -->|1. get internal token| IA
    DA -->|2. get pseudoniem by BSN| PA
    PW -->|patient login| IA

    style DB fill:#326ce5,stroke:#fff,color:#fff
    style PW fill:#68a063,stroke:#fff,color:#fff
    style IA fill:#f39c12,stroke:#fff,color:#fff
    style PA fill:#e74c3c,stroke:#fff,color:#fff
    style DA fill:#9b59b6,stroke:#fff,color:#fff
    style MS fill:#95a5a6,stroke:#fff,color:#fff
```

## Services at a glance

| Service | Responsibility | External port |
|---------|---------------|---------------|
| **IdentityApi** | Issues JWT tokens for patients, companies, and internal service calls |
| **PseudoniemApi** | Translates BSN → pseudonymous GUID; callable only with an `Internal` JWT |
| **DossierApi** | Manages company registrations and patient consent dossiers |
| **PatientWebsite** | Patient-facing Razor Pages UI; the only publicly reachable service |
| **MigrationService** | One-shot EF Core migration runner; exits after completion |

## What you will explore

1. Start the whole system with one Docker Compose command
2. Walk through each service using the **Bruno** API collection
3. Understand how JWT tokens, pseudonymisation, and service-to-service authentication work together

Ready? [Lets give it a go](LetsGiveItAGo) .