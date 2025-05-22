---
title: NServiceBus
parent: Messaging
layout: default
nav_order: 4.1
---

# NServiceBus
NServiceBus is a messaging framework for building distributed systems and microservices on the .NET platform. It's developed by Particular Software and is designed to help developers build scalable, reliable, and fault-tolerant applications using asynchronous messaging and the publish/subscribe pattern.

## Key Concepts

- **Endpoint:** An endpoint is a logical service that sends or receives messages. Each endpoint typically has its own queue.
- **Queue:** A queue is where messages are stored until they are processed by an endpoint.
- **Publisher/Subscriber:** In a publish/subscribe model, publishers send events, and subscribers receive them. NServiceBus manages the routing of these events.
- **Routing:** NServiceBus handles the routing of commands, events, and replies between endpoints based on configuration.

## Common Topologies

NServiceBus supports different topologies depending on the underlying transport (e.g., RabbitMQ, Azure Service Bus):

- **Single Queue Topology:** All messages go through a single queue. Simple but not scalable for large systems.
- **Endpoint-per-Queue Topology:** Each endpoint has its own queue. This is the most common and scalable approach.
- **Publish/Subscribe Topology:** Events are published to a topic or exchange, and multiple subscribers can receive the same event.

## Example

In a typical NServiceBus setup for a microservices architecture:

- Each microservice (endpoint) has its own queue.
- Services communicate by sending commands (to a specific endpoint) or publishing events (to multiple subscribers).
- NServiceBus manages message delivery, retries, and error handling automatically.

---

NServiceBus abstracts much of the complexity of messaging infrastructure, allowing you to focus on business logic while ensuring reliable message delivery and processing.
