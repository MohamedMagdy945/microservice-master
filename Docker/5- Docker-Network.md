# Docker Networks

## Definition

Docker network is a system that allows containers to communicate with each other and with external systems. It provides isolation and connectivity between containers.

---

## Importance

* Enables communication between containers
* Isolates container environments
* Supports microservices architecture
* Allows secure and controlled networking

---

## Types of Networks

* Bridge: Default network for containers on the same host
* Host: Uses the host’s networking directly
* None: Disables networking for the container
* Overlay: Connects containers across multiple hosts (used in Docker Swarm)

---

## Commands

Create a network:

```bash
docker network create my_network
```

List networks:

```bash
docker network ls
```

Inspect a network:

```bash
docker network inspect my_network
```

Remove a network:

```bash
docker network rm my_network
```

---

## Usage in Containers

Run container with a network:

```bash
docker run -d --network my_network nginx
```

Connect existing container to a network:

```bash
docker network connect my_network container_id
```

This allows containers within the same network to communicate using container names.
