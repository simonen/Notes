
#### Stateless Services

A **stateless** service is a service **that does not store data locally** between requests. Each request is independent, and the service doesn't rely on any saved state to operate correctly.

Characteristics
- Can be scaled easily 
- Requests can be routed to any instance
- Failures are easy to recover from - just restart the container.
- No persistent storage required inside the container

Examples
- Web servers serving static files.
- API servers that store data in an external database
- Load balancers.

#### Stateful Services

A **stateful service maintains data or state between requests**. The service depends on that state to function correctly.

Characteristics 
- Scaling is tricky - new replicas need access to the **existing state**.
- Failover is harder - if a node dies, its state must be preserved or recovered.
- Typically requires **persistent storage**.

Examples
- Databases
- Message queues (RabbitMQ, Kafka)
- Any service that stores session data or logs locally.

#### Linux Primitives

- `Namespaces`
- `Cgroups

