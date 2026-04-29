---
name: paperclip-coolify-deployer
description: "Deploy and self-host Paperclip — the open-source AI agent orchestration dashboard — on Coolify v4. Use this skill immediately whenever the user mentions Paperclip, wants to install or run an AI company/org-chart management tool, deploy an agent orchestration platform, or get Paperclip working on their VPS/self-hosted server. Trigger for any combination of \"Paperclip\" + \"Coolify\" / \"VPS\" / \"Docker\" / \"self-hosted\" / \"deploy\" / \"install\". Also use when the user gets deployment errors like \"Remote branch main not found\" or \"EACCES permission denied\" while deploying Paperclip. This skill provides the complete step-by-step workflow: from Coolify project creation, through the critical `master` branch (not `main`) gotcha, environment variable setup, `/paperclip` persistent volume mount, permission fixes (chown 1000:1000), health check configuration, deployment, and post-deploy onboarding (CEO invite URL)."
---

# Paperclip Coolify Deployer

Deploy Paperclip on a self-hosted Coolify v4 server.

Paperclip is an open-source platform for managing AI agents in an org chart, tracking tasks and budgets. This skill automates or guides every step of deploying it on Coolify — a self-hosted PaaS (Platform as a Service).

## Prerequisites

- A running Coolify v4 server with a reachable Server in its sidebar (green dot)
- Docker / Buildx available on that server
- At least **2 GB RAM, 2 CPU cores, 10 GB disk**
- OpenCode must have access to the **Coolify MCP tools** (`coolify_*`)

## Workflow Overview

The deployment has 8 phases. The agent should execute them sequentially, pausing for user confirmation only where indicated.

1. Discover Coolify infrastructure
2. Create/select a Project and Environment
3. Add Paperclip Application
4. Configure Environment Variables
5. Configure Persistent Storage
6. Preempt known issues (permissions, health check)
7. Deploy
8. Post-deployment onboarding

> **Important:** This workflow is specifically for Paperclip on Coolify v4. Other PaaS (Railway, Heroku) are not covered.

---

## Phase 1: Discover Coolify Infrastructure

First, figure out where everything goes.

1. Ask the user (or infer from context) for:
   - Their Coolify server's **public IP address** (e.g., `YOUR_SERVER_IP`)
   - Whether they use `sslip.io` or a **custom domain**
   - Whether they prefer creating a **new project** or using an **existing one**

2. If the IP is not given, try to find it:
   - Use `coolify_list_servers` to discover servers
   - Use `coolify_server_domains` or `coolify_get_server` to inspect them  

3. Pick the FQDN for Paperclip:
   - Default: `http://paperclip.<SERVER_IP>.sslip.io`
   - If the user has a custom domain, substitute accordingly
   - Ensure the scheme (`http://` vs `https://`) matches what the user actually types in their browser

---

## Phase 2: Create Project & Environment

In Coolify, every app lives in a Project inside an Environment (typically `production`).

### If creating a new project:

1. Call `coolify_projects` with `action: "create"`
   - `name`: `"AI Infrastructure"` (or user preference)
   - Optionally set `description`
2. The result contains `uuid` — save this for later  

### If using an existing project:

1. Call `coolify_projects` with `action: "list"`
2. Ask the user to pick one, or use context to infer which
3. Save its `uuid`

### Then get the environment:

1. Call `coolify_environments` with `action: "list"` and the project UUID
2. Look for the environment named `production`
3. Save its `uuid`

> **Edge case:** If `production` doesn't exist, either use another environment or create one. Ask the user.

---

## Phase 3: Add the Paperclip Application

This is the crucial step with several known failure modes.

### Step 3a: Create Application Resource

Call `coolify_application` with:
- `action: "create"`
- `project_uuid`: (from Phase 2)
- `environment_uuid`: (from Phase 2)
- `name`: `"paperclip"`
- `description`: `"Paperclip AI - Open-source orchestration for zero-human companies"`
- `build_pack`: `"dockerfile"`
- `git_repository`: `"https://github.com/paperclipai/paperclip"`
- `git_branch`: `"master"` — **CRITICAL**: Paperclip uses `master`, not `main`
- `ports_exposes`: `"3100"`
- `server_uuid`: (from Phase 1)

