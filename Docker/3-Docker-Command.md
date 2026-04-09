# Docker Guide

## Docker Basic Commands

### Pull Images

Used to download Docker images from Docker Hub.

```bash
docker pull nginx
docker pull ubuntu
docker pull node
```

Pull specific version:

```bash
docker pull nginx:latest
```

---

### List Images

Show all images available locally.

```bash
docker images
```

---

### Run Containers

Create and run a container from an image.

Run container:

```bash
docker run nginx
```

Run in background (detached mode):

```bash
docker run -d nginx
```

Run with port mapping:

```bash
docker run -d -p 8080:80 nginx
```

---

### List Containers

```bash
docker ps
docker ps -a
```

---

### Stop Container

```bash
docker stop container_id
```

---

### Remove Container

```bash
docker rm container_id
```

---

### Remove Image

```bash
docker rmi image_id
```

---

## Notes

* Pull images before running if not available locally
* Use -d for background mode
* Use port mapping when exposing services
