# Bellamy Book — Self-Host

**Run your own social network** with Docker. No source code required — use pre-built images, set your domain and secrets in `.env`, and go live.

---

## What is Bellamy Book?

**Bellamy Book** is a modern social network platform: posts, stories, messaging, video calls, blogs, search, and an admin panel. This repository is the **self-host kit**: everything you need to deploy it on your own server using Docker and Docker Compose.

| Link | Description |
|------|-------------|
| [**Landing**](https://bellamybook.com/landing) | Product overview and features |
| [**Demo**](https://bellamybook.com) | Try the app live |
| [**Docs**](https://docs.bellamybook.com) | Full documentation: installation, configuration, R2, SMTP, Turnstile, LiveKit, Google Login, and more |
| [**Docker Hub**](https://hub.docker.com/u/bellamy31) | Pre-built images (`bellamy31/bellamybook-*`) for self-hosting |
| [**Support the project**](https://buymeacoffee.com/nmtri31082x) | Buy Me a Coffee — support development |

---

## Why self-host?

- **Your data, your server** — Full control over users and content.
- **Your domain** — Run at `app.yourdomain.com` with your branding.
- **Optional integrations** — MinIO or Cloudflare R2 for storage; SMTP, Turnstile, LiveKit, Google Login as you need.
- **One compose, one `.env`** — No build step; pull images, configure, run.

---

## Quick start (self-hosters)

### 1. Get this repo

Clone or download this folder (the `dockerPublish` bundle). You need: `docker-compose.yml`, `.env.example`, and the config directories (`traefik/`, `primary/`, `replica/`, `scripts/`, `opensearch-config/`).

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and set at least:

- **Docker images:** `DOCKER_REGISTRY=bellamy31`, `IMAGE_TAG=latest` (defaults; change if you use another publisher or tag).
- **Your domain:** `API_PUBLIC_URL`, `FRONTEND_PUBLIC_URL`, `ADMIN_PUBLIC_URL`, and all `TRAEFIK_*_HOST` to your hostnames.
- **Secrets:** Replace every `CHANGE_ME_*` — Postgres, Redis, MongoDB, Neo4j, RabbitMQ, MinIO, and **JWT** (`JwtSettings__Secret`, e.g. `openssl rand -base64 64`).

See [Environment Configuration](https://docs.bellamybook.com/docs/self-host/configuration/environment) and the [configuration checklist](https://docs.bellamybook.com/docs/self-host/configuration/environment#configuration-checklist) in the docs for full details.

### 3. Create MongoDB keyfile (required)

The stack uses a **MongoDB replica set** for the app and workers. MongoDB requires a **keyfile** for replica set authentication. You must create this file **before** your first `docker compose up`; the `mongo-keyfile-init` service copies it into a volume used by MongoDB.

**Where:** In the same directory as `docker-compose.yml` (your dockerPublish folder).

**Commands (run once):**

```bash
cd dockerPublish
openssl rand -base64 756 > mongo-keyfile
chmod 600 mongo-keyfile
```

- **Name:** The file must be named exactly `mongo-keyfile` (compose mounts it as `./mongo-keyfile`).
- **Permissions:** `chmod 600` (read/write for owner only). Some setups use `chmod 400`; either is accepted by MongoDB.
- **Do not** commit `mongo-keyfile` to version control or share it; treat it as a secret.

If you skip this step, `mongo-keyfile-init` will fail with "Source keyfile /tmp/keyfile not found" and MongoDB will not start.

### 4. Run

```bash
docker compose pull
docker compose up -d
docker compose run --rm db-migration
```

### 5. Access

- **With Traefik:** Use the hostnames in your `.env` (e.g. `https://app.yourdomain.com`). Point DNS to this server.
- **Without Traefik:** Frontend → http://localhost:8081, Admin → http://localhost:8084, API → http://localhost:5000.

Full step-by-step and optional services (R2, SMTP, Turnstile, Google Login, LiveKit, MinIO policies): **[Self-Host with Pre-Built Images](https://docs.bellamybook.com/docs/self-host/installation/docker-publish)**.

---

## Requirements

- **Docker** and **Docker Compose**
- **16GB+ RAM**, **8+ CPU** recommended for the full stack
- **Domain** (optional; you can test with localhost and ports)

---

## Configuration overview

| What | Docs |
|------|------|
| Environment variables, JWT, databases, storage | [Environment](https://docs.bellamybook.com/docs/self-host/configuration/environment) |
| JWT secret (required for login) | [JWT](https://docs.bellamybook.com/docs/self-host/configuration/jwt) |
| MinIO (default) or Cloudflare R2 | [Storage](https://docs.bellamybook.com/docs/self-host/configuration/storage), [R2 Setup](https://docs.bellamybook.com/docs/self-host/configuration/r2-setup) |
| MinIO bucket policies (security) | [Storage — MinIO security](https://docs.bellamybook.com/docs/self-host/configuration/storage#bucket-policies-security) |
| Mail (password reset, notifications) | [SMTP](https://docs.bellamybook.com/docs/self-host/configuration/smtp) |
| CAPTCHA on login/register | [Turnstile](https://docs.bellamybook.com/docs/self-host/configuration/turnstile) |
| Sign in with Google | [Google OAuth](https://docs.bellamybook.com/docs/self-host/configuration/google-oauth) |
| Voice and video calls | [LiveKit](https://docs.bellamybook.com/docs/self-host/configuration/livekit) |

---

## Updating

Set `IMAGE_TAG` in `.env` to the new tag (e.g. `v1.0.1`), then:

```bash
docker compose pull
docker compose up -d
```

---

## For publishers (building and pushing images)

If you maintain and publish the Docker images (e.g. to Docker Hub):

1. **Build and push** — See **[BUILD_AND_PUBLISH_DOCKERHUB.md](../tools/publishdocker/BUILD_AND_PUBLISH_DOCKERHUB.md)** (in `tools/publishdocker`). From the repo root:
   ```bash
   ./tools/publishdocker/publish-images.sh bellamy31 latest
   ```
   Or: `DOCKER_REGISTRY=youruser IMAGE_TAG=v1.0.0 ./tools/publishdocker/publish-images.sh`

2. **Image naming** — `.env.example` uses `DOCKER_REGISTRY=bellamy31` and `IMAGE_TAG=latest` by default. All services use `image: ${DOCKER_REGISTRY}/bellamybook-<service>:${IMAGE_TAG}`.

3. **Runtime config** — Frontend and admin read `API_PUBLIC_URL`, `FRONTEND_PUBLIC_URL`, `ADMIN_PUBLIC_URL`, `Minio__PublicUrl` from the user’s `.env` at container start. One image set works for any domain.

4. **What to distribute** — This folder only: `docker-compose.yml`, `.env.example`, README, config dirs. No source code or CI.

---

## Image reference

Images follow `${DOCKER_REGISTRY}/bellamybook-<service>:${IMAGE_TAG}`. Default registry: `bellamy31`, tag: `latest`.

| Service | Image |
|---------|--------|
| db-migration | bellamybook-db-migration |
| api | bellamybook-api |
| frontend | bellamybook-frontend |
| admin | bellamybook-admin |
| websocket-worker | bellamybook-websocket-worker |
| chat-worker | bellamybook-chat-worker |
| interaction-worker | bellamybook-interaction-worker |
| scoring-worker | bellamybook-scoring-worker |
| trending-worker | bellamybook-trending-worker |
| graph-worker | bellamybook-graph-worker |
| hashtag-worker | bellamybook-hashtag-worker |
| elasticsearch-sync-worker | bellamybook-elasticsearch-sync-worker |
| media-processing-worker | bellamybook-media-processing-worker |
| expo-push-worker | bellamybook-expo-push-worker |
| webpush-worker | bellamybook-webpush-worker |
| blog-autogen-worker | bellamybook-blog-autogen-worker |

Infrastructure (PostgreSQL, Redis, MongoDB, Kafka, etc.) uses standard public images defined in `docker-compose.yml`.

---

## Links

- [Landing](https://bellamybook.com/landing) · [Demo](https://bellamybook.com) · [Documentation](https://docs.bellamybook.com) · [Docker Hub](https://hub.docker.com/u/bellamy31) · [Support (Buy Me a Coffee)](https://buymeacoffee.com/nmtri31082x)
