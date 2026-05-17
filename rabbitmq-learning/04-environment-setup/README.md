# 04 — Environment Setup

> **Goal:** Get RabbitMQ running locally using Docker and explore the Management Dashboard.

---

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed
- Port `5672` (AMQP) and `15672` (Management UI) available

---

## 1. Run RabbitMQ with Docker

### Quick Start (Single Command)

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  rabbitmq:3-management
```

| Flag | Purpose |
|------|---------|
| `-d` | Run in background (detached) |
| `--name rabbitmq` | Container name for easy reference |
| `-p 5672:5672` | AMQP protocol port (clients connect here) |
| `-p 15672:15672` | Management UI port |
| `rabbitmq:3-management` | Image with management plugin pre-installed |

### With Persistent Data (Recommended)

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -v rabbitmq-data:/var/lib/rabbitmq \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3-management
```

| Variable | Purpose |
|----------|---------|
| `RABBITMQ_DEFAULT_USER` | Override default username |
| `RABBITMQ_DEFAULT_PASS` | Override default password |
| `-v rabbitmq-data:/var/lib/rabbitmq` | Persist messages/config across restarts |

---

## 2. Verify It's Running

```bash
# Check container is up
docker ps

# View logs (wait for "Server startup complete")
docker logs rabbitmq

# Expected output includes:
#  RabbitMQ 3.x.x
#  Starting broker... completed
```

---

## 3. Access the Management Dashboard

Open your browser:

```
URL:      http://localhost:15672
Username: guest
Password: guest
```

> If you set custom credentials above, use those instead.

### Dashboard Overview

```
┌─────────────────────────────────────────────────────┐
│  RabbitMQ Management                                 │
├───────┬──────────┬──────────┬───────────┬───────────┤
│Overview│Connections│Channels │Exchanges  │ Queues    │
└───────┴──────────┴──────────┴───────────┴───────────┘
```

**Tabs explained:**

| Tab | What you can do |
|-----|----------------|
| **Overview** | See message rates, node health |
| **Connections** | View active AMQP connections |
| **Channels** | View active channels per connection |
| **Exchanges** | List/create/delete exchanges |
| **Queues** | List/create/delete queues, view messages |
| **Admin** | Manage users, virtual hosts, policies |

---

## 4. Docker Compose Setup (Full Stack)

For a more complete development setup with a `.NET` app:

```yaml
# docker-compose.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  rabbitmq-data:
```

```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# Stop and remove data
docker-compose down -v
```

---

## 5. Useful Docker Commands

```bash
# Start / Stop container
docker start rabbitmq
docker stop rabbitmq

# View real-time logs
docker logs -f rabbitmq

# Open shell inside container
docker exec -it rabbitmq bash

# RabbitMQ CLI inside container
docker exec rabbitmq rabbitmqctl list_queues
docker exec rabbitmq rabbitmqctl list_exchanges
docker exec rabbitmq rabbitmqctl list_bindings
docker exec rabbitmq rabbitmqctl status

# Reset everything
docker stop rabbitmq && docker rm rabbitmq
```

---

## 6. Connection String Reference

### .NET / AMQP URI Format

```
amqp://username:password@hostname:port/vhost
```

| Environment | Connection String |
|-------------|-------------------|
| Local (default) | `amqp://guest:guest@localhost:5672/` |
| Local (custom) | `amqp://admin:admin123@localhost:5672/` |
| Docker Compose | `amqp://admin:admin123@rabbitmq:5672/` |
| Cloud | `amqps://user:pass@your-host.com:5671/` |

> Use `amqps://` (with S) for TLS/SSL connections in production.

---

## 7. Ports Reference

| Port | Protocol | Purpose |
|------|----------|---------|
| `5672` | AMQP | Main messaging protocol |
| `5671` | AMQP/TLS | Encrypted AMQP |
| `15672` | HTTP | Management UI & REST API |
| `15671` | HTTPS | Management UI with TLS |
| `4369` | EPMD | Erlang Port Mapper (clustering) |
| `25672` | Erlang | Inter-node communication |

---

## 8. First Verification Test

Publish and consume a test message via the Management UI:

1. Go to **Queues** tab → **Add a new queue**
   - Name: `test-queue`
   - Durable: ✅
   - Click **Add queue**

2. Go to **Exchanges** tab → Default exchange `(AMQP default)`
   - Scroll to **Publish message**
   - Routing key: `test-queue`
   - Payload: `Hello RabbitMQ!`
   - Click **Publish message**

3. Back in **Queues** → click `test-queue`
   - Scroll to **Get messages**
   - Click **Get Message(s)**
   - You should see your message! 🎉

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Port already in use | `lsof -i :5672` to find conflicting process |
| Can't login to UI | Default user `guest` only works on localhost |
| Container exits immediately | Check logs: `docker logs rabbitmq` |
| Messages not persisting | Add `-v rabbitmq-data:/var/lib/rabbitmq` |

---

**Previous:** [03 — Exchange Types ←](../03-exchange-types/README.md)  
**Next:** [05 — Hello World →](../05-hello-world/README.md)
