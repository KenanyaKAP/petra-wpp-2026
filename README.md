# WPP 2026 — Deployment Guide

Docker Compose setup for **Welcome, Petra Parents 2026**. Images are pulled from Docker Hub — no source code needed.

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with Compose plugin
- A domain with at least 2 subdomains pointed to your server
- An SMTP server for sending invitation emails

---

## 1. Create the `.env` file

```bash
cp .env.example .env
```

Then fill in your values. See the reference below.

---

## 2. Environment Variables

### Ports

| Variable | Description |
|---|---|
| `INVITATION_PORT` | Host port for the invitation frontend |
| `ADMIN_PORT` | Host port for the admin panel |
| `BACKEND_PORT` | Host port for the backend API |

> Leave these as-is unless the ports are already in use on your server.

---

### Admin & Auth

| Variable | Description |
|---|---|
| `ADMIN_USERNAME` | Login username for the admin panel |
| `ADMIN_PASSWORD` | Login password — **change this** |
| `JWT_SECRET` | Secret key for login tokens — **minimum 32 chars, change this** |

---

### URLs & CORS

These three variables must be consistent with your domain setup.

| Variable | Description |
|---|---|
| `CORS_ORIGINS` | Comma-separated list of your frontend URLs — the API will only accept requests from these origins |
| `INVITATION_BASE_URL` | Public URL of the invitation frontend — used to build links in invitation emails |
| `VITE_API_URL` | Public URL of the backend API — used by the browser to call the API |

**Example:**
```env
CORS_ORIGINS=https://invite.example.com,https://admin.example.com
INVITATION_BASE_URL=https://invite.example.com
VITE_API_URL=https://api.example.com
```

---

### SMTP (Email)

| Variable | Description |
|---|---|
| `SMTP_HOST` | SMTP server hostname (e.g. `smtp.gmail.com`) |
| `SMTP_PORT` | SMTP port — `587` for TLS, `465` for SSL |
| `SMTP_USER` | Email account username |
| `SMTP_PASS` | Email account password or app password |
| `SMTP_FROM` | The "From" address shown in sent emails |

---

## 3. Domain Setup

You need **at least 2 subdomains** forwarded to your server:

| Subdomain | Port | Notes |
|---|---|---|
| `invite.example.com` | `3100` | Must be public — guests open this |
| `api.example.com` | `3180` | Must be public — browsers call this directly |
| `admin.example.com` | `3101` | Optional to expose publicly |

Set up a reverse proxy (e.g. Nginx) for each subdomain and use [Certbot](https://certbot.eff.org/) to enable HTTPS.

> If `CORS_ORIGINS` doesn't exactly match your frontend URLs (including `https://`), the browser will block API calls.

---

## 4. Start

```bash
mkdir -p data
docker compose pull
docker compose up -d
```

---

## 5. Update

```bash
docker compose pull
docker compose up -d
```

---

## 6. Stop

```bash
docker compose down
```

The `data/` folder (SQLite database) persists on your host across restarts and updates.

