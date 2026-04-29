# How to Install Paperclip on Coolify: A Complete Beginner's Guide

> **Date:** April 2026  
> **Difficulty:** Beginner  
> **Time:** ~15–20 minutes  
> **Goal:** Deploy [Paperclip](https://github.com/paperclipai/paperclip) — an open-source AI orchestration platform — on your own server using [Coolify](https://coolify.io/).

---

## What You'll Build

By the end of this guide, you'll have:
- **Paperclip** running in a Docker container on your server
- A public URL to access it (`http://paperclip.YOUR_SERVER_IP.sslip.io`)
- Persistent storage so your data survives restarts
- A fully configured instance ready to onboard your first "CEO" agent

**Coolify** is a self-hosted PaaS (Platform as a Service) that makes deploying Docker apps as easy as clicking buttons. Think of it like a free, open-source alternative to Heroku or Railway — but running on your own server.

**Paperclip** is an open-source "orchestration platform for zero-human companies." It lets you manage AI agents in an org chart, assign tasks, track budgets, and monitor work — all from a dashboard.

---

## What You Need Before Starting

### 1. A Server with Coolify Installed
- Ubuntu Server (20.04 or 24.04 recommended)
- Docker 24+ installed
- Coolify installed and running
- At least **2 GB RAM**, **2 CPU cores**, and **10 GB disk space** free (Paperclip + builds take space)
- Root or sudo SSH access to the server

### 2. Basic Info You Should Have Handy
- Your server's **public IP address** (e.g., `YOUR_SERVER_IP`)
- Access to your Coolify dashboard URL (usually `https://coolify.YOURDOMAIN.com` or `http://YOUR_SERVER_IP`)
- An SSH client (Terminal on Mac/Linux, PuTTY or Windows Terminal on Windows)

> **Tip:** If this is your first time accessing Coolify, log in and make sure you can see the dashboard before proceeding.

---

## Overview of the Steps

Here's the bird's-eye view of what we'll do:

1. [Check your Coolify infrastructure](#step-1-check-your-coolify-infrastructure)
2. [Create a project](#step-2-create-a-project-in-coolify)
3. [Add the Paperclip app from GitHub](#step-3-add-the-paperclip-application)
4. [Configure environment variables](#step-4-set-environment-variables)
5. [Configure storage](#step-5-add-persistent-storage)
6. [Fix the deployment issues](#step-6-fix-common-deployment-issues)
7. [Deploy!](#step-7-deploy)
8. [Post-deployment setup](#step-8-post-deployment-setup)

Let's go through each one in detail.

---

## Step 1: Check Your Coolify Infrastructure

Before we start, let's make sure your Coolify server is healthy and ready.

### What to Check
1. Log into your Coolify dashboard
2. Click on **Servers** in the sidebar
3. Confirm your server shows as **"Reachable"** (green dot)
4. Check that your server's proxy (usually Traefik) is running

> **Common Issue:** If your server shows as "Unreachable," SSH into it and run:
> ```bash
> systemctl status coolify
> ```
> If it's stopped, start it with:
> ```bash
> systemctl start coolify
> ```

---

## Step 2: Create a Project in Coolify

A "Project" in Coolify is like a folder that groups related apps together.

1. In the Coolify dashboard, click **Projects** in the left sidebar
2. Click the **"+ New Project"** button (usually in the top right)
3. Name your project: `AI Infrastructure` (or whatever you prefer, e.g., `Agency Stack`)
4. (Optional) Add a description like: *"AI orchestration and agent management"*
5. Click **Create**

6. Inside your new project, you'll see one environment already created, typically called **production**. Click on it.

> **Important Note:** You'll be working inside the `production` environment from now on. Every setting we configure goes here.

---

## Step 3: Add the Paperclip Application

Now we'll pull the Paperclip code from GitHub and tell Coolify to build a Docker container from it.

1. Inside your project's `production` environment, click **"+ Add New Resource"**
2. Select **"Application"**
3. Choose source: **"GitHub App"** → **"Public GitHub"**

### Source Configuration
Fill in these fields exactly:

| Field | Value |
|-------|-------|
| **Repository** | `https://github.com/paperclipai/paperclip` |
| **Branch** | `master` ⚠️ *(Not `main`! Paperclip uses `master` as the default branch)* |
| **Build Pack** | `Dockerfile` |
| **Server** | Select your server from the dropdown |

> **⚠️ Critical:** The branch must be `master`. If you select `main`, the deployment will fail with the error: `fatal: Remote branch main not found in upstream origin`.

4. Click **Continue**

### General Settings
On the next screen, configure the basics:

| Field | Value |
|-------|-------|
| **Name** | `paperclip` |
| **Description** | `Paperclip AI - Open-source orchestration for zero-human companies` |
| **Port Exposes** | `3100` *(This is the port Paperclip listens on inside the container)* |

5. Scroll down to **FQDN / Domains**
6. Set it to: `http://paperclip.YOUR_SERVER_IP.sslip.io`
   - Replace `YOUR_SERVER_IP` with your actual server IP (e.g., `YOUR_SERVER_IP`)
   - Example: `http://paperclip.YOUR_SERVER_IP.sslip.io`

> **What is sslip.io?** It's a free DNS service. It lets you use a subdomain that points to your IP without needing to buy a real domain name. `paperclip.YOUR_SERVER_IP.sslip.io` automatically resolves to `YOUR_SERVER_IP`.

7. Optionally enable **Is it a static site?** — **NO** (leave unchecked)
8. Click **Continue**

---

## Step 4: Set Environment Variables

Environment variables are configuration settings that the app reads when it starts. Paperclip needs several to know where it's running and how to secure itself.

1. Go to the **"Environment Variables"** tab in your app settings
2. Add each variable one by one by clicking **"+ Add"**

Here are the exact variables you need:

### Required Environment Variables

| Key | Value | Explanation |
|-----|-------|-------------|
| `HOST` | `0.0.0.0` | Tells the server to accept connections from anywhere (not just localhost) |
| `PAPERCLIP_HOME` | `/paperclip` | The folder inside the container where Paperclip stores its data |
| `PAPERCLIP_PUBLIC_URL` | `http://paperclip.YOUR_SERVER_IP.sslip.io` | The URL you will access Paperclip from. **Must match your FQDN exactly.** |
| `BETTER_AUTH_SECRET` | *(see below)* | A long random secret key for encrypting authentication tokens |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | `paperclip.YOUR_SERVER_IP.sslip.io` | Security setting — which domains are allowed to access Paperclip. Must match your FQDN without `http://`. |

> **Important:** Replace `YOUR_SERVER_IP` in both `PAPERCLIP_PUBLIC_URL` and `PAPERCLIP_ALLOWED_HOSTNAMES` with your actual IP.

### Generating `BETTER_AUTH_SECRET`

You cannot type anything random here — it must be a 64-character hex string. Here's how to generate it:

1. **SSH into your server:**
   ```bash
   ssh root@YOUR_SERVER_IP
   ```

2. **Run this command:**
   ```bash
   openssl rand -hex 32
   ```

3. **Copy the output.** It will look something like:
   ```
   f47ac10b58cc4372a5670e02b2c3d479e8f8e5a6b7c8d9e0f1a2b3c4d5e6f7a8
   ```

4. **Paste this value** as the `BETTER_AUTH_SECRET` in Coolify

5. (Optional) Add this variable if you don't want to send usage telemetry:

| Key | Value |
|-----|-------|
| `PAPERCLIP_TELEMETRY_DISABLED` | `1` |

6. Click **Save**

---

## Step 5: Add Persistent Storage

By default, when a Docker container restarts, everything inside it is wiped. We need to save Paperclip's data (your companies, agents, tasks, etc.) to a folder on the host server that survives restarts.

1. Go to the **"Storage"** or **"Volumes"** tab in your app settings
2. Click **"+ Add Volume"**
3. For **Mount Type**, select: **Directory Mount**

### Volume Configuration

| Field | Value |
|-------|-------|
| **Source Directory** | Leave it on the auto-filled path, something like `/data/coolify/applications/YOUR_APP_UUID` |
| **Destination Directory** | `/paperclip` *(This is the path inside the container)* |

> **Important:** Make sure **Destination Directory** is `/paperclip`, not `/etc/nginx` or anything else. This is where Paperclip expects its home folder to be.

4. Click **Add**

> **What happens here?** The container folder `/paperclip` is "linked" to a real folder on your server's hard drive. Even if you delete and recreate the container, the data stays safe.

---

## Step 6: Fix Common Deployment Issues

Before we hit deploy, let's fix two common issues that are guaranteed to happen.

### Issue 1: Docker Volume Permissions

When Coolify creates the directory on the host, it's owned by the `root` user. But inside the container, Paperclip runs as the `node` user (UID 1000). When Paperclip tries to write to `/paperclip`, it gets a `Permission Denied` error.

**Error you'll see if not fixed:**
```
Error: EACCES: permission denied, mkdir '/paperclip/instances/default/logs'
```

**Fix (do this BEFORE or AFTER first failed deploy):**

1. **SSH into your server:**
   ```bash
   ssh root@YOUR_SERVER_IP
   ```

2. **Find your app's host directory (it's listed in the Storage tab)** — typically something like:
   ```
   /data/coolify/applications/YOUR_APP_UUID
   ```

3. **Change ownership to user 1000 (the `node` user inside the container):**
   ```bash
   chown -R 1000:1000 /data/coolify/applications/YOUR_APP_UUID
   chmod -R u+rwX /data/coolify/applications/YOUR_APP_UUID
   ```

   Replace `YOUR_APP_UUID` with the actual folder name.

> **Note:** You only need to do this once, right after adding the volume. After the first successful start, Paperclip owns the folder.

### Issue 2: Health Check Failures

Coolify's built-in health check tries to ping the app with `curl` or `wget` to verify it's running. Some Docker images don't include `curl`, and Paperclip's startup takes longer than the default timeout. This causes the container to be marked as unhealthy and gets rolled back.

**Fix:**
1. Go to the **"Health Check"** tab in your app settings
2. Toggle **"Health Check Enabled"** to **OFF**
   - Alternatively, you can keep it on but increase:
     - **Retries** to `10`
     - **Start Period** to `60`
     - **Interval** to `10`

3. Click **Save**

> **Which should I choose?** If this is your first time deploying, just **turn the health check OFF** to avoid headaches. Once the app is stable and running, you can turn it back on if you want.

---

## Step 7: Deploy

Finally — the moment of truth!

1. Click the **"Deploy"** button in the top right of your app's page
2. Coolify will now:
   - Clone the Paperclip repository from GitHub
   - Build the Docker image (this takes **3–5 minutes** — it's a large Node.js monorepo)
   - Start the container
   - Run the application

3. Watch the **Deployment Logs** panel at the bottom of the screen

### What You Should See (Success)

```
Docker 29.3.1 with BuildKit and Buildx detected on deployment server (localhost).
Starting deployment of paperclipai/paperclip:master to localhost.
Building docker image started.
... (build logs) ...
Building docker image completed.
Rolling update started.
New container started.
Attempt 3 of 10 | Healthcheck status: "healthy"
New container is healthy.
Removing old containers.
Rolling update completed.
```

If you see **"New container is healthy"** — congratulations, it's live! 🎉

### What the URL Looks Like
Open your browser and go to:
```
http://paperclip.YOUR_SERVER_IP.sslip.io
```

You should see the Paperclip login/onboarding page.

---

## Step 8: Post-Deployment Setup

The app is running, but Paperclip isn't fully set up yet. You need to run two interactive commands inside the running container.

> **⚠️ Important:** Coolify names containers using the app UUID, not the app name. The container won't be called `paperclip` — it will have a name like `YOUR_APP_UUID-202945090592`. Follow the steps below carefully.

### How to Find Your Container Name

Run this command on your server to see all running containers:

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Look for the one that has **port 3100** exposed. It will look something like:

```
NAMES                                   STATUS                   PORTS
YOUR_APP_UUID-202945090592   Up 9 minutes (healthy)   3100/tcp
```

Copy that **full name** — that's your container.

> **Tip:** Alternatively, you can use this shorter command to get just the Paperclip container name automatically:
> ```bash
> docker ps -q --filter "publish=3100"
> ```
> This outputs the container ID, which you can use instead of the name.

### Command 1: Onboard Paperclip

Using the **container name** you copied above:

```bash
docker exec -it --user node YOUR_APP_UUID-202945090592 pnpm paperclipai onboard
```

**Or**, using the automatic container ID method (copy-paste friendly):

```bash
docker exec -it --user node $(docker ps -q --filter "publish=3100") pnpm paperclipai onboard
```

**What happens:**

1. **First, you may see this prompt:**
   ```
   ! Corepack is about to download https://registry.npmjs.org/pnpm/-/pnpm-9.15.4.tgz
   ? Do you want to continue? [Y/n]
   ```
   - Type **Y** and press Enter. This is normal — it's just downloading the `pnpm` package manager.

2. **The onboard wizard starts.** Select the **"Quickstart"** option (use arrow keys and Enter).

3. **When it asks if you want to start Paperclip now:**
   - If you're using this guide, you already deployed Paperclip via Coolify, so the server is already running.
   - Say **NO** if asked to start it again.

4. **The server info screen appears** — this confirms everything is configured correctly. Your output should look similar to this:

   ```
   Mode             embedded-postgres  |  vite-dev-middleware
   Deploy           authenticated (private)
   Bind             lan (0.0.0.0)
   Auth             ready
   Server           3101 (requested 3100)
   API              http://localhost:3101/api (health: http://localhost:3101/api/health)
   UI               http://localhost:3101
   Database         /paperclip/instances/default/db (pg:54329)
   Migrations       already applied
   Agent JWT        set
   Heartbeat        enabled (30000ms)
   ```

   > **Note:** You may see `Server 3101 (requested 3100)` — this is fine! If port 3100 is busy, Paperclip automatically picks the next free port (3101). Coolify's proxy routes traffic correctly regardless.

> **⚠️ HTTPS Users:** If you're using HTTPS (e.g., with a real domain and SSL certificate), make sure `PAPERCLIP_PUBLIC_URL` starts with `https://`, not `http://`. The invite URL must match what you type in your browser exactly — if there's a mismatch between `http` and `https`, the invite link will not work.

5. **A CEO invite URL is generated!** Look for a line like:

   ```
   Invite URL: http://paperclip.YOUR_SERVER_IP.sslip.io/invite/pcp_bootstrap_0db2f2d2bc5610d3acaa3f47c58334607cd84d9f90697532
   ```

   **Copy this invite URL immediately!** This is your one-time link to create the first admin account.

   > **Important:** If the onboard command already generated the invite for you (as in the output above), **you may not need to run the `auth bootstrap-ceo` command separately.** Skip to [Step 3 below](#step-3-create-your-ceo-account) and open the invite URL in your browser.

6. Type `exit` or press `Ctrl+D` to leave the container.

---

### Command 2: Create the First Admin (CEO)

**Only run this if the `onboard` command above did NOT generate an invite URL.**

Using the **container name**:

```bash
docker exec -it --user node YOUR_APP_UUID-202945090592 pnpm paperclipai auth bootstrap-ceo
```

**Or**, using the automatic method:

```bash
docker exec -it --user node $(docker ps -q --filter "publish=3100") pnpm paperclipai auth bootstrap-ceo
```

This will output an invite URL like:
```
Invite URL: http://paperclip.YOUR_SERVER_IP.sslip.io/invite/pcp_bootstrap_...
```

**Copy this URL** and open it in your browser.

---

### Step 3: Create Your CEO Account

1. Open the **invite URL** in your web browser
2. Fill in your **name, email, and password**
3. You are now logged in as the **CEO** — the top-level admin of your Paperclip instance
4. You can now create companies, hire agents, assign tasks, and manage everything from the dashboard

---

## What a Successful Setup Looks Like

When everything is working, you should see output similar to this in the logs:

```
Starting Paperclip server...
WARN: Embedded PostgreSQL already running; reusing existing process (pid=73, port=54329)
INFO: Embedded PostgreSQL ready
INFO: Authenticated mode auth origin configuration
  ───────────────────────────────────────────────────────
  Mode       embedded-postgres  |  vite-dev-middleware
  Bind       lan (0.0.0.0)
  Auth       ready
  API        http://localhost:3101/api (health: http://localhost:3101/api/health)
  Database   /paperclip/instances/default/db (pg:54329)
  Migrations already applied
  Agent JWT  set
  Heartbeat  enabled (30000ms)
  DB Backup  enabled (every 60m, keep 30d)
  Config     /paperclip/instances/default/config.json
  ───────────────────────────────────────────────────────
  INFO: Server listening on 0.0.0.0:3101
  INFO: Opened browser at http://127.0.0.1:3101
```

If you see the `Server listening` message, your Paperclip instance is fully live and accessible at your URL! 🎉

---

## You're Done! What Now?

After opening the invite URL and creating your account:
- You are now logged in as the **CEO** (the top-level admin)
- You can start creating companies, hiring agents, and assigning tasks
- Explore the Paperclip dashboard — it has sections for Org Chart, Tasks, Budgets, Governance, and more

### Quick Verification

To make sure everything is working, run:
```bash
curl http://paperclip.YOUR_SERVER_IP.sslip.io
```

You should get HTML output from Paperclip's frontend.

---

## Complete Reference Summary

### Environment Variables Checklist

| Variable | Example Value |
|----------|---------------|
| `HOST` | `0.0.0.0` |
| `PAPERCLIP_HOME` | `/paperclip` |
| `PAPERCLIP_PUBLIC_URL` | `http://paperclip.YOUR_SERVER_IP.sslip.io` |
| `BETTER_AUTH_SECRET` | `f47ac10b58cc4372...64chars` |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | `paperclip.YOUR_SERVER_IP.sslip.io` |
| `PAPERCLIP_TELEMETRY_DISABLED` | `1` *(optional)* |

### Volume Mount Checklist

| Mount Type | Source (Host) | Destination (Container) |
|------------|---------------|---------------------------|
| Directory Mount | `/data/coolify/applications/[UUID]` (auto) | `/paperclip` |

### Port Configuration

| Setting | Value |
|---------|-------|
| Port Exposes | `3100` |
| Public URL | `http://paperclip.YOUR_SERVER_IP.sslip.io` |

---

## Troubleshooting

### Problem: "Remote branch main not found"
**Cause:** You selected branch `main` but Paperclip uses `master`.  
**Fix:** Change the Git Branch in Coolify app settings from `main` to `master`. Redeploy.

### Problem: "EACCES: permission denied, mkdir '/paperclip/instances/default/logs'"
**Cause:** The host volume folder is owned by `root`, but Paperclip runs as user `node` (UID 1000).  
**Fix:** SSH into the server and run:
```bash
chown -R 1000:1000 /data/coolify/applications/YOUR_APP_UUID
chmod -R u+rwX /data/coolify/applications/YOUR_APP_UUID
```
Then redeploy.

### Problem: "New container is unhealthy" during deployment
**Cause:** Health check is failing because the app hasn't finished booting or `curl` is missing from the image.  
**Fix:** Go to the **Health Check** tab in Coolify and toggle it **OFF**, then redeploy.

### Problem: "No configuration changed & image found... Build step skipped"
**Cause:** Coolify cached the Docker image because you didn't change any code, but the container is still exiting.  
**Fix:** Make any small change (like editing the description) and click **Redeploy**, or force a rebuild from the deployment settings.

### Problem: "I can't access the URL in my browser"
**Causes & Fixes:**
- **DNS hasn't propagated:** Wait 2-3 minutes for `sslip.io` to update
- **Firewall blocking port 80/443:** Make sure your server's firewall allows HTTP traffic
- **Wrong FQDN:** Double-check that `PAPERCLIP_PUBLIC_URL` and the Coolify FQDN match
- **App not running:** Check Coolify logs for errors. If the container is stopped, check the deployment logs for the real issue.

### Problem: "Invite URL doesn't work"
**Cause:** The `PAPERCLIP_PUBLIC_URL` does not match the URL you're actually using in your browser.  
**Fix:** Make sure your browser URL exactly matches what you put in `PAPERCLIP_PUBLIC_URL` and `PAPERCLIP_ALLOWED_HOSTNAMES`, including `http` vs `https`.

---

## Credits & Resources

- **Paperclip Repository:** [https://github.com/paperclipai/paperclip](https://github.com/paperclipai/paperclip)
- **Paperclip Documentation:** [https://paperclip.ing/docs](https://paperclip.ing/docs)
- **Coolify Documentation:** [https://coolify.io/docs](https://coolify.io/docs)
- **Community Help:** [Paperclip Discord](https://discord.gg/m4HZY7xNG3)

**Special thanks to** the Paperclip and Coolify communities for the tools, and to all the folks who shared their Docker setup notes that made this guide possible.

---

## Quick Commands Reference

> **Note:** In all commands below, replace `$(docker ps -q --filter "publish=3100")` with your actual container name if you prefer. Find it with `docker ps`.

**Check deployed containers:**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

**View Paperclip logs:**
```bash
docker logs $(docker ps -q --filter "publish=3100")
```

**SSH into the Paperclip container as root:**
```bash
docker exec -it $(docker ps -q --filter "publish=3100") bash
```

**SSH into the Paperclip container as the node user:**
```bash
docker exec -it --user node $(docker ps -q --filter "publish=3100") bash
```

**Restart the Paperclip app from Coolify CLI (on server):**
```bash
coolify restart paperclip
```

---

**Happy orchestrating!** 🎉