#### Common failure: Wrong branch
If `git_branch` is set to `"main"`, deployment will fail with:
```
fatal: Remote branch main not found in upstream origin
```
If this error appears in logs later, immediately switch branch to `"master"` and redeploy.

### Step 3b: Set FQDN

Call `coolify_application` with `action: "update"` and:
- `fqdn`: The FQDN chosen in Phase 1 (e.g., `"http://paperclip.YOUR_SERVER_IP.sslip.io"`)

### Step 3c: Ensure non-static deployment

Make sure the app is **not** marked as a static site. If the Coolify UI shows a static-site toggle, ensure it is **unchecked**.

---

## Phase 4: Configure Environment Variables

Paperclip requires several environment variables to start correctly.

### Required variables

Call `coolify_env_vars` in batch or individually for these:

| Key | Value | Notes |
|-----|-------|-------|
| `HOST` | `0.0.0.0` | Accept connections from outside the container |
| `PAPERCLIP_HOME` | `/paperclip` | Persistent data folder inside the container |
| `PAPERCLIP_PUBLIC_URL` | `http://paperclip.<IP>.sslip.io` | **Must match FQDN exactly**, including `http://` or `https://` |
| `BETTER_AUTH_SECRET` | `<64-char-hex>` | See below for how to generate |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | `paperclip.<IP>.sslip.io` | **Without** `http://` prefix |

### Generating `BETTER_AUTH_SECRET`

This must be a 64-character hex string. The agent should either:
1. Ask the user to run `openssl rand -hex 32` on the server, or
2. If there is a trusted way to execute commands on the server (e.g., Coolify terminal, local terminal with SSH access), run it and capture the output.

> **Why this matters:** Paperclip uses this secret for authentication token encryption. A weak or missing secret prevents login.

### Optional variable

| Key | Value | Notes |
|-----|-------|-------|
| `PAPERCLIP_TELEMETRY_DISABLED` | `1` | Disables usage telemetry if desired |

After setting all variables, call `coolify_application_logs` or ask the user to verify the Environment Variables tab in Coolify.

---

## Phase 5: Configure Persistent Storage

Without persistent storage, all Paperclip data (companies, agents, tasks) is lost on container restart.

### Using Coolify Storage tab

1. Navigate to the app's Storage (or Volumes) tab
2. Click **"+ Add Volume"**
3. Set **Mount Type** to **"Directory Mount"**
4. Configure:
   - **Source Directory**: auto-filled host path (e.g., `/data/coolify/applications/YOUR_APP_UUID`)
   - **Destination Directory**: `/paperclip`
5. Save

> **Why this host path matters:** Coolify generates a unique UUID folder per application under `/data/coolify/applications/` on the host. Accept the auto-filled path unless the user intentionally customized their Coolify data directory.

---

## Phase 6: Preempt Known Issues

Two issues will almost certainly happen if not handled proactively.

### Issue A: Permission Denied on Volume

**Root cause:** The host directory is owned by `root`, but Paperclip runs inside the container as user `node` (UID `1000`). It tries to write to `/paperclip` and gets `EACCES permission denied`.

**Proactive fix (recommended):**

1. Find the exact host directory for this app's volume from Coolify
2. Execute (or provide to the user):
   ```bash
   sudo chown -R 1000:1000 /data/coolify/applications/YOUR_APP_UUID
   sudo chmod -R u+rwX /data/coolify/applications/YOUR_APP_UUID
   ```

**Reactive fix:** If the first deployment fails with the `EACCES` error, apply the `chown` commands above, redeploy, and it should work.

### Issue B: Health Check Failures

**Root cause:** The container doesn't have `curl`, and Paperclip's startup takes longer than the default Coolify health-check interval.

**Fix:** Disable the health check.

1. In Coolify, go to the app's **Health Check** tab
2. Toggle **"Health Check Enabled"** to **OFF**
3. Save

Alternatively, if the user wants to keep it on, increase:
- Retries: `10`
- Start Period: `60`
- Interval: `10`

> **Recommendation:** For first-time deployers, turn health check **OFF** to avoid the container being rolled back prematurely. Re-enable later once stable.

---

## Phase 7: Deploy

1. Call `coolify_deploy` with the application tag or UUID
2. Wait for deployment logs

### Expected success pattern in logs

```
Building docker image completed.
Rolling update started.
New container started.
Attempt 3 of 10 | Healthcheck status: "healthy"
New container is healthy.
Removing old containers.
Rolling update completed.
```

