# Docker Volumes

## Definition

A Docker volume is a persistent storage mechanism used to store data generated and used by Docker containers. Volumes exist independently of the container lifecycle, so data is not lost when a container is removed.

---

## Importance

* Ensures data persistence after container deletion
* Allows sharing data between multiple containers
* Improves performance compared to container filesystem
* Recommended way to manage persistent data in Docker

---

## Types

* Volumes: Managed by Docker
* Bind Mounts: Map a host directory to a container
* tmpfs: Temporary storage in memory

---

## Commands

Create a volume:

```bash
docker volume create my_volume
```

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect my_volume
```

Remove a volume:

```bash
docker volume rm my_volume
```

---

## Usage in Container

Run container with volume:

```bash
docker run -d -v my_volume:/app/data nginx
```

This mounts the volume to the container and keeps data persistent even if the container is removed.
