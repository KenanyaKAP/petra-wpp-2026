# WPP 2026 — Deployment Guide

This package contains a Docker Compose setup for the **Welcome, Petra Parents 2026** web app.
No source code needed — images are pulled from Docker Hub automatically.

---

## What's Inside

| Service | Image | Default Port | Description |
|---|---|---|---|
| `backend` | `kennyap/kenny-wpp-2026-be` | `3180` | REST API (Go) |
| `frontend-invitation` | `kennyap/kenny-wpp-2026-fe-i` | `3100` | Guest-facing invitation & RSVP page |
| `frontend-admin` | `kennyap/kenny-wpp-2026-fe-a` | `3101` | Admin panel for managing guest list |

---

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with Compose plugin installed
- A domain name with at least 2 subdomains pointed to your server (see [Domain Setup](#domain-setup))
- Gmail account with an [App Password](https://myaccount.google.com/apppasswords) for sending emails (requires 2-Step Verification)

---

## 1. Create the `.env` file

Copy the example and fill in your values:

```bash
cp .env.example .env
```

Then edit `.env`:

```env
# ── Ports ─────────────────────────────────────────────────────
INVITATION_PORT=3100
ADMIN_PORT=3101
BACKEND_PORT=3180
```

> You can leave the ports as-is unless something on your server already uses them.

---

## 2. Environment Variables Reference

### Ports

| Variable | Description |
|---|---|
| `INVITATION_PORT` | Host port the invitation frontend listens on |
| `ADMIN_PORT` | Host port the admin frontend listens on |
| `BACKEND_PORT` | Host port the backend API listens on |

---

### Admin & Auth

| Variable | Description | Example |
|---|---|---|
| `ADMIN_USERNAME` | Login username for the admin panel | `admin` |
| `ADMIN_PASSWORD` | Login password — **change this** | `MySuperSecret123` |
| `JWT_SECRET` | Secret key used to sign login tokens — **minimum 32 chars, change this** | `random_long_string_here_abc123xyz` |

---

### URLs & CORS

| Variable | Description | Example |
|---|---|---|
| `CORS_ORIGINS` | Comma-separated list of frontend origins the API will accept requests from — **must match your actual frontend URLs** | `https://invite.example.com,https://admin.example.com` |
| `INVITATION_BASE_URL` | Public URL of the invitation frontend — used to build invitation links in emails | `https://invite.example.com` |
| `VITE_API_URL` | Public URL of the backend API — used by the browser to call the API | `https://api.example.com` |

> These three work together. See [Domain Setup](#domain-setup) for how to configure them correctly.

---

### SMTP (Email)

| Variable | Description | Example |
|---|---|---|
| `SMTP_HOST` | SMTP server hostname | `smtp.gmail.com` |
| `SMTP_PORT` | SMTP port (`587` for TLS, `465` for SSL) | `587` |
| `SMTP_USER` | Email account used to authenticate | `you@gmail.com` |
| `SMTP_PASS` | App Password (not your Gmail login password) — generate at [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords) | `abcd efgh ijkl mnop` |
| `SMTP_FROM` | The "From" address shown in sent emails | `you@gmail.com` |

---

## 3. Domain Setup

You need **at minimum 2 subdomains** pointing to your server:

| Subdomain | Points to | Purpose |
|---|---|---|
| `invite.example.com` | Your server (port `3100`) | Guest invitation & RSVP page — **must be public** |
| `api.example.com` | Your server (port `3180`) | Backend REST API — **must be public** (browsers call this) |
| `admin.example.com` | Your server (port `3101`) | Admin panel — can be restricted to your IP only |

> The admin panel does **not** need to be public-facing. You can restrict it to your IP via your firewall or reverse proxy if desired.

### Nginx Reverse Proxy example

Each subdomain should proxy to its container port. Example for Nginx:

```nginx
# invite.example.com
server {
    listen 80;
    server_name invite.example.com;
    location / {
        proxy_pass http://localhost:3100;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# api.example.com
server {
    listen 80;
    server_name api.example.com;
    location / {
        proxy_pass http://localhost:3180;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# admin.example.com
server {
    listen 80;
    server_name admin.example.com;
    location / {
        proxy_pass http://localhost:3101;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Use [Certbot](https://certbot.eff.org/) to add HTTPS (`certbot --nginx`).

### CORS configuration

Once your domains are set, update your `.env` so the three URL variables match:

```env
# All frontend origins the API is allowed to receive requests from
CORS_ORIGINS=https://invite.example.com,https://admin.example.com

# Used to build the invitation link in emails sent to guests
INVITATION_BASE_URL=https://invite.example.com

# Used by the browser to call the backend API
VITE_API_URL=https://api.example.com
```

> If `CORS_ORIGINS` does not include your frontend's exact origin (protocol + domain), the browser will block API calls with a CORS error.

---

## 4. Start the App

```bash
# Create the data directory for the database
mkdir -p data

# Pull latest images and start
docker compose pull
docker compose up -d
```

Check that everything is running:

```bash
docker compose ps
docker compose logs -f
```

---

## 5. Updating

```bash
docker compose pull
docker compose up -d
```

---

## 6. Stopping

```bash
docker compose down
```

The `data/` folder (SQLite database) is kept on your host so data persists across restarts and updates.
