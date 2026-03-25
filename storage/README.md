# Shared Storage Setup — Docker Compose

## Overview

This Docker Compose file provisions a **persistent shared storage volume** on the host machine at `/root/storage/shared`. It ensures the directory exists with the correct permissions before any dependent services start.

---

## Directory Structure

```
/root/storage/shared        ← actual folder on the host machine
        ↑
shared-media (named volume) ← Docker volume pointing to the same path
```

---

## Services

### `setup-storage`

A one-time initialization service that:

- Creates the `/root/storage/shared` directory on the host if it doesn't already exist
- Sets permissions to `777` (read/write/execute for all users)
- Exits immediately after setup (`restart: "no"`)

> This service does **not** run continuously. It runs once at startup and stops.

---

## Volumes

### Named Volume: `shared-media`

| Property | Value |
|----------|-------|
| Volume name | `shared-media` |
| Driver | `local` |
| Type | `bind` |
| Host path | `/root/storage/shared` |

This named volume is a **bind mount** — it maps directly to the host folder `/root/storage/shared`. Any service that mounts `shared-media` will read and write to this same folder on the host.

---

## Usage

### 1. Start the setup

```bash
docker compose up setup-storage
```

### 2. Verify the volume was created

```bash
docker volume inspect shared-media
```

Expected output:
```json
[
  {
    "Name": "shared-media",
    "Driver": "local",
    "Mountpoint": "/root/storage/shared",
    "Options": {
      "device": "/root/storage/shared",
      "o": "bind",
      "type": "none"
    }
  }
]
```

### 3. Use the shared volume in other services

Reference the `shared-media` volume as `external: true` in any other compose file that needs access to the shared folder.

```yaml
services:
  my-service:
    image: my-image
    volumes:
      - shared-media:/app/media   # mounts /root/storage/shared into the container

volumes:
  shared-media:
    external: true               # tells Compose the volume already exists
```

---

## Usage Examples

### Example 1 — File upload service sharing files with a web server

An upload service writes files to the shared volume, and Nginx serves them statically.

```yaml
services:
  uploader:
    image: my-upload-app
    volumes:
      - shared-media:/app/uploads   # writes uploaded files here

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - shared-media:/usr/share/nginx/html/media   # serves the same files

volumes:
  shared-media:
    external: true
```

---

### Example 2 — Media processor sharing output with an API

A video processor writes output files to the shared volume, and an API serves them to clients.

```yaml
services:
  processor:
    image: my-video-processor
    volumes:
      - shared-media:/output   # processed files written here

  api:
    image: my-api
    ports:
      - "3000:3000"
    volumes:
      - shared-media:/var/media   # API reads processed files from here

volumes:
  shared-media:
    external: true
```

---

### Example 3 — Backup the shared folder

Run a one-off container to zip and back up the contents of the shared folder.

```bash
docker run --rm \
  -v shared-media:/data \
  -v /root/backups:/backup \
  alpine \
  tar czf /backup/shared-media-$(date +%Y%m%d).tar.gz -C /data .
```

---

### Example 4 — Browse files inside the shared volume

Quickly inspect what's inside the shared volume using an interactive container.

```bash
docker run --rm -it \
  -v shared-media:/data \
  alpine sh
```

Then inside the container:
```sh
ls /data
```

---

### Example 5 — depends_on to ensure setup runs first

Use `depends_on` to make sure `setup-storage` completes before your service starts.

```yaml
services:
  setup-storage:
    image: alpine
    command: sh -c "mkdir -p /root/storage/shared && chmod 777 /root/storage/shared && echo 'Storage ready!'"
    volumes:
      - /root/storage/shared:/root/storage/shared
      - shared-media:/mnt/shared
    restart: "no"

  my-app:
    image: my-app-image
    depends_on:
      - setup-storage       # waits for setup-storage to finish first
    volumes:
      - shared-media:/app/media

volumes:
  shared-media:
    name: shared-media
    driver: local
    driver_opts:
      type: none
      device: /root/storage/shared
      o: bind
```

---

## How It Works

```
docker compose up
       ↓
Compose creates the "shared-media" named volume
       ↓
Named volume is bound to /root/storage/shared on the host
       ↓
setup-storage runs: mkdir + chmod on /root/storage/shared
       ↓
Other services mount "shared-media" → they access /root/storage/shared
```

---

## Notes

- The host path `/root/storage/shared` must be accessible by the Docker daemon
- `chmod 777` grants full permissions — tighten this in production if needed
- The `setup-storage` service should run (or have already run) before any service that depends on this volume
- Data stored in `/root/storage/shared` **persists** across container restarts and removals

---

## Requirements

- Docker Engine
- Docker Compose v2+