### If deployment fails

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Remote branch main not found` | Wrong branch | Set branch to `master`, redeploy |
| `EACCES permission denied, mkdir '/paperclip/instances/default/logs'` | Host volume owned by root | Run `chown -R 1000:1000 /data/coolify/applications/UUID` |
| `New container is unhealthy` | Health check timing out | Turn health check OFF in Coolify, redeploy |
| `Build step skipped` | Coolify cached stale image | Force rebuild from deployment settings |

---

## Phase 8: Post-Deployment Onboarding

Paperclip is running, but it's not fully configured until you run two interactive commands inside the container.

### Step 8a: Find the container

Use one of these approaches:

**Option 1 (agent-friendly):**
```bash
docker ps -q --filter "publish=3100"
```

**Option 2 (human-friendly, on the server):**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```
Look for the one with port `3100` exposed. Copy that full name or ID.

### Step 8b: Onboard Paperclip

Run inside the container (replace `CONTAINER_ID` with actual ID or UUID):
```bash
docker exec -it --user node CONTAINER_ID pnpm paperclipai onboard
```

**What happens:**
1. Corepack may ask to download `pnpm` — confirm `Y`
2. Select **"Quickstart"** via arrow keys
3. When asked if you want to start Paperclip now, say **NO** (Coolify already started it)
4. Look for the server info screen — this confirms configuration
5. **Copy the CEO invite URL immediately.** Example:
   ```
   Invite URL: http://paperclip.YOUR_SERVER_IP.sslip.io/invite/pcp_bootstrap_0db2f2d2bc5610d3acaa3f47c58334607cd84d9f90697532
   ```
6. Exit the container (`exit` or `Ctrl+D`)

> **Why "Quickstart"?** It pre-configures an embedded PostgreSQL database and default company setup. Other options (like manual) are for advanced users.

### Step 8c: Bootstrap CEO (if onboard didn't give an invite URL)

If onboarding did **not** output an invite link, run:
```bash
docker exec -it --user node CONTAINER_ID pnpm paperclipai auth bootstrap-ceo
```
Copy the resulting invite URL.

### Step 8d: User opens invite URL

1. Instruct the user to open the invite URL in their browser
2. Fill in **name, email, password**
3. They are now logged in as the **CEO** — top-level admin

### Verification

Once complete, test with:
```bash
curl http://paperclip.<IP>.sslip.io
```
Expected: HTML from Paperclip's frontend.

---

## Quick Reference: All Commands

### Check containers
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### View Paperclip logs
```bash
docker logs $(docker ps -q --filter "publish=3100")
```

### SSH into container as node user
```bash
docker exec -it --user node $(docker ps -q --filter "publish=3100") bash
```

### Restart from Coolify CLI
```bash
coolify restart paperclip
```

---

## Troubleshooting Summary

### "Remote branch main not found"
- **Cause:** Branch set to `main`
- **Fix:** Change to `master`, redeploy

### "EACCES permission denied, mkdir '/paperclip/instances/default/logs'"
- **Cause:** Host volume owned by root, Paperclip runs as UID 1000
- **Fix:** `chown -R 1000:1000 /data/coolify/applications/UUID && chmod -R u+rwX ...`

### "New container is unhealthy"
- **Cause:** Health check timing or missing curl
- **Fix:** Turn Health Check **OFF**, redeploy

### Can't access URL in browser
- **DNS delay:** Wait 2-3 minutes for `sslip.io` to propagate
- **Firewall:** Ensure ports 80/443 are open
- **FQDN mismatch:** Verify `PAPERCLIP_PUBLIC_URL` exactly matches browser URL
- **App crashed:** Check deployment logs in Coolify

### Invite URL doesn't work
- **Cause:** `PAPERCLIP_PUBLIC_URL` or `PAPERCLIP_ALLOWED_HOSTNAMES` mismatched with actual browser URL
- **Fix:** Ensure both match exactly (including `http` vs `https`)

---

## Version & Compatibility

- **Tested on:** Coolify v4 (latest stable / 4.x beta as of 2026-04)
- **Paperclip source:** `https://github.com/paperclipai/paperclip` (branch: `master`)
- **Required ports:** 3100 (container internal), 80/443 (host ingress)
- **Required storage:** `/paperclip` inside container → host directory mount
