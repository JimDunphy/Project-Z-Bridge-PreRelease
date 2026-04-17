# Deployment / Packaging

This repo includes a development-focused Docker setup (`docker-compose.yml`) that mounts the whole workspace and runs `cargo run`.

For admins who want to **try Project Z‑Bridge without building**, use the **runtime image** + **runtime compose**.

For public-Internet deployments behind nginx, also read: `docs/sysadmin/SYSADMIN_PRODUCTION_HARDENING.md`.

## Runtime prerequisites

- Docker + Docker Compose plugin
- A copy of the classic ZWC `zimbra.war` (not distributed here)
- Stalwart JMAP endpoint URL + user credentials

## Obtaining `zimbra.war` (classic ZWC assets)

Project Z‑Bridge intentionally does **not** redistribute ZWC assets. Administrators supply `zimbra.war` locally.

Common ways to obtain it:

- **From an existing Zimbra install/build** (where you already have the classic web client WAR).
- **Download the supported release artifact** (recommended):
  - `./manage.sh webclient-build`
  - This downloads the latest supported `zimbra.war` from `JimDunphy/DockerZimbraRHEL8` GitHub Releases into the repo root.
- **Build your own WAR manually** if you want a different/custom build:
  - Repo: `https://github.com/JimDunphy/DockerZimbraRHEL8`
  - Command: `./docker.sh --build-war 10.1`
  - Result: produces a classic `zimbra.war` artifact for the current 10.1 line using a containerized build environment.

Convenience wrapper (optional):

- `./manage.sh webclient-build`
  - Downloads the latest supported `zimbra.war` release asset into the repo root.
- `./manage.sh webclient-build --version 10.1.16`
  - Downloads that exact release instead of the latest.

After you have the WAR at `./zimbra.war`, extract it:

- `./manage.sh webclient-extract`

If you are swapping to a newer ZWC version and something regresses, see `SYSADMIN_WEBCLIENT_EXTRACT.md` for the known failure modes and first checks.

## Quick start (runtime)

1) Build the runtime image (one-time on the machine doing the packaging):

- `./manage.sh image-build`

2) Extract `zimbra.war` into `static/zimbra/`:

- `./manage.sh webclient-extract /path/to/zimbra.war`

3) Copy `.env.example` → `.env` and set at least:

- `STALWART_BASE_URL`

4) Start using the runtime compose:

- `./manage.sh up-runtime`

Open `http://127.0.0.1:${BRIDGE_PORT:-7777}/zimbra/`.

Note: Today the ZWC UI is mounted at `/zimbra/` for maximum compatibility. A future enhancement will allow configuring this mount point (example: `/mail/`) via an env var; see `docs/developer/ARCHITECTURE_CONTEXT_PATH_PLAN.md`.

## Nginx reverse proxy (recommended for production)

In production, the shim is typically bound to loopback and exposed via an HTTPS reverse proxy.

Key requirements:

- Proxy **all** of these paths to the shim:
  - `/zimbra/` (ZWC UI)
  - `/service/` (SOAP + image proxy + misc endpoints)
  - `/home/` and `/service/home/` (attachments/briefcase)
  - (Future) If/when the ZWC context path becomes configurable, proxy `/<context>/` instead of `/zimbra/` (see `docs/developer/ARCHITECTURE_CONTEXT_PATH_PLAN.md`).
- Forward origin information so the shim can generate correct absolute URLs (for example `dfbackground` URLs used by ZWC):
  - `Host`
  - `X-Forwarded-Proto`
- Alternatively (and often simplest), set `BRIDGE_PUBLIC_BASE_URL=https://mail.example.com/` in `.env`.

Example nginx `server {}` block:

```nginx
server {
  listen 443 ssl http2;
  server_name mail.example.com;

  # TODO: configure ssl_certificate / ssl_certificate_key

  # Large attachments / exports
  client_max_body_size 200m;

  # Shim runs on localhost
  set $zbridge http://127.0.0.1:7777;

  location /zimbra/ {
    proxy_pass $zbridge;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location /service/ {
    proxy_pass $zbridge;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location /home/ {
    proxy_pass $zbridge;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location /service/home/ {
    proxy_pass $zbridge;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

Notes:

- If you only proxy `/zimbra/`, features like attachments and the remote-image proxy (`/service/image-proxy/...`) will not work.
- The shim sets `ZM_AUTH_TOKEN` with `HttpOnly; SameSite=Lax`. When serving over HTTPS, ensure cookies are treated as secure:
  - Preferred: add shim support to set `Secure` (future); OR
  - Use nginx cookie rewriting (varies by nginx version/modules).

## Keeping the bridge running (Docker restarts)

If the Docker daemon restarts (host reboot, package upgrades, manual `systemctl restart docker`, etc), containers without a restart policy can remain stopped and the UI will fail to connect (you’ll see the bridge exit with code `143`, i.e. SIGTERM).

This repo sets a restart policy in both compose files:

- `restart: unless-stopped`

So `project-z-bridge-bridge` comes back automatically after Docker restarts.

Note: restart policies only apply after the container is (re)created. If you already have a running bridge container from before this was added, recreate it once:

- `docker compose down && docker compose up -d --build`

## What happens to a logged-in browser tab if the bridge restarts?

Today, bridge sessions are stored **in memory** inside the shim process (`state.sessions`). The `ZM_AUTH_TOKEN` cookie is just a **bridge session id** that maps to that in-memory session.

Implication:

- If the bridge restarts (container restart, `cargo run` restart, host reboot), the in-memory sessions are lost.
- Any already-open ZWC tab will start getting `service.AUTH_REQUIRED` faults on `/service/soap` calls because its cookie no longer maps to a live session.

Expected user experience (current):

- If it’s a **transient network glitch**, the shim injects a small ZWC patch that:
  - silently retries for ~5 seconds
  - shows a **non-blocking banner** from ~5–30 seconds
  - shows a **blocking modal** after ~30 seconds with a “Retry Now” button
  - clears the banner/modal immediately on recovery
- If it’s a **bridge restart** (stale `ZM_AUTH_TOKEN` cookie), the shim returns `service.AUTH_REQUIRED` and **expires** `ZM_AUTH_TOKEN`, causing ZWC’s native session-expired handling to show the login UI (no manual refresh needed, but you must log in again).

Future improvement options (not implemented):

- Persist sessions in a shared store (Redis/SQL/shared volume) so restarts don’t force re-login.
- Use a stateless cookie scheme (signed/encrypted session payload) so any replica can resume a session after restart.

## Packaging for copy-to-server deployments

Two options:

### Source bundle (admins build locally)

- `./manage.sh package-src`

This creates a tarball under `dist/` that excludes local/dev artifacts (`.env`, `.dev/`, logs, and LLM scratch dirs).

### Runtime bundle (admins do not build)

- `./manage.sh package-runtime`

This creates a tarball under `dist/` containing:

- `docker-compose.runtime.yml`
- `.env.example`
- `manage.sh`
- a saved Docker image tarball (`docker load` compatible)

On the target server:

- unpack the tarball
- `docker load < dist/*.image.tar.gz`
- extract `zimbra.war` into `static/zimbra/`
- `./manage.sh up-runtime`

## Notes

- The runtime compose uses a named volume `bridge_data` for `BRIDGE_DATA_DIR=/data`.
- The ZWC webclient assets are mounted read-only from `./static/zimbra` to `/webclient`.
