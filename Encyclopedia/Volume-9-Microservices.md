# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
# VOLUME 9: MICROSERVICES

> **Audience:** .NET Developer (~3 years). You've built REST APIs. Now we go deep into distributed architecture, messaging, resilience, and patterns used in real production systems at scale.

---

## TABLE OF CONTENTS

1. [Why Microservices?](#why-microservices)
2. [Service Communication — Synchronous (REST/gRPC)](#synchronous-communication)
3. [Service Communication — Asynchronous (Messaging)](#asynchronous-communication)
4. [RabbitMQ In Depth](#rabbitmq)
5. [Apache Kafka In Depth](#apache-kafka)
6. [NATS](#nats)
7. [API Gateway](#api-gateway)
8. [CQRS](#cqrs)
9. [Event Sourcing](#event-sourcing)
10. [Saga Pattern](#saga-pattern)
11. [Outbox Pattern](#outbox-pattern)
12. [Resilience with Polly](#resilience-with-polly)
13. [Distributed Tracing](#distributed-tracing)
14. [Service Mesh](#service-mesh)
15. [Interview Q&A — Master List](#interview-qa)

---

## 1. WHY MICROSERVICES? {#why-microservices}

### 1.1 Definition
Microservices is an architectural style where an application is decomposed into **small, independently deployable services**, each owning its own data, business capability, and deployment lifecycle.

### 1.2 Historical Reason
In the early 2010s, Netflix, Amazon, and Uber hit the walls of monolithic architecture — a single deployable unit where any change required redeploying everything, any outage brought down everything, and any team touching the same codebase created merge conflicts and coordination overhead. Martin Fowler and James Lewis formally described Microservices in 2014, but the pattern was already in production at these giants.

### 1.3 Start with a Monolith — Really

**The Monolith First Rule** (Martin Fowler's advice, 2015):

> "Don't start a new project with microservices, even if you're sure your application will be big enough to make it worthwhile."

**Why?**
- You don't know your domain boundaries yet. Wrong service cuts create distributed monoliths.
- Microservices demand mature DevOps (CI/CD per service, container orchestration, distributed tracing).
- A startup burning runway debugging Kafka consumer lag is a startup that dies.

**When to migrate:**
| Signal | Meaning |
|--------|---------|
| Deploy of one feature breaks another | Coupling is too high |
| One team's release blocks another | Organizational bottleneck |
| A single module needs different scaling than others | Scale mismatch |
| Independent security/compliance boundary needed | Regulatory isolation |
| 5+ developers blocked on the same codebase | Conway's Law kicking in |

### 1.4 Conway's Law

> "Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations." — Mel Conway, 1967

**Production Reality:** If you have a "Payments team" and an "Orders team," you will naturally end up with a Payments Service and an Orders Service. Don't fight this — design your services to match your org chart, or change your org chart to match your desired architecture (Inverse Conway Maneuver).

**Amazon's "Two-Pizza Team" rule:** Each microservice team should be small enough to be fed by two pizzas (~6–8 people). If a team is larger, the service is probably too big.

### 1.5 Microservices Trade-offs

| Benefit | Cost |
|---------|------|
| Independent deployment | Complex CI/CD pipeline |
| Independent scaling | Network latency everywhere |
| Technology heterogeneity | Distributed debugging is painful |
| Fault isolation | Partial failures, eventual consistency |
| Small, focused teams | Service discovery, load balancing needed |
| Clear domain ownership | Data consistency across services is hard |

**The Distributed Systems Tax:** Every microservice boundary you add introduces: serialization cost, network call latency, partial failure possibility, eventual consistency challenges, and increased operational complexity.

### 1.6 Execution Flow — Monolith vs Microservices

```
MONOLITH
User Request → Single Process → [OrderModule] → [InventoryModule] → [PaymentModule]
                                     ↓ In-process calls (nanoseconds)
                                 Single Database
                                 Single Deploy

MICROSERVICES
User Request → API Gateway → Order Service  ──HTTP/gRPC──→ Inventory Service
                                  │                              │
                                  └──── Kafka Event ──────────→ Payment Service
                               Order DB                    Inventory DB    Payment DB
                             (independent)               (independent)  (independent)
```

---

## 2. SERVICE COMMUNICATION — SYNCHRONOUS {#synchronous-communication}

### 2.1 Definition
Synchronous communication means the caller **waits for a response** before continuing. The caller is blocked during the call duration.

### 2.2 When to Use Synchronous Communication
- **Query operations:** "What is the current stock price?" — you need the answer right now.
- **Real-time user-facing requests:** User registration, login, product search.
- **When the response is needed to proceed:** Payment authorization before confirming order.

### 2.3 REST vs gRPC

| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1 | HTTP/2 |
| Payload | JSON (text, verbose) | Protocol Buffers (binary, compact) |
| Schema | OpenAPI (optional) | `.proto` file (enforced) |
| Performance | Baseline | ~5–10x faster serialization |
| Streaming | Limited (SSE, chunked) | Native bi-directional streaming |
| Browser support | Universal | Requires gRPC-Web proxy |
| Code generation | Optional (Swagger codegen) | Mandatory (protoc) |
| Human-readable | Yes | No (binary) |
| Use case | Public APIs, browser clients | Internal service-to-service |

**gRPC in .NET 8:**

```csharp
// orders.proto
syntax = "proto3";
option csharp_namespace = "OrderService.Grpc";

service OrderGrpc {
    rpc GetOrder (GetOrderRequest) returns (OrderResponse);
    rpc StreamOrders (StreamOrdersRequest) returns (stream OrderResponse); // server streaming
}

message GetOrderRequest { string orderId = 1; }
message OrderResponse {
    string orderId = 1;
    string status = 2;
    double totalAmount = 3;
}
```

```csharp
// Server Implementation
public class OrderGrpcService : OrderGrpc.OrderGrpcBase
{
    private readonly IOrderRepository _repo;

    public OrderGrpcService(IOrderRepository repo) => _repo = repo;

    public override async Task<OrderResponse> GetOrder(
        GetOrderRequest request, ServerCallContext context)
    {
        var order = await _repo.GetByIdAsync(request.OrderId);
        if (order is null)
            throw new RpcException(new Status(StatusCode.NotFound, $"Order {request.OrderId} not found"));

        return new OrderResponse
        {
            OrderId = order.Id.ToString(),
            Status = order.Status.ToString(),
            TotalAmount = (double)order.TotalAmount
        };
    }

    // Server-side streaming example
    public override async Task StreamOrders(
        StreamOrdersRequest request,
        IServerStreamWriter<OrderResponse> responseStream,
        ServerCallContext context)
    {
        var orders = _repo.GetOrdersForCustomer(request.CustomerId);
        await foreach (var order in orders)
        {
            if (context.CancellationToken.IsCancellationRequested) break;
            await responseStream.WriteAsync(new OrderResponse
            {
                OrderId = order.Id.ToString(),
                Status = order.Status.ToString()
            });
        }
    }
}

// Program.cs
builder.Services.AddGrpc();
app.MapGrpcService<OrderGrpcService>();
```

```csharp
// Client consumption
builder.Services.AddGrpcClient<OrderGrpc.OrderGrpcClient>(o =>
{
    o.Address = new Uri("https://order-service:5001");
});

// In a controller/handler
var response = await _grpcClient.GetOrderAsync(new GetOrderRequest { OrderId = orderId });
```

### 2.4 Service Discovery

**Problem:** In a dynamic microservices environment, service instances start/stop, scale up/down. You can't hardcode IP addresses.

**Client-side Discovery (Consul, Eureka):**
```
Service A → Query Service Registry → Gets list of Service B instances → Load balance itself → Call Service B
```

**Server-side Discovery (Kubernetes, AWS ALB):**
```
Service A → Call DNS name "order-service" → K8s kube-proxy resolves to a pod IP → Call lands on a pod
```

**In Kubernetes (.NET):** Just use the Service DNS name. K8s handles discovery transparently.
```csharp
// In appsettings.json
"Services": {
  "OrderService": "http://order-service:8080",  // K8s DNS
  "InventoryService": "http://inventory-service:8080"
}
```

### 2.5 Load Balancing in Kubernetes vs Client-Side

**K8s Service:** Round-robin by default, kube-proxy does it at the OS networking level (iptables).

**Client-side with gRPC:** Because gRPC uses persistent HTTP/2 connections, K8s service-level load balancing doesn't work well (all traffic goes to one pod). Use client-side load balancing or a service mesh.

```csharp
// gRPC client-side LB in .NET
builder.Services.AddGrpcClient<OrderGrpc.OrderGrpcClient>(o =>
{
    o.Address = new Uri("dns:///order-service:5001");
})
.ConfigureChannel(o =>
{
    o.ServiceConfig = new ServiceConfig
    {
        LoadBalancingConfigs = { new RoundRobinConfig() }
    };
});
```

---

## 3. SERVICE COMMUNICATION — ASYNCHRONOUS {#asynchronous-communication}

### 3.1 Definition
Asynchronous communication means the sender **does not wait** for a response. It publishes a message/event and continues. The receiver processes independently.

### 3.2 When to Use Async
- **Commands with side effects:** "Place this order" → triggers inventory reservation, email notification, etc.
- **Events:** "Order was placed" → multiple consumers react independently.
- **Decoupling for resilience:** If the Notification Service is down, we still place orders (message waits in queue).
- **High-throughput pipelines:** Log ingestion, telemetry, analytics.

### 3.3 At-Least-Once vs Exactly-Once Delivery

| Guarantee | Meaning | Who Achieves It |
|-----------|---------|-----------------|
| At-most-once | Message may be lost, never duplicated | Fire-and-forget UDP-style |
| At-least-once | Message is delivered, may be duplicated | RabbitMQ with manual ack, Kafka |
| Exactly-once | Delivered exactly once, no duplicates | Kafka transactions, or idempotent consumers |

**The Practical Answer:** Design your consumers to be **idempotent** (processing the same message twice has the same effect as processing it once). Then at-least-once is equivalent to exactly-once. This is far simpler than implementing true exactly-once semantics.

### 3.4 Message Ordering Guarantees

- **RabbitMQ:** Single queue = FIFO. Multiple consumers = no ordering guarantee.
- **Kafka:** Ordering guaranteed **within a partition**. Use the same partition key (e.g., `orderId`) to ensure order events for the same order are processed in sequence.

---

## 4. RABBITMQ IN DEPTH {#rabbitmq}

### 4.1 Definition
RabbitMQ is an open-source message broker implementing the **AMQP** (Advanced Message Queuing Protocol). It routes messages from producers to consumers through a system of Exchanges, Queues, and Bindings.

### 4.2 Historical Reason
Built in 2007 by Rabbit Technologies (later acquired by VMware/Pivotal/Broadcom). It was designed to decouple services in enterprise applications, before Kafka existed.

### 4.3 Core Architecture

```
Producer → Exchange → [Binding with routing key] → Queue → Consumer
                ↓
         (Exchange decides where message goes based on type + routing key)
```

**Exchange Types:**

| Type | Routing Logic | Use Case |
|------|--------------|---------|
| **Direct** | Route by exact routing key match | Command routing: key=`order.created` goes to orders queue |
| **Fanout** | Broadcast to ALL bound queues | Notification: all subscribers get event |
| **Topic** | Route by routing key pattern (`*`, `#`) | `order.*` = any single word after order |
| **Headers** | Route by message header attributes | Complex routing without key conventions |

**Topic Exchange Pattern Matching:**
- `*` matches exactly one word: `order.*` matches `order.created` but NOT `order.created.v2`
- `#` matches zero or more words: `order.#` matches `order.created`, `order.created.v2`, `order`

### 4.4 Dead Letter Queue (DLQ)

When a message fails (rejected, TTL expired, queue full), it goes to the DLQ. This prevents message loss while allowing your main consumer to recover.

```
Queue (main) → Consumer processes → NACK (failure) → Dead Letter Exchange → DLQ → Inspect/Retry
```

### 4.5 Message Acknowledgment

**Auto-ack:** Message removed from queue the moment it's delivered. If consumer crashes while processing → **message is lost**.

**Manual-ack:** Consumer explicitly acks after successful processing. If consumer crashes → message is **requeued** automatically.

### 4.6 MassTransit with RabbitMQ in .NET

```csharp
// NuGet: MassTransit.RabbitMQ

// 1. Define Messages (contracts)
public record OrderPlacedEvent(Guid OrderId, Guid CustomerId, decimal TotalAmount);
public record ReserveInventoryCommand(Guid OrderId, List<OrderItem> Items);

// 2. Configure MassTransit
builder.Services.AddMassTransit(x =>
{
    x.AddConsumer<OrderPlacedConsumer>();
    x.AddConsumer<InventoryReservationConsumer>();

    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.Host("rabbitmq://localhost", h =>
        {
            h.Username("guest");
            h.Password("guest");
        });

        // Configure Dead Letter Queue
        cfg.ReceiveEndpoint("order-placed-queue", e =>
        {
            e.DeadLetterExchange = "order-dlx";
            e.UseMessageRetry(r => r.Exponential(5,
                TimeSpan.FromSeconds(1),
                TimeSpan.FromSeconds(30),
                TimeSpan.FromSeconds(5)));
            e.ConfigureConsumer<OrderPlacedConsumer>(context);
        });

        cfg.ReceiveEndpoint("inventory-reservation-queue", e =>
        {
            e.ConfigureConsumer<InventoryReservationConsumer>(context);
        });
    });
});

// 3. Publish an Event
public class OrderService
{
    private readonly IPublishEndpoint _publishEndpoint;

    public OrderService(IPublishEndpoint publishEndpoint)
        => _publishEndpoint = publishEndpoint;

    public async Task PlaceOrderAsync(PlaceOrderRequest request)
    {
        var order = Order.Create(request);
        await _orderRepository.SaveAsync(order);

        // Publish event — MassTransit handles exchange/queue naming
        await _publishEndpoint.Publish(new OrderPlacedEvent(
            order.Id,
            order.CustomerId,
            order.TotalAmount
        ));
    }
}

// 4. Consumer Implementation
public class OrderPlacedConsumer : IConsumer<OrderPlacedEvent>
{
    private readonly ILogger<OrderPlacedConsumer> _logger;
    private readonly INotificationService _notifications;

    public OrderPlacedConsumer(ILogger<OrderPlacedConsumer> logger,
        INotificationService notifications)
    {
        _logger = logger;
        _notifications = notifications;
    }

    public async Task Consume(ConsumeContext<OrderPlacedEvent> context)
    {
        var evt = context.Message;
        _logger.LogInformation("Processing order {OrderId} for customer {CustomerId}",
            evt.OrderId, evt.CustomerId);

        await _notifications.SendOrderConfirmationAsync(evt.CustomerId, evt.OrderId);

        // Implicit ack on successful completion
        // If exception is thrown → MassTransit retries based on policy
    }
}
```

### 4.7 Execution Flow — Order Placed via RabbitMQ

```
[Order API] 
    → PlaceOrderAsync()
    → Save order to DB (status: Pending)
    → Publish "OrderPlacedEvent" to RabbitMQ
    
[RabbitMQ]
    → order.events Exchange (Topic)
    → Binding: order.# → inventory-reservation-queue
    → Binding: order.# → notification-queue
    → Binding: order.# → analytics-queue

[Inventory Service Consumer]
    → Receives OrderPlacedEvent
    → Reserves stock
    → Publishes InventoryReservedEvent

[Notification Service Consumer]
    → Receives OrderPlacedEvent
    → Sends confirmation email
```

### 4.8 Interview Q&A — RabbitMQ

**Q: What happens if a consumer crashes mid-processing?**
> With manual ack, the message remains unacknowledged. RabbitMQ requeues it to another consumer after the connection closes. With auto-ack, the message is lost.

**Q: RabbitMQ vs Kafka — choose one.**
> RabbitMQ for smart routing, task queues, work distribution (push-based). Kafka for event streaming, replay, audit trail, high-throughput pipelines (pull-based, log-based).

---

## 5. APACHE KAFKA {#apache-kafka}

### 5.1 Definition
Apache Kafka is a **distributed event streaming platform** built around an append-only, partitioned, replicated commit log. Consumers pull messages; messages are retained regardless of consumption.

### 5.2 Historical Reason
Created at LinkedIn in 2011 to handle real-time activity feeds. Open-sourced and donated to Apache. LinkedIn was processing 1 trillion messages per day by 2015.

### 5.3 Core Concepts

```
Producer → Topic [Partition 0: M1, M2, M3 | Partition 1: M4, M5 | Partition 2: M6] → Consumer Group A
                                                                                    → Consumer Group B
```

- **Topic:** Named stream of records (like a database table for events)
- **Partition:** Ordered, immutable sequence of records within a topic. Parallelism unit.
- **Consumer Group:** Multiple consumers sharing work. Each partition assigned to exactly one consumer in a group.
- **Offset:** Position of a message in a partition. Consumer tracks its own offset.
- **Broker:** Kafka server. Multiple brokers = cluster.
- **Replication:** Each partition replicated across N brokers. Leader handles writes; followers sync.

### 5.4 How Kafka Guarantees Ordering

**Ordering is guaranteed ONLY within a partition.**

```
Topic: order-events
  Partition 0 (key: orderId=100): OrderCreated → OrderPaid → OrderShipped → OrderDelivered (ORDERED)
  Partition 1 (key: orderId=200): OrderCreated → OrderPaid (ORDERED, but interleaved with partition 0)
```

**Partition Assignment:** Use a meaningful partition key:
```csharp
// Always route order-100 events to same partition
var message = new Message<string, string>
{
    Key = order.Id.ToString(),  // Same key → same partition → ordered
    Value = JsonSerializer.Serialize(orderEvent)
};
await producer.ProduceAsync("order-events", message);
```

### 5.5 Offset Management — At-Least-Once Delivery

```
Consumer reads message at offset 42
Consumer processes message (crashes before committing offset)
Consumer restarts → re-reads from last committed offset (e.g., 40)
→ Re-processes offsets 40, 41, 42 (at-least-once)

Solution: Idempotent processing — check if orderId already processed
```

### 5.6 Kafka in .NET with Confluent.Kafka

```csharp
// Producer
public class KafkaOrderEventPublisher
{
    private readonly IProducer<string, string> _producer;

    public KafkaOrderEventPublisher(IConfiguration config)
    {
        var producerConfig = new ProducerConfig
        {
            BootstrapServers = config["Kafka:BootstrapServers"],
            Acks = Acks.All,                    // Wait for all replicas to ack
            EnableIdempotence = true,           // Exactly-once at producer level
            MessageSendMaxRetries = 3,
            RetryBackoffMs = 100
        };
        _producer = new ProducerBuilder<string, string>(producerConfig).Build();
    }

    public async Task PublishOrderEventAsync<T>(string orderId, T eventData)
    {
        var message = new Message<string, string>
        {
            Key = orderId,                      // Partition key for ordering
            Value = JsonSerializer.Serialize(eventData),
            Headers = new Headers
            {
                { "eventType", Encoding.UTF8.GetBytes(typeof(T).Name) },
                { "correlationId", Encoding.UTF8.GetBytes(Activity.Current?.TraceId.ToString() ?? Guid.NewGuid().ToString()) }
            }
        };

        var result = await _producer.ProduceAsync("order-events", message);
        // result.Offset tells you where the message landed
    }
}

// Consumer (hosted background service)
public class OrderEventConsumer : BackgroundService
{
    private readonly IConsumer<string, string> _consumer;
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<OrderEventConsumer> _logger;

    public OrderEventConsumer(IConfiguration config, IServiceScopeFactory scopeFactory,
        ILogger<OrderEventConsumer> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;

        var consumerConfig = new ConsumerConfig
        {
            BootstrapServers = config["Kafka:BootstrapServers"],
            GroupId = "inventory-service-group",
            AutoOffsetReset = AutoOffsetReset.Earliest,
            EnableAutoCommit = false            // Manual commit for reliability
        };

        _consumer = new ConsumerBuilder<string, string>(consumerConfig).Build();
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _consumer.Subscribe("order-events");

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var consumeResult = _consumer.Consume(stoppingToken);
                if (consumeResult is null) continue;

                await ProcessMessageAsync(consumeResult, stoppingToken);

                // Manual commit AFTER successful processing
                _consumer.Commit(consumeResult);
            }
            catch (ConsumeException ex)
            {
                _logger.LogError(ex, "Kafka consume error");
            }
            catch (OperationCanceledException)
            {
                break;
            }
        }

        _consumer.Close();
    }

    private async Task ProcessMessageAsync(
        ConsumeResult<string, string> result, CancellationToken ct)
    {
        using var scope = _scopeFactory.CreateScope();
        var handler = scope.ServiceProvider.GetRequiredService<IOrderEventHandler>();

        _logger.LogInformation("Processing offset {Offset} partition {Partition}",
            result.Offset, result.Partition);

        // Idempotency check
        var processed = await handler.IsAlreadyProcessedAsync(result.Offset.Value, result.Partition);
        if (processed)
        {
            _logger.LogWarning("Duplicate message at offset {Offset}, skipping", result.Offset);
            return;
        }

        await handler.HandleAsync(result.Message.Value, ct);
        await handler.MarkProcessedAsync(result.Offset.Value, result.Partition);
    }
}
```

### 5.7 Kafka vs RabbitMQ — Decision Guide

| Dimension | RabbitMQ | Kafka |
|-----------|---------|-------|
| Message model | Queued (consumed = gone) | Log (retained, replayable) |
| Throughput | ~50k msg/s | ~1M+ msg/s |
| Message replay | No (once consumed, gone) | Yes (replay from any offset) |
| Routing | Rich (exchanges, topic patterns) | Simple (topic + partition) |
| Consumer model | Push | Pull |
| Message TTL | Configurable | Configurable retention (time/size) |
| Ordering | Per-queue | Per-partition |
| Use cases | Task queues, RPC, smart routing | Event streaming, audit logs, analytics |
| Operational complexity | Lower | Higher |

**Choose Kafka when:**
- You need to replay events (audit, ML training, replaying for new service)
- High throughput (telemetry, logs, clickstream)
- Multiple consumer groups need to independently process the same events

**Choose RabbitMQ when:**
- Task distribution (workers picking up jobs)
- Complex routing (topic exchange patterns)
- Lower operational complexity acceptable

---

## 6. NATS {#nats}

### 6.1 Definition
NATS is an ultra-lightweight, high-performance messaging system written in Go. It focuses on simplicity and speed, with sub-millisecond latency.

### 6.2 When to Use NATS vs Kafka

| Scenario | Use |
|---------|-----|
| Ultra-low latency (<1ms) request/reply | NATS Core |
| IoT telemetry, edge computing | NATS Core |
| Persistent streaming with replay | NATS JetStream |
| Complex routing, enterprise patterns | RabbitMQ |
| High-throughput event log | Kafka |
| Simple cloud-native messaging | NATS |

**NATS Core** is at-most-once (no persistence). **NATS JetStream** adds persistence and at-least-once delivery.

```csharp
// NATS in .NET (NATS.Net)
var connection = new NatsConnection(new NatsOpts
{
    Url = "nats://localhost:4222"
});

// Publisher
await connection.PublishAsync("orders.created", new OrderCreatedEvent(orderId));

// Subscriber
await foreach (var msg in connection.SubscribeAsync<OrderCreatedEvent>("orders.*"))
{
    Console.WriteLine($"Received: {msg.Data?.OrderId}");
}
```

---

## 7. API GATEWAY {#api-gateway}

### 7.1 Definition
An API Gateway is a **single entry point** for all client requests, acting as a reverse proxy that routes requests to appropriate microservices while handling cross-cutting concerns.

### 7.2 What It Does

```
Client → API Gateway → [Auth] → [Rate Limit] → [Route] → [Load Balance] → Service A
                                                                          → Service B
```

**Responsibilities:**
- **Routing:** `/api/orders` → Order Service, `/api/products` → Product Service
- **Authentication:** Validate JWT before forwarding
- **Rate Limiting:** 100 req/min per user
- **SSL Termination:** HTTPS at gateway, HTTP internally
- **Request Aggregation:** Combine results from multiple services
- **Caching:** Cache read responses
- **Logging/Tracing:** Attach correlation IDs

### 7.3 Ocelot for .NET

```csharp
// NuGet: Ocelot

// Program.cs
builder.Configuration.AddJsonFile("ocelot.json");
builder.Services.AddOcelot();
// ...
app.UseOcelot().Wait();
```

```json
// ocelot.json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/orders/{orderId}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "order-service", "Port": 8080 }
      ],
      "UpstreamPathTemplate": "/orders/{orderId}",
      "UpstreamHttpMethod": ["GET"],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer"
      },
      "RateLimitOptions": {
        "ClientWhitelist": ["admin"],
        "EnableRateLimiting": true,
        "Period": "1m",
        "PeriodTimespan": 1,
        "Limit": 100
      },
      "FileCacheOptions": {
        "TtlSeconds": 15,
        "Region": "orders"
      }
    },
    {
      "DownstreamPathTemplate": "/api/products/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "product-service", "Port": 8080 },
        { "Host": "product-service-2", "Port": 8080 }
      ],
      "UpstreamPathTemplate": "/products/{everything}",
      "LoadBalancerOptions": { "Type": "RoundRobin" }
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://api.myapp.com"
  }
}
```

### 7.4 Kong vs AWS API Gateway

| Feature | Ocelot | Kong | AWS API Gateway |
|---------|--------|------|----------------|
| Hosting | Self-hosted .NET | Self-hosted | Managed |
| Config | JSON/code | Plugin-based | Console/Terraform |
| Performance | Good | Excellent (Nginx-based) | Variable |
| Plugins | Limited | 100s of plugins | Lambda, WAF, etc. |
| Cost | Free | Free (OSS) / Commercial | Pay-per-request |
| Use case | .NET microservices | Polyglot microservices | AWS-native apps |

---

## 8. CQRS (COMMAND QUERY RESPONSIBILITY SEGREGATION) {#cqrs}

### 8.1 Definition
CQRS separates the **read model** (queries) from the **write model** (commands). Commands change state; queries return state. They can use different models, different DBs, different optimizations.

### 8.2 Historical Reason
Coined by Greg Young (2010), evolved from Bertrand Meyer's Command-Query Separation (CQS) principle. Addresses the impedance mismatch between write operations (transactional, normalized) and read operations (denormalized, reporting).

### 8.3 Problem It Solves

**Traditional:** One model for reads AND writes.
- Write: Normalized schema (3NF) with FK constraints — slow to query across joins
- Read: Same normalized model — you JOIN 6 tables to render a dashboard
- Optimization conflict: Index for fast reads hurts write performance

**CQRS:** Separate models optimized for each concern.

### 8.4 Simple CQRS vs Full CQRS

```
SIMPLE CQRS (same DB, different code models):
Commands → Write Service → CommandHandler → Same DB
Queries  → Read Service  → QueryHandler  → Same DB (but denormalized views/projections)

FULL CQRS (separate DBs):
Commands → Write Service → CommandHandler → Write DB (SQL, normalized)
                                              ↓ publish event
Queries  → Read Service  → QueryHandler  → Read DB (Redis/MongoDB/ElasticSearch, denormalized)
                                              ↑ updated by event consumer
```

### 8.5 MediatR as CQRS Implementation in .NET

```csharp
// NuGet: MediatR

// ══════════════════════════════════════
// COMMANDS (write side — changes state)
// ══════════════════════════════════════

// Command definition (returns unit or a result)
public record PlaceOrderCommand(
    Guid CustomerId,
    List<OrderItem> Items,
    string ShippingAddress
) : IRequest<PlaceOrderResult>;

public record PlaceOrderResult(Guid OrderId, string Status);

// Command Handler
public class PlaceOrderCommandHandler : IRequestHandler<PlaceOrderCommand, PlaceOrderResult>
{
    private readonly IOrderRepository _orderRepo;
    private readonly IPublishEndpoint _bus;
    private readonly IInventoryService _inventory;

    public PlaceOrderCommandHandler(IOrderRepository orderRepo,
        IPublishEndpoint bus, IInventoryService inventory)
    {
        _orderRepo = orderRepo;
        _bus = bus;
        _inventory = inventory;
    }

    public async Task<PlaceOrderResult> Handle(
        PlaceOrderCommand command, CancellationToken cancellationToken)
    {
        // Validate
        var available = await _inventory.CheckAvailabilityAsync(command.Items, cancellationToken);
        if (!available)
            throw new BusinessException("One or more items are out of stock.");

        // Create aggregate
        var order = Order.Create(command.CustomerId, command.Items, command.ShippingAddress);

        // Persist
        await _orderRepo.AddAsync(order, cancellationToken);

        // Publish domain event
        await _bus.Publish(new OrderPlacedEvent(order.Id, order.CustomerId, order.TotalAmount), cancellationToken);

        return new PlaceOrderResult(order.Id, order.Status.ToString());
    }
}

// ══════════════════════════════════════
// QUERIES (read side — returns data)
// ══════════════════════════════════════

// Query definition
public record GetOrderSummaryQuery(Guid OrderId) : IRequest<OrderSummaryDto>;

public record OrderSummaryDto(
    Guid OrderId,
    string Status,
    string CustomerName,
    decimal TotalAmount,
    List<string> ItemNames,
    DateTime CreatedAt
);

// Query Handler — hits read DB or cache, returns denormalized DTO
public class GetOrderSummaryQueryHandler : IRequestHandler<GetOrderSummaryQuery, OrderSummaryDto>
{
    private readonly IReadDbContext _readDb;       // Read-optimized DB
    private readonly IDistributedCache _cache;

    public GetOrderSummaryQueryHandler(IReadDbContext readDb, IDistributedCache cache)
    {
        _readDb = readDb;
        _cache = cache;
    }

    public async Task<OrderSummaryDto> Handle(
        GetOrderSummaryQuery query, CancellationToken cancellationToken)
    {
        // Check cache first
        var cacheKey = $"order-summary:{query.OrderId}";
        var cached = await _cache.GetStringAsync(cacheKey, cancellationToken);
        if (cached is not null)
            return JsonSerializer.Deserialize<OrderSummaryDto>(cached)!;

        // Single flat query — no joins needed (denormalized read model)
        var dto = await _readDb.OrderSummaries
            .AsNoTracking()
            .Where(o => o.OrderId == query.OrderId)
            .Select(o => new OrderSummaryDto(
                o.OrderId, o.Status, o.CustomerName,
                o.TotalAmount, o.ItemNames, o.CreatedAt))
            .FirstOrDefaultAsync(cancellationToken)
            ?? throw new NotFoundException($"Order {query.OrderId} not found");

        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(dto),
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5) },
            cancellationToken);

        return dto;
    }
}

// ══════════════════════════════════════
// CONTROLLER (thin — just dispatches)
// ══════════════════════════════════════
[ApiController]
[Route("api/orders")]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrdersController(IMediator mediator) => _mediator = mediator;

    [HttpPost]
    public async Task<IActionResult> PlaceOrder(PlaceOrderRequest request)
    {
        var command = new PlaceOrderCommand(request.CustomerId, request.Items, request.ShippingAddress);
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetOrder), new { id = result.OrderId }, result);
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(Guid id)
    {
        var result = await _mediator.Send(new GetOrderSummaryQuery(id));
        return Ok(result);
    }
}

// ══════════════════════════════════════
// PIPELINE BEHAVIORS (cross-cutting)
// ══════════════════════════════════════
// Logging, validation, retry — applied to all commands/queries
public class LoggingBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse> where TRequest : notnull
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
        => _logger = logger;

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var requestName = typeof(TRequest).Name;
        _logger.LogInformation("Handling {RequestName}: {@Request}", requestName, request);
        var stopwatch = Stopwatch.StartNew();

        var response = await next();

        stopwatch.Stop();
        _logger.LogInformation("Handled {RequestName} in {ElapsedMs}ms", requestName, stopwatch.ElapsedMilliseconds);
        return response;
    }
}

// Register everything
builder.Services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssembly(typeof(Program).Assembly);
    cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
    cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
});
```

### 8.6 When CQRS is Overkill
- Simple CRUD with no read/write performance mismatch
- Small team with no need for separate read/write optimization
- Admin panels, internal tools, simple APIs
- **Rule of thumb:** Start without CQRS. Add it when you have a demonstrable read/write performance problem.

### 8.7 Read Model Synchronization

In Full CQRS, a consumer listens for events and updates the read model:

```csharp
// Event consumer that updates read model
public class OrderPlacedReadModelUpdater : IConsumer<OrderPlacedEvent>
{
    private readonly IReadDbContext _readDb;

    public async Task Consume(ConsumeContext<OrderPlacedEvent> context)
    {
        var evt = context.Message;
        // Upsert into denormalized read model table
        var summary = new OrderSummaryReadModel
        {
            OrderId = evt.OrderId,
            Status = "Pending",
            TotalAmount = evt.TotalAmount,
            CreatedAt = evt.OccurredOn
        };
        await _readDb.OrderSummaries.AddAsync(summary);
        await _readDb.SaveChangesAsync();
    }
}
```

---

## 9. EVENT SOURCING {#event-sourcing}

### 9.1 Definition
Event Sourcing stores **all changes to application state as a sequence of events**. The current state is derived by replaying events from the beginning (or from a snapshot). The event log IS the source of truth — not the current state table.

### 9.2 Historical Reason
Originated in Domain-Driven Design (DDD) communities. Greg Young introduced it alongside CQRS. Financial systems (trading, banking) have always used double-entry bookkeeping — a form of event sourcing.

### 9.3 Problem It Solves

**Traditional (state-based):**
```
Account table: { id: 1, balance: 850 }
Question: "What was the balance on March 15th?" → IMPOSSIBLE to answer
```

**Event Sourcing:**
```
Events table:
  AccountOpened    { accountId: 1, initialBalance: 1000, at: Mar-01 }
  MoneyDeposited   { accountId: 1, amount: 500, at: Mar-10 }
  MoneyWithdrawn   { accountId: 1, amount: 650, at: Mar-15 }

Current state = Replay: 1000 + 500 - 650 = 850
March 15 state = Replay up to Mar-15 = 850 (after withdrawal)
```

### 9.4 Internal Working

```
Command → Aggregate.Handle(command) → Aggregate raises DomainEvents → 
Events appended to Event Store → State updated by applying events →
Events published to message bus → Read models updated
```

### 9.5 Aggregates and Domain Events

```csharp
// Domain Event base
public abstract record DomainEvent
{
    public Guid EventId { get; } = Guid.NewGuid();
    public DateTime OccurredOn { get; } = DateTime.UtcNow;
    public abstract string EventType { get; }
}

// Domain Events
public record OrderCreatedEvent(
    Guid OrderId, Guid CustomerId, List<OrderItem> Items, decimal TotalAmount)
    : DomainEvent
{
    public override string EventType => "OrderCreated";
}

public record OrderPaymentConfirmedEvent(Guid OrderId, string TransactionId)
    : DomainEvent
{
    public override string EventType => "OrderPaymentConfirmed";
}

public record OrderShippedEvent(Guid OrderId, string TrackingNumber)
    : DomainEvent
{
    public override string EventType => "OrderShipped";
}

// Aggregate Root — holds state, raises events
public class OrderAggregate
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    public decimal TotalAmount { get; private set; }
    public string? TrackingNumber { get; private set; }
    public int Version { get; private set; }

    private readonly List<DomainEvent> _uncommittedEvents = new();
    public IReadOnlyList<DomainEvent> UncommittedEvents => _uncommittedEvents;

    // Required for event store reconstruction
    private OrderAggregate() { }

    // Factory — creates aggregate via event
    public static OrderAggregate Create(Guid customerId, List<OrderItem> items)
    {
        var aggregate = new OrderAggregate();
        var total = items.Sum(i => i.Price * i.Quantity);

        aggregate.RaiseEvent(new OrderCreatedEvent(Guid.NewGuid(), customerId, items, total));
        return aggregate;
    }

    // Business operations
    public void ConfirmPayment(string transactionId)
    {
        if (Status != OrderStatus.Pending)
            throw new InvalidOperationException($"Cannot confirm payment for order in {Status} status");

        RaiseEvent(new OrderPaymentConfirmedEvent(Id, transactionId));
    }

    public void Ship(string trackingNumber)
    {
        if (Status != OrderStatus.PaymentConfirmed)
            throw new InvalidOperationException("Order must be paid before shipping");

        RaiseEvent(new OrderShippedEvent(Id, trackingNumber));
    }

    // Event Application — updates internal state
    private void Apply(OrderCreatedEvent evt)
    {
        Id = evt.OrderId;
        CustomerId = evt.CustomerId;
        TotalAmount = evt.TotalAmount;
        Status = OrderStatus.Pending;
    }

    private void Apply(OrderPaymentConfirmedEvent evt)
    {
        Status = OrderStatus.PaymentConfirmed;
    }

    private void Apply(OrderShippedEvent evt)
    {
        Status = OrderStatus.Shipped;
        TrackingNumber = evt.TrackingNumber;
    }

    // Infrastructure
    private void RaiseEvent(DomainEvent evt)
    {
        ApplyEvent(evt);
        _uncommittedEvents.Add(evt);
    }

    private void ApplyEvent(DomainEvent evt)
    {
        Version++;
        switch (evt)
        {
            case OrderCreatedEvent e: Apply(e); break;
            case OrderPaymentConfirmedEvent e: Apply(e); break;
            case OrderShippedEvent e: Apply(e); break;
        }
    }

    // Rebuild from event store
    public static OrderAggregate Rehydrate(IEnumerable<DomainEvent> events)
    {
        var aggregate = new OrderAggregate();
        foreach (var evt in events)
            aggregate.ApplyEvent(evt);
        return aggregate;
    }

    public void ClearUncommittedEvents() => _uncommittedEvents.Clear();
}

// Event Store Repository
public class EventSourcedOrderRepository
{
    private readonly IEventStore _eventStore;
    private readonly IPublishEndpoint _bus;

    public EventSourcedOrderRepository(IEventStore eventStore, IPublishEndpoint bus)
    {
        _eventStore = eventStore;
        _bus = bus;
    }

    public async Task SaveAsync(OrderAggregate order, CancellationToken ct = default)
    {
        var streamName = $"order-{order.Id}";
        var events = order.UncommittedEvents.ToList();

        // Append to event store (optimistic concurrency via version)
        await _eventStore.AppendToStreamAsync(streamName, order.Version - events.Count, events, ct);

        // Publish to message bus for read model updates
        foreach (var evt in events)
            await _bus.Publish(evt, ct);

        order.ClearUncommittedEvents();
    }

    public async Task<OrderAggregate?> GetByIdAsync(Guid orderId, CancellationToken ct = default)
    {
        var streamName = $"order-{orderId}";
        var events = await _eventStore.ReadStreamAsync(streamName, ct);

        if (!events.Any()) return null;

        return OrderAggregate.Rehydrate(events);
    }
}
```

### 9.6 Snapshots for Performance

Replaying 10,000 events every time you load an aggregate is slow. Use snapshots:

```csharp
public async Task<OrderAggregate?> GetByIdAsync(Guid orderId, CancellationToken ct)
{
    var streamName = $"order-{orderId}";

    // Check for latest snapshot
    var snapshot = await _snapshotStore.GetLatestAsync(streamName, ct);

    if (snapshot is not null)
    {
        // Load only events AFTER snapshot version
        var events = await _eventStore.ReadStreamFromVersionAsync(
            streamName, snapshot.Version + 1, ct);

        var aggregate = OrderAggregate.FromSnapshot(snapshot);
        aggregate.ReplayEvents(events);
        return aggregate;
    }

    // No snapshot — replay all events
    var allEvents = await _eventStore.ReadStreamAsync(streamName, ct);
    return OrderAggregate.Rehydrate(allEvents);
}

// Create snapshot every N events
if (order.Version % 50 == 0)
    await _snapshotStore.SaveAsync(new OrderSnapshot(order), ct);
```

### 9.7 When to Use Event Sourcing

✅ **Use when:**
- Audit trail is mandatory: financial transactions, healthcare records
- Temporal queries: "What was the state at time T?"
- Event replay: new service needs historical events
- Complex business workflows with compensation

❌ **Do NOT use when:**
- Simple CRUD with no audit requirements
- Small teams without DDD knowledge (high learning curve)
- High-frequency state updates that make event log impractical (e.g., real-time cursor position)

---

## 10. SAGA PATTERN {#saga-pattern}

### 10.1 Definition
A Saga is a sequence of **local transactions**, each publishing events or messages to trigger the next step. If any step fails, compensating transactions are executed to undo previously completed steps.

### 10.2 Why Not Two-Phase Commit (2PC)?
2PC (distributed transaction) requires all participating services to hold locks until the coordinator confirms. In a microservices environment with dozens of services:
- **Lock contention:** Everything grinds to a halt
- **Coordinator becomes SPOF:** If coordinator crashes mid-commit, all participants hang
- **Cross-DB impossible:** 2PC works between same DB vendor (e.g., SQL Server ↔ SQL Server) — not across PostgreSQL + MongoDB + Redis

### 10.3 Choreography-based Saga

Each service publishes events; the next service reacts. No central coordinator.

```
Order Service → publishes OrderPlaced
  → Inventory Service reacts → reserves stock → publishes InventoryReserved
    → Payment Service reacts → charges card → publishes PaymentCharged
      → Order Service reacts → confirms order → publishes OrderConfirmed

FAILURE PATH:
Payment Service → charge fails → publishes PaymentFailed
  → Inventory Service reacts → releases reservation → publishes InventoryReleased
    → Order Service reacts → cancels order → publishes OrderCancelled
```

**Pros:** Simple, no single point of failure, loose coupling.
**Cons:** Hard to understand flow across services, no central visibility.

### 10.4 Orchestration-based Saga

A central **Saga Orchestrator** (state machine) tells each participant what to do.

```
Saga Orchestrator (Order Saga)
  Step 1: Send ReserveInventoryCommand → Inventory Service
    Success: → Step 2
    Failure: → DONE (order cancelled)
  Step 2: Send ChargePaymentCommand → Payment Service
    Success: → Step 3
    Failure: → Compensate: Send ReleaseInventoryCommand → DONE
  Step 3: Send ConfirmOrderCommand → Order Service
    Success: → DONE (saga complete)
```

**Pros:** Centralized logic, easy to debug, explicit state machine.
**Cons:** Orchestrator can become a bottleneck; tighter coupling to orchestrator.

### 10.5 MassTransit Saga State Machine in .NET

```csharp
// Saga State — persisted to DB
public class OrderSagaState : SagaStateMachineInstance
{
    public Guid CorrelationId { get; set; }  // Required by MassTransit
    public string CurrentState { get; set; } = null!;
    public Guid OrderId { get; set; }
    public Guid CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public string? PaymentTransactionId { get; set; }
    public string? InventoryReservationId { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Saga State Machine
public class OrderSagaStateMachine : MassTransitStateMachine<OrderSagaState>
{
    // States
    public State InventoryReserving { get; private set; } = null!;
    public State PaymentProcessing { get; private set; } = null!;
    public State Completed { get; private set; } = null!;
    public State Failed { get; private set; } = null!;

    // Events
    public Event<OrderPlacedEvent> OrderPlaced { get; private set; } = null!;
    public Event<InventoryReservedEvent> InventoryReserved { get; private set; } = null!;
    public Event<InventoryReservationFailedEvent> InventoryReservationFailed { get; private set; } = null!;
    public Event<PaymentChargedEvent> PaymentCharged { get; private set; } = null!;
    public Event<PaymentFailedEvent> PaymentFailed { get; private set; } = null!;

    public OrderSagaStateMachine()
    {
        InstanceState(x => x.CurrentState);

        // Correlate by OrderId
        Event(() => OrderPlaced, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => InventoryReserved, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => InventoryReservationFailed, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => PaymentCharged, x => x.CorrelateById(m => m.Message.OrderId));
        Event(() => PaymentFailed, x => x.CorrelateById(m => m.Message.OrderId));

        // Initial → InventoryReserving
        Initially(
            When(OrderPlaced)
                .Then(ctx =>
                {
                    ctx.Saga.OrderId = ctx.Message.OrderId;
                    ctx.Saga.CustomerId = ctx.Message.CustomerId;
                    ctx.Saga.TotalAmount = ctx.Message.TotalAmount;
                    ctx.Saga.CreatedAt = DateTime.UtcNow;
                })
                .Send(ctx => new ReserveInventoryCommand(ctx.Message.OrderId, ctx.Message.Items))
                .TransitionTo(InventoryReserving)
        );

        // InventoryReserving → PaymentProcessing (or Failed)
        During(InventoryReserving,
            When(InventoryReserved)
                .Then(ctx => ctx.Saga.InventoryReservationId = ctx.Message.ReservationId)
                .Send(ctx => new ChargePaymentCommand(ctx.Message.OrderId, ctx.Saga.TotalAmount, ctx.Saga.CustomerId))
                .TransitionTo(PaymentProcessing),

            When(InventoryReservationFailed)
                .Then(ctx => Console.WriteLine($"Inventory failed for order {ctx.Saga.OrderId}"))
                .Send(ctx => new CancelOrderCommand(ctx.Message.OrderId, "Inventory unavailable"))
                .TransitionTo(Failed)
        );

        // PaymentProcessing → Completed or Compensate
        During(PaymentProcessing,
            When(PaymentCharged)
                .Then(ctx => ctx.Saga.PaymentTransactionId = ctx.Message.TransactionId)
                .Send(ctx => new ConfirmOrderCommand(ctx.Message.OrderId))
                .TransitionTo(Completed),

            When(PaymentFailed)
                // Compensating transaction: release inventory reservation
                .Send(ctx => new ReleaseInventoryCommand(ctx.Saga.OrderId, ctx.Saga.InventoryReservationId!))
                .Send(ctx => new CancelOrderCommand(ctx.Saga.OrderId, "Payment declined"))
                .TransitionTo(Failed)
        );
    }
}

// Registration
builder.Services.AddMassTransit(x =>
{
    x.AddSagaStateMachine<OrderSagaStateMachine, OrderSagaState>()
        .EntityFrameworkRepository(r =>
        {
            r.ConcurrencyMode = ConcurrencyMode.Optimistic;
            r.AddDbContext<OrderSagaDbContext>((provider, optBuilder) =>
            {
                optBuilder.UseSqlServer(config.GetConnectionString("SagaDb"));
            });
        });
    // ... rest of config
});
```

### 10.6 Execution Flow — Order Saga

```
[Order API] PlaceOrder command received
     ↓ Save order (status: Pending) + publish OrderPlacedEvent
[Saga Starts] State: Initial → InventoryReserving
     ↓ Send ReserveInventoryCommand to Inventory Service
[Inventory Service] Checks stock → Success → Publish InventoryReservedEvent
[Saga] State: InventoryReserving → PaymentProcessing
     ↓ Send ChargePaymentCommand to Payment Service
[Payment Service] Charges card → Success → Publish PaymentChargedEvent
[Saga] State: PaymentProcessing → Completed
     ↓ Send ConfirmOrderCommand to Order Service
[Order Service] Updates order status to Confirmed
     ↓ Send OrderConfirmedEmail notification
[DONE] ✅

FAILURE SCENARIO (payment fails):
[Payment Service] Insufficient funds → Publish PaymentFailedEvent
[Saga] Sends ReleaseInventoryCommand (compensating)
[Saga] Sends CancelOrderCommand
[Inventory Service] Releases reservation
[Order Service] Updates order to Cancelled
[DONE] ✅ (compensated)
```

---

## 11. OUTBOX PATTERN {#outbox-pattern}

### 11.1 The Dual-Write Problem

```
// THIS IS BROKEN — not atomic:
await _orderRepo.SaveAsync(order);          // Step 1: DB write (succeeds)
await _bus.PublishAsync(orderPlacedEvent);  // Step 2: Message publish (CRASHES here)
// Order saved, but event never published → Inventory never reserved → Order stuck as Pending forever
```

### 11.2 What the Outbox Pattern Solves

Write the event to an **Outbox table** in the **same DB transaction** as the business data. A separate publisher reads the Outbox and publishes messages to the bus, marking them as sent.

```
BEGIN TRANSACTION
  INSERT INTO Orders VALUES (...)
  INSERT INTO OutboxMessages VALUES (OrderPlacedEvent serialized)
COMMIT
↓
[Outbox Publisher - separate process]
  SELECT unpublished messages FROM OutboxMessages
  FOR EACH message:
    publish to RabbitMQ/Kafka
    UPDATE OutboxMessages SET PublishedAt = NOW()
```

**Atomicity guaranteed:** Both succeed or both fail (same transaction). The message is never lost.

### 11.3 Implementation in .NET

```csharp
// 1. Outbox table entity
public class OutboxMessage
{
    public Guid Id { get; set; } = Guid.NewGuid();
    public string EventType { get; set; } = null!;
    public string Payload { get; set; } = null!;
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? PublishedAt { get; set; }
    public int RetryCount { get; set; }
    public string? Error { get; set; }
}

// 2. Save order + outbox message atomically
public class OrderCommandHandler : IRequestHandler<PlaceOrderCommand, PlaceOrderResult>
{
    private readonly AppDbContext _dbContext;

    public async Task<PlaceOrderResult> Handle(PlaceOrderCommand cmd, CancellationToken ct)
    {
        var order = Order.Create(cmd.CustomerId, cmd.Items);

        // Add both in same context → same transaction
        _dbContext.Orders.Add(order);
        _dbContext.OutboxMessages.Add(new OutboxMessage
        {
            EventType = nameof(OrderPlacedEvent),
            Payload = JsonSerializer.Serialize(new OrderPlacedEvent(
                order.Id, order.CustomerId, order.TotalAmount))
        });

        await _dbContext.SaveChangesAsync(ct);  // Single atomic commit
        return new PlaceOrderResult(order.Id, order.Status.ToString());
    }
}

// 3. Outbox Publisher (Polling)
public class OutboxPublisherService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly IPublishEndpoint _bus;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await PublishPendingMessagesAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
        }
    }

    private async Task PublishPendingMessagesAsync(CancellationToken ct)
    {
        using var scope = _scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        var pending = await db.OutboxMessages
            .Where(m => m.PublishedAt == null && m.RetryCount < 5)
            .OrderBy(m => m.CreatedAt)
            .Take(100)
            .ToListAsync(ct);

        foreach (var msg in pending)
        {
            try
            {
                var eventType = Type.GetType(msg.EventType)
                    ?? throw new InvalidOperationException($"Unknown event type: {msg.EventType}");

                var payload = JsonSerializer.Deserialize(msg.Payload, eventType)!;
                await _bus.Publish(payload, eventType, ct);

                msg.PublishedAt = DateTime.UtcNow;
            }
            catch (Exception ex)
            {
                msg.RetryCount++;
                msg.Error = ex.Message;
            }
        }

        await db.SaveChangesAsync(ct);
    }
}
```

### 11.4 Polling vs Transaction Log Tailing

| Approach | Mechanism | Latency | Complexity |
|----------|----------|---------|-----------|
| Polling Publisher | SELECT + UPDATE loop every N seconds | Seconds | Low |
| Debezium (CDC) | Reads DB transaction log directly | Milliseconds | High |

**Debezium** (Change Data Capture) captures every row change from PostgreSQL/MySQL write-ahead log. Zero polling overhead. Better for high-throughput systems.

---

## 12. RESILIENCE WITH POLLY {#resilience-with-polly}

### 12.1 Definition
Polly is a .NET resilience library that allows you to define policies for handling transient failures: retries, circuit breakers, timeouts, bulkheads, and fallbacks.

### 12.2 The Problem
In distributed systems, **transient failures are normal**:
- Network hiccup → HTTP call fails for 200ms then recovers
- Database connection pool exhausted momentarily
- Downstream service restarting (K8s pod rolling update)

Without resilience policies, a single transient failure cascades into a user error.

### 12.3 .NET 8 — Microsoft.Extensions.Http.Resilience

```csharp
// .NET 8 standard way — built on Polly v8 under the hood
// NuGet: Microsoft.Extensions.Http.Resilience

builder.Services.AddHttpClient<IOrderServiceClient, OrderServiceClient>()
    .AddStandardResilienceHandler(options =>
    {
        // Retry
        options.Retry.MaxRetryAttempts = 3;
        options.Retry.Delay = TimeSpan.FromMilliseconds(200);
        options.Retry.BackoffType = DelayBackoffType.Exponential;
        options.Retry.UseJitter = true; // Prevents retry storms

        // Circuit Breaker
        options.CircuitBreaker.SamplingDuration = TimeSpan.FromSeconds(30);
        options.CircuitBreaker.FailureRatio = 0.5;  // Open if 50% of requests fail
        options.CircuitBreaker.MinimumThroughput = 10;
        options.CircuitBreaker.BreakDuration = TimeSpan.FromSeconds(30);

        // Timeout
        options.AttemptTimeout.Timeout = TimeSpan.FromSeconds(5);
        options.TotalRequestTimeout.Timeout = TimeSpan.FromSeconds(30);
    });
```

### 12.4 Polly v8 — Manual Pipeline

```csharp
// NuGet: Polly, Polly.Extensions.Http

// Retry with exponential backoff + jitter
var retryPipeline = new ResiliencePipelineBuilder<HttpResponseMessage>()
    .AddRetry(new RetryStrategyOptions<HttpResponseMessage>
    {
        MaxRetryAttempts = 4,
        Delay = TimeSpan.FromSeconds(1),
        BackoffType = DelayBackoffType.Exponential,
        UseJitter = true,
        ShouldHandle = new PredicateBuilder<HttpResponseMessage>()
            .Handle<HttpRequestException>()
            .HandleResult(r => r.StatusCode == HttpStatusCode.TooManyRequests
                             || r.StatusCode >= HttpStatusCode.InternalServerError),
        OnRetry = args =>
        {
            Console.WriteLine($"Retry attempt {args.AttemptNumber} after {args.RetryDelay.TotalSeconds}s");
            return ValueTask.CompletedTask;
        }
    })
    .Build();

// Circuit Breaker — Closed/Open/Half-Open States
var circuitBreakerPipeline = new ResiliencePipelineBuilder<HttpResponseMessage>()
    .AddCircuitBreaker(new CircuitBreakerStrategyOptions<HttpResponseMessage>
    {
        SamplingDuration = TimeSpan.FromSeconds(10),
        FailureRatio = 0.5,
        MinimumThroughput = 8,
        BreakDuration = TimeSpan.FromSeconds(30),
        OnOpened = args =>
        {
            Console.WriteLine($"Circuit OPENED — will not attempt calls for {args.BreakDuration}");
            return ValueTask.CompletedTask;
        },
        OnClosed = _ =>
        {
            Console.WriteLine("Circuit CLOSED — calls resuming");
            return ValueTask.CompletedTask;
        },
        OnHalfOpened = _ =>
        {
            Console.WriteLine("Circuit HALF-OPEN — trial call");
            return ValueTask.CompletedTask;
        }
    })
    .Build();

// Timeout
var timeoutPipeline = new ResiliencePipelineBuilder()
    .AddTimeout(new TimeoutStrategyOptions
    {
        Timeout = TimeSpan.FromSeconds(5),
        OnTimeout = args =>
        {
            Console.WriteLine($"Request timed out after {args.Timeout.TotalSeconds}s");
            return ValueTask.CompletedTask;
        }
    })
    .Build();
```

### 12.5 Circuit Breaker States

```
CLOSED (normal):
  Requests flow through → failures counted
  If failure ratio > threshold → OPEN

OPEN (failing fast):
  All requests immediately fail with BrokenCircuitException
  After BreakDuration → HALF-OPEN

HALF-OPEN (testing):
  One trial request allowed through
  Success → CLOSED
  Failure → OPEN again
```

**Why Circuit Breaker?** If Payment Service is down and you keep retrying, you:
1. Waste threads waiting for timeouts
2. Exhaust connection pools
3. Cascade failures to Order Service

Circuit breaker **fails fast**, freeing threads immediately.

### 12.6 Bulkhead Isolation

```csharp
// Isolate resources — prevent one slow downstream from consuming all threads
var bulkheadPipeline = new ResiliencePipelineBuilder()
    .AddConcurrencyLimiter(new ConcurrencyLimiterStrategyOptions
    {
        PermitLimit = 10,        // Max 10 concurrent calls to this service
        QueueLimit = 5           // Queue up to 5 more, reject the rest
    })
    .Build();
```

### 12.7 Fallback

```csharp
var fallbackPipeline = new ResiliencePipelineBuilder<ProductDto?>()
    .AddFallback(new FallbackStrategyOptions<ProductDto?>
    {
        ShouldHandle = new PredicateBuilder<ProductDto?>()
            .Handle<Exception>(),
        FallbackAction = _ =>
            Outcome.FromResultAsValueTask<ProductDto?>(GetCachedProduct())
    })
    .Build();
```

---

## 13. DISTRIBUTED TRACING {#distributed-tracing}

### 13.1 Correlation IDs

Without correlation, when a request fails across 5 services, finding which service failed is like finding a needle in a haystack.

```csharp
// Middleware to propagate Correlation ID
public class CorrelationIdMiddleware
{
    private const string CorrelationIdHeader = "X-Correlation-Id";
    private readonly RequestDelegate _next;

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.TryGetValue(CorrelationIdHeader, out var correlationId)
            || string.IsNullOrEmpty(correlationId))
        {
            correlationId = Guid.NewGuid().ToString();
        }

        context.Items[CorrelationIdHeader] = correlationId.ToString();
        context.Response.Headers[CorrelationIdHeader] = correlationId.ToString();

        // Add to all logs
        using var logScope = _logger.BeginScope(
            new Dictionary<string, object> { ["CorrelationId"] = correlationId.ToString() });

        await _next(context);
    }
}
```

### 13.2 OpenTelemetry in .NET

```csharp
// NuGet: OpenTelemetry.Sdk, OpenTelemetry.Exporter.Jaeger,
//        OpenTelemetry.Instrumentation.AspNetCore, OpenTelemetry.Instrumentation.Http

builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing
            .SetResourceBuilder(ResourceBuilder.CreateDefault()
                .AddService("order-service", serviceVersion: "1.0"))
            .AddAspNetCoreInstrumentation(opts =>
            {
                opts.RecordException = true;
                opts.Filter = ctx => !ctx.Request.Path.StartsWithSegments("/health");
            })
            .AddHttpClientInstrumentation()
            .AddEntityFrameworkCoreInstrumentation()
            .AddSource("OrderService.*")            // Custom activity sources
            .AddJaegerExporter(opts =>
            {
                opts.AgentHost = "jaeger";
                opts.AgentPort = 6831;
            });
    })
    .WithMetrics(metrics =>
    {
        metrics
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddRuntimeInstrumentation()
            .AddPrometheusExporter();
    });

// Custom span in application code
private static readonly ActivitySource _activitySource = new("OrderService.OrderProcessing");

public async Task<Order> ProcessOrderAsync(Guid orderId)
{
    using var activity = _activitySource.StartActivity("ProcessOrder");
    activity?.SetTag("order.id", orderId);
    activity?.SetTag("service", "OrderService");

    try
    {
        var order = await _repo.GetByIdAsync(orderId);
        activity?.SetTag("order.status", order?.Status.ToString());
        return order!;
    }
    catch (Exception ex)
    {
        activity?.RecordException(ex);
        activity?.SetStatus(ActivityStatusCode.Error, ex.Message);
        throw;
    }
}
```

### 13.3 Distributed Trace Context Propagation

OpenTelemetry auto-propagates `traceparent` / `tracestate` headers (W3C Trace Context standard) across HTTP calls. Every service that processes an HTTP request with a `traceparent` header joins the same trace.

```
Request enters Order Service → Trace ID: abc123, Span ID: span-1
  Order Service calls Inventory Service (HTTP) → traceparent: 00-abc123-span-1-01
    Inventory Service creates child span → Trace ID: abc123 (same!), Span ID: span-2
      Inventory Service calls DB → Trace ID: abc123, Span ID: span-3

→ Jaeger UI shows entire waterfall for trace abc123
```

---

## 14. SERVICE MESH {#service-mesh}

### 14.1 What a Service Mesh Does

A service mesh is an infrastructure layer that handles **service-to-service communication** transparently via sidecar proxies (e.g., Envoy). Each pod gets a proxy sidecar that intercepts all network traffic.

```
Pod A (App + Envoy sidecar) ─── mTLS, retries, tracing ──→ Pod B (App + Envoy sidecar)
                                [Control Plane: Istio/Linkerd manages all proxies]
```

**What service mesh provides:**
- **mTLS:** Mutual TLS between all services — zero-trust networking
- **Traffic management:** Canary deployments, A/B testing, circuit breaking
- **Observability:** Automatic distributed tracing and metrics
- **Retries & timeouts:** Without code changes

### 14.2 When You Need It vs When You Don't

| Need It | Don't Need It |
|---------|--------------|
| 10+ microservices | < 5 services |
| Strict zero-trust security | Dev environment |
| Canary/shadow traffic | Simple routing |
| Multi-language (polyglot) | All .NET, Polly handles resilience |
| Compliance requiring mTLS everywhere | Internal-only services |

---

## 15. INTERVIEW Q&A — MASTER LIST {#interview-qa}

### Architecture Questions

**Q: What is the difference between Choreography and Orchestration sagas?**
> Choreography: decentralized, each service reacts to events, hard to visualize flow but no SPOF. Orchestration: central state machine drives the saga, clear visibility, but orchestrator can become a bottleneck. Use choreography for simple 2-3 step flows; orchestration for complex workflows needing auditability.

**Q: How do you prevent double-processing a message in Kafka?**
> Make consumers idempotent. Maintain a processed-events table: `INSERT INTO ProcessedEvents(eventId) ON CONFLICT DO NOTHING`. If insert succeeds → process. If conflict → skip (already processed). Alternatively use Kafka transactions with exactly-once semantics, but idempotent consumers are simpler.

**Q: Explain the Outbox Pattern. Why is it necessary?**
> Without it, saving to DB and publishing a message are two separate operations. If the app crashes between them, data is inconsistent. The Outbox pattern writes the message to an outbox table in the SAME transaction as the business data. A separate publisher reads and publishes it. Atomicity is guaranteed.

**Q: When would you choose Event Sourcing?**
> When you need: complete audit trail (finance, healthcare), temporal queries (state at time T), event replay for new services, or complex domain with rich behavioral history. NOT for simple CRUD — it adds significant complexity.

**Q: How does Circuit Breaker help in microservices?**
> When a downstream service is failing, without circuit breaker: every request waits for timeout (5s), threads pile up, connection pools exhaust, upstream service also becomes slow. With circuit breaker: after failure threshold, all requests fail immediately (fast fail). Resources freed. After break duration, trial request sent — if succeeds, circuit closes.

**Q: What is the CAP theorem relevance to microservices?**
> Each microservice's database makes a CAP trade-off. In an Order service using PostgreSQL with synchronous replication: we favor Consistency over Availability. In a Notification service using Cassandra: we favor Availability over Consistency. Microservices let each service choose its own CAP trade-off rather than a monolith's one-size-fits-all.

**Q: How do you handle distributed transactions without 2PC?**
> Use the Saga pattern. Each service performs a local transaction and publishes an event. If any step fails, compensating transactions undo the previous steps. Eventual consistency is accepted.

**Q: What is your approach to ensuring message ordering in Kafka?**
> Use a consistent partition key (e.g., orderId or customerId). All events for the same entity land on the same partition, guaranteeing order. Never use random partition assignment when ordering matters. One consumer per partition per consumer group — within a partition, messages are always FIFO.

**Q: What's the difference between gRPC and REST? When would you use each?**
> REST over JSON: human-readable, universal browser support, good for public APIs. gRPC over Protocol Buffers: binary (~5x smaller payload), strongly typed contracts, HTTP/2 multiplexing, streaming support. Use gRPC for internal service-to-service calls where performance and schema enforcement matter. Use REST for public APIs and browser clients.

**Q: How does the Outbox pattern differ from 2PC?**
> 2PC requires a distributed transaction coordinator and locks across multiple systems — fragile and blocking. The Outbox pattern uses a single local transaction (always fast and safe), then eventual delivery via a background publisher. It trades strict consistency for resilience and simplicity.

---

*Volume 9 complete — approximately 8,500 words of production-grade microservices architecture knowledge.*
