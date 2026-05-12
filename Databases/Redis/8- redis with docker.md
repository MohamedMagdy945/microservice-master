# Redis with Docker

Running Redis in containerized environments makes deployment easier, more scalable, and consistent across development and production.

This guide covers running Redis using Docker, Docker Compose, volumes, networking, and Kubernetes.

---

# Table of Contents

1. Redis Containers
2. Docker Compose
3. Redis Volumes
4. Container Networking
5. Redis in Kubernetes

---

# 1. Redis Containers

Redis can be easily run using Docker containers.

## Run Redis Container

```bash
docker run -d --name redis-server -p 6379:6379 redis
```

---

## Explanation

| Option | Description          |
| ------ | -------------------- |
| -d     | Run in background    |
| --name | Container name       |
| -p     | Port mapping         |
| redis  | Official Redis image |

---

## Connect to Redis CLI

```bash
docker exec -it redis-server redis-cli
```

---

## Verify Redis

```bash
PING
```

Response:

```bash
PONG
```

---

# 2. Docker Compose

Docker Compose is used to define multi-container applications.

---

## Basic Redis Compose File

```yaml
version: '3.8'

services:
  redis:
    image: redis
    container_name: redis-server
    ports:
      - "6379:6379"
```

---

## Run Compose

```bash
docker-compose up -d
```

---

## Stop Compose

```bash
docker-compose down
```

---

## Why Use Docker Compose?

* Easy environment setup
* Reproducible deployments
* Supports multiple services
* Simplifies development workflow

---

# 3. Redis Volumes

Redis volumes are used to persist data outside the container lifecycle.

Without volumes, all data is lost when container stops.

---

## Add Volume to Docker

```bash
docker run -d \
  --name redis-server \
  -p 6379:6379 \
  -v redis-data:/data \
  redis
```

---

## Docker Compose with Volume

```yaml
version: '3.8'

services:
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

---

## Benefits of Volumes

* Data persistence
* Safe container restarts
* Backup support
* Production readiness

---

# 4. Container Networking

Docker networking allows containers to communicate with each other.

---

## Default Bridge Network

Containers can communicate using service names.

Example:

```bash
redis://redis-server:6379
```

---

## Custom Network Example

```bash
docker network create redis-net
```

```bash
docker run -d --name redis --network redis-net redis
```

---

## Docker Compose Networking

Compose automatically creates a network.

Example:

```yaml
services:
  redis:
    image: redis

  app:
    build: .
    depends_on:
      - redis
```

App can connect using:

```text
redis:6379
```

---

## Benefits

* Service discovery
* Isolated environments
* Easy scaling
* Secure communication

---

# 5. Redis in Kubernetes

Redis can be deployed in Kubernetes for scalability and high availability.

---

## Redis Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
```

---

## Redis Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

---

## Types of Kubernetes Services

| Type         | Description              |
| ------------ | ------------------------ |
| ClusterIP    | Internal access only     |
| NodePort     | External access via node |
| LoadBalancer | Cloud load balancer      |

---

## Why Use Redis in Kubernetes?

* High availability
* Auto scaling
* Self-healing pods
* Easy deployment automation

---

## Production Recommendation

For production Redis in Kubernetes:

* Use StatefulSets
* Enable persistence volumes
* Configure replication
* Use Redis Sentinel or Cluster mode

---

# Summary

Using Redis with Docker and Kubernetes provides:

* Easy deployment
* Scalability
* Portability
* High availability
* Production readiness

Combining containers with Redis is the standard approach for modern distributed systems.
