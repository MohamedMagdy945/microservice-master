# 05 — Hello World (First Hands-On)

> **Goal:** Build your first producer and consumer in .NET — send a message and receive it.

---

## The Flow

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Producer   │─────►│    Queue     │─────►│   Consumer   │
│  (sends msg) │      │ (hello-queue)│      │ (reads msg)  │
└──────────────┘      └──────────────┘      └──────────────┘
```

---

## Project Setup

```bash
# Create two console apps
dotnet new console -n HelloProducer
dotnet new console -n HelloConsumer

# Add RabbitMQ client to both
cd HelloProducer && dotnet add package RabbitMQ.Client
cd ../HelloConsumer && dotnet add package RabbitMQ.Client
```

---

## Producer — Send a Message

```csharp
// HelloProducer/Program.cs
using RabbitMQ.Client;
using System.Text;

// 1. Create connection factory
var factory = new ConnectionFactory
{
    HostName = "localhost",
    Port     = 5672,
    UserName = "guest",
    Password = "guest"
};

// 2. Open connection and channel
using var connection = factory.CreateConnection();
using var channel    = connection.CreateModel();

// 3. Declare the queue (idempotent — safe to call every time)
channel.QueueDeclare(
    queue:      "hello-queue",
    durable:    false,   // won't survive RabbitMQ restart
    exclusive:  false,   // other consumers can connect
    autoDelete: false,   // won't auto-delete
    arguments:  null
);

// 4. Prepare and publish the message
string message = "Hello, RabbitMQ! 🐇";
var body = Encoding.UTF8.GetBytes(message);

channel.BasicPublish(
    exchange:   "",            // default exchange
    routingKey: "hello-queue", // route to queue by name
    basicProperties: null,
    body:       body
);

Console.WriteLine($"[x] Sent: {message}");
```

**Run it:**
```bash
cd HelloProducer
dotnet run
# Output: [x] Sent: Hello, RabbitMQ! 🐇
```

---

## Consumer — Receive Messages

```csharp
// HelloConsumer/Program.cs
using RabbitMQ.Client;
using RabbitMQ.Client.Events;
using System.Text;

var factory = new ConnectionFactory
{
    HostName = "localhost",
    Port     = 5672,
    UserName = "guest",
    Password = "guest"
};

using var connection = factory.CreateConnection();
using var channel    = connection.CreateModel();

// Declare the same queue (safe if already exists)
channel.QueueDeclare(
    queue:      "hello-queue",
    durable:    false,
    exclusive:  false,
    autoDelete: false,
    arguments:  null
);

// Limit to processing one message at a time
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);

Console.WriteLine("[*] Waiting for messages. Press CTRL+C to exit.");

// Create event-driven consumer
var consumer = new EventingBasicConsumer(channel);

consumer.Received += (model, ea) =>
{
    // Decode the message
    var body    = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);

    Console.WriteLine($"[✓] Received: {message}");

    // Acknowledge — tell RabbitMQ the message was processed
    channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
};

// Start consuming
channel.BasicConsume(
    queue:     "hello-queue",
    autoAck:   false,   // manual acknowledgment
    consumer:  consumer
);

// Keep alive
Console.ReadLine();
```

**Run it:**
```bash
cd HelloConsumer
dotnet run
# Output: [*] Waiting for messages. Press CTRL+C to exit.
# Output: [✓] Received: Hello, RabbitMQ! 🐇
```

---

## Step-by-Step Walkthrough

### Step 1: ConnectionFactory
```csharp
var factory = new ConnectionFactory { HostName = "localhost" };
```
Configures how to connect (host, port, credentials, vhost).

### Step 2: Connection
```csharp
using var connection = factory.CreateConnection();
```
Opens a TCP connection to RabbitMQ. Expensive to create — reuse it.

### Step 3: Channel
```csharp
using var channel = connection.CreateModel();
```
Opens a lightweight virtual channel over the connection. Use one per thread.

### Step 4: Queue Declaration
```csharp
channel.QueueDeclare("hello-queue", durable: false, ...);
```
Creates the queue if it doesn't exist. Safe to call multiple times (idempotent).

### Step 5: Publish (Producer)
```csharp
channel.BasicPublish(exchange: "", routingKey: "hello-queue", body: body);
```
Sends message to default exchange → routes to `hello-queue` by name.

### Step 6: Consume (Consumer)
```csharp
var consumer = new EventingBasicConsumer(channel);
consumer.Received += (model, ea) => { /* handle message */ };
channel.BasicConsume("hello-queue", autoAck: false, consumer);
```
Event-driven — fires `Received` for each message delivered.

### Step 7: Acknowledge
```csharp
channel.BasicAck(ea.DeliveryTag, multiple: false);
```
Tells RabbitMQ the message was processed successfully. Remove from queue.

---

## What to Observe in the Dashboard

1. Open http://localhost:15672
2. Go to **Queues** tab
3. You should see `hello-queue`

**Before consumer starts:**
```
hello-queue | Messages: 1 | Consumers: 0
```

**After consumer starts:**
```
hello-queue | Messages: 0 | Consumers: 1
```

---

## Understanding autoAck

| Mode | Behavior | Risk |
|------|----------|------|
| `autoAck: true` | Acknowledged on delivery | Message lost if consumer crashes mid-processing |
| `autoAck: false` | Must call `BasicAck` manually | Safe — message requeued if consumer crashes |

**Always use `autoAck: false` in production.**

---

## Experiment Ideas

1. **Start consumer first, then producer** — messages queue up and deliver on connection
2. **Run multiple consumers** — messages are distributed round-robin
3. **Stop consumer mid-send** — messages wait in queue until consumer restarts
4. **Send 10 messages, one consumer** — observe queue depth in dashboard

```csharp
// Send 10 messages
for (int i = 1; i <= 10; i++)
{
    var msg = $"Message #{i}";
    var body = Encoding.UTF8.GetBytes(msg);
    channel.BasicPublish("", "hello-queue", null, body);
    Console.WriteLine($"[x] Sent: {msg}");
    Thread.Sleep(500);
}
```

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Connection refused` | RabbitMQ not running | `docker start rabbitmq` |
| `ACCESS_REFUSED` | Wrong credentials | Check username/password |
| `PRECONDITION_FAILED` | Queue exists with different settings | Delete queue in dashboard and redeclare |
| Message not received | autoAck issue | Check `BasicAck` is being called |

---

**Previous:** [04 — Environment Setup ←](../04-environment-setup/README.md)  
**Next:** [06 — .NET Integration Basics →](../06-dotnet-integration/README.md)
