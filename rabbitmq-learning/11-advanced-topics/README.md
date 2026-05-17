# 11 — Advanced Topics

> **Goal:** Master clustering, high availability, federation, and performance tuning for enterprise RabbitMQ deployments.

---

## 1. RabbitMQ Clustering

A **cluster** is multiple RabbitMQ nodes that share definitions (exchanges, queues, users) but not queue contents by default.

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Node 1     │◄──►│   Node 2     │◄──►│   Node 3     │
│ rabbit@host1 │    │ rabbit@host2 │    │ rabbit@host3 │
│              │    │              │    │              │
│ [queue-A]   │    │              │    │              │
│             │    │ [queue-B]    │    │ [queue-C]    │
└──────────────┘    └──────────────┘    └──────────────┘
         ▲
    Metadata replicated to all nodes
    Queue contents only on owning node (by default)
```

### Docker Compose Cluster (3 Nodes)

```yaml
version: '3.8'

services:
  rabbitmq1:
    image: rabbitmq:3-management
    hostname: rabbitmq1
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret-cluster-cookie"  # must be identical on all nodes
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    ports:
      - "5672:5672"
      - "15672:15672"

  rabbitmq2:
    image: rabbitmq:3-management
    hostname: rabbitmq2
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret-cluster-cookie"
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    ports:
      - "5673:5672"
      - "15673:15672"
    depends_on: [rabbitmq1]

  rabbitmq3:
    image: rabbitmq:3-management
    hostname: rabbitmq3
    environment:
      RABBITMQ_ERLANG_COOKIE: "secret-cluster-cookie"
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    ports:
      - "5674:5672"
      - "15674:15672"
    depends_on: [rabbitmq1]
```

```bash
# Join nodes to cluster
docker exec rabbitmq2 rabbitmqctl stop_app
docker exec rabbitmq2 rabbitmqctl join_cluster rabbit@rabbitmq1
docker exec rabbitmq2 rabbitmqctl start_app

docker exec rabbitmq3 rabbitmqctl stop_app
docker exec rabbitmq3 rabbitmqctl join_cluster rabbit@rabbitmq1
docker exec rabbitmq3 rabbitmqctl start_app

# Verify cluster
docker exec rabbitmq1 rabbitmqctl cluster_status
```

---

## 2. High Availability (Quorum Queues)

Classic HA mirrored queues are deprecated in RabbitMQ 3.9+. Use **Quorum Queues** instead.

### Quorum Queues

A quorum queue replicates messages across a majority of nodes using Raft consensus.

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Node 1  │    │  Node 2  │    │  Node 3  │
│ [leader] │◄──►│[follower]│◄──►│[follower]│
│  queue   │    │  queue   │    │  queue   │
└──────────┘    └──────────┘    └──────────┘
   All 3 nodes have a copy — majority (2/3) must acknowledge writes
```

```csharp
// Declare a quorum queue
channel.QueueDeclare("orders-quorum", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-queue-type",      "quorum" },   // ← enables quorum queue
        { "x-quorum-initial-group-size", 3 } // replicate to 3 nodes
    });
```

```bash
# Check quorum queue status via CLI
rabbitmqctl list_quorum_queues
```

### Quorum vs Classic Queue

| Feature | Classic Queue | Quorum Queue |
|---------|--------------|--------------|
| Replication | Optional (mirroring) | Built-in (Raft) |
| Data safety | Risk of split-brain | Strong consistency |
| Max priority | 255 | Not supported |
| Dead letter | Supported | Supported |
| Lazy mode | Supported | Default behavior |
| Recommended for | Dev/simple | Production |

---

## 3. Shovel Plugin

The **Shovel** plugin moves messages between queues — within a broker or across brokers.

```
Source Broker (Datacenter A)          Destination Broker (Datacenter B)
┌────────────────────────┐            ┌────────────────────────┐
│  [source-queue]        │──[Shovel]──►│  [dest-queue]          │
│  "amqp://host-a:5672/" │            │  "amqp://host-b:5672/" │
└────────────────────────┘            └────────────────────────┘
```

### Enable and Configure

```bash
# Enable plugin
rabbitmq-plugins enable rabbitmq_shovel
rabbitmq-plugins enable rabbitmq_shovel_management
```

```bash
# Configure via CLI
rabbitmqctl set_parameter shovel my-shovel \
  '{"src-protocol": "amqp091",
    "src-uri": "amqp://guest:guest@host-a",
    "src-queue": "source-queue",
    "dest-protocol": "amqp091",
    "dest-uri": "amqp://guest:guest@host-b",
    "dest-queue": "dest-queue",
    "ack-mode": "on-confirm"}'
```

**Use cases:**
- Migrating data between brokers
- Cross-region message forwarding
- Queue bridging between environments

---

## 4. Federation Plugin

**Federation** links exchanges or queues across brokers without clustering. Useful across datacenters or organizations.

```
Region: Europe                    Region: USA
┌──────────────────┐              ┌──────────────────┐
│  [orders-EU]     │──[Federation]►│  [orders-global] │
│ exchange         │              │ exchange         │
└──────────────────┘              └──────────────────┘
```

```bash
# Enable
rabbitmq-plugins enable rabbitmq_federation
rabbitmq-plugins enable rabbitmq_federation_management

# Setup upstream
rabbitmqctl set_parameter federation-upstream eu-upstream \
  '{"uri": "amqp://user:pass@eu.broker.com", "max-hops": 1}'

# Apply policy
rabbitmqctl set_policy federate-orders "^orders" \
  '{"federation-upstream-set": "all"}' \
  --apply-to exchanges
```

### Shovel vs Federation

| | Shovel | Federation |
|-|--------|------------|
| Direction | One-way | Bidirectional possible |
| Topology | Point-to-point | Hub and spoke |
| Use case | Migration, bridging | Multi-region replication |
| Message ack | After move | Per-hop |

---

## 5. Performance Tuning

### Publisher Side

```csharp
// ❌ Slow: confirm after every message
for (int i = 0; i < 1000; i++)
{
    channel.BasicPublish(...);
    channel.WaitForConfirms(); // blocks!
}

// ✅ Fast: batch publishing with async confirms
channel.ConfirmSelect();
for (int i = 0; i < 1000; i++)
{
    channel.BasicPublish(...);
}
channel.WaitForConfirmsOrDie(TimeSpan.FromSeconds(30)); // wait once for all
```

### Consumer Side

```csharp
// ❌ Slow: process one at a time
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);

// ✅ Better throughput: process in batches
channel.BasicQos(prefetchSize: 0, prefetchCount: 50, global: false);
```

### RabbitMQ Server Tuning

```bash
# rabbitmq.conf
## Memory threshold — pause publishers at 70% RAM
vm_memory_high_watermark.relative = 0.7

## Disk space — pause publishers when < 2GB free
disk_free_limit.absolute = 2GB

## Max file handles (raise OS limit too)
# ulimit -n 65536
## Then in rabbitmq.conf:
# (set in OS-level systemd service)

## Enable lazy queues (writes to disk sooner, saves RAM)
# Set x-queue-mode: lazy on queue declaration
```

```csharp
// Lazy queue — good for large queues, saves memory
channel.QueueDeclare("large-queue", durable: true, exclusive: false, autoDelete: false,
    arguments: new Dictionary<string, object>
    {
        { "x-queue-mode", "lazy" } // messages go to disk immediately
    });
```

### Throughput Benchmarks (Approximate)

| Configuration | Messages/sec |
|---------------|-------------|
| Single node, transient, no confirms | ~50,000 |
| Single node, persistent, with confirms | ~5,000 |
| Cluster (3 nodes), quorum queue | ~3,000 |
| Lazy queue, persistent | ~10,000 |

---

## 6. Event-Driven Architecture Patterns

### Saga Pattern (Distributed Transactions)

```
OrderCreated
    │
    ▼
[Saga Orchestrator]
    ├──► Reserve Inventory  ──► InventoryReserved / InventoryFailed
    ├──► Process Payment    ──► PaymentProcessed / PaymentFailed
    └──► Send Notification  ──► NotificationSent

On any failure:
    InventoryFailed ──► [Compensate] ──► CancelOrder, ReleaseInventory
```

### Outbox Pattern (Reliable Publishing)

Ensures messages are always published even if the app crashes after DB write but before publish.

```
1. In same DB transaction:
   INSERT order → orders table
   INSERT event → outbox table (pending)

2. Background worker polls outbox:
   SELECT * FROM outbox WHERE status = 'pending'

3. Publish to RabbitMQ

4. Mark outbox record as 'published'
```

```csharp
// Outbox record
public class OutboxMessage
{
    public Guid     Id          { get; set; } = Guid.NewGuid();
    public string   EventType   { get; set; } = "";
    public string   Payload     { get; set; } = "";
    public string   Exchange    { get; set; } = "";
    public string   RoutingKey  { get; set; } = "";
    public string   Status      { get; set; } = "pending"; // pending | published | failed
    public DateTime CreatedAt   { get; set; } = DateTime.UtcNow;
    public DateTime? PublishedAt { get; set; }
}
```

### Event Sourcing with RabbitMQ

```
Command → Aggregate → Event stored in EventStore → Published to RabbitMQ
                                                         │
                                              All subscribers project
                                              their own read models
```

---

## Advanced Concepts Summary

| Topic | When to Use |
|-------|------------|
| **Clustering** | Horizontal scaling, eliminate single point of failure |
| **Quorum Queues** | Any production workload requiring HA |
| **Shovel** | Migration, cross-broker message bridging |
| **Federation** | Multi-datacenter / multi-region replication |
| **Performance Tuning** | High throughput (> 10k msg/sec) workloads |
| **Saga Pattern** | Multi-step distributed transactions |
| **Outbox Pattern** | 100% reliable publish guarantee |
| **Event Sourcing** | Full audit trail, temporal queries |

---

## Further Reading

- [RabbitMQ Official Documentation](https://www.rabbitmq.com/documentation.html)
- [Quorum Queues Guide](https://www.rabbitmq.com/quorum-queues.html)
- [Clustering Guide](https://www.rabbitmq.com/clustering.html)
- [Federation Plugin](https://www.rabbitmq.com/federation.html)
- [Shovel Plugin](https://www.rabbitmq.com/shovel.html)
- [RabbitMQ .NET Client Docs](https://rabbitmq.github.io/rabbitmq-dotnet-client/)
- [MassTransit (Higher-level .NET library)](https://masstransit.io/)

---

**Previous:** [10 — Production Best Practices ←](../10-production-best-practices/README.md)  
**Back to start:** [01 — Fundamentals →](../01-fundamentals/README.md)
