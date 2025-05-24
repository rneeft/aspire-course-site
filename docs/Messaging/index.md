---
title: Messaging
layout: default
nav_order: 4
---

# The New System

Now that we have separated the Identity Provider responsibility from the system, we can update our earlier design to reflect these changes.

Below is an updated system diagram showing how data and authentication flow between components:

```mermaid
graph TD;
    accTitle: System as Identity Provider and Data Flow
    accDescr: The system acts as an identity provider. Health insurers upload data files, which the system processes and stores in the database. Authenticated healthcare providers can query the database for their patients.
    HI[Health Insurers] -- Upload Data Files --> API(API)
    API -- Process Data Files --> DB[(Insurance)]
    API -- Searching --> DB[(Insurance data)]
    IDP -- Registers --> HCP[Healthcare Providers]
    API -- Authenticate --> HI[Health Insurers]
    IDP -- Authenticate --> HCP[Healthcare Providers]
    HCP -- Query for Patients --> API
    IDP(IdentityProvider) -- userdata --> DBIDP[(User data)]
```

{: .highlight }
> API key verification remains a responsibility of the API.

## Messaging

The next step is to introduce messaging into our API. The data files that health insurers upload could be hundreds of megabytes in size. Using messaging will help us process these large files more efficiently and decouple the upload from the processing logic.

There are many frameworks and messaging infrastructure available for handling messaging, such as [RabbitMQ](https://www.rabbitmq.com/), [MassTransit](https://masstransit.io/),and [NServiceBus](https://particular.net/nservicebus). For this course, we will be using NServiceBus, however feel free to try out others. 