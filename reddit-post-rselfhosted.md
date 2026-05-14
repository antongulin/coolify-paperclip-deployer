# Reddit Post: r/selfhosted

---

**Project Name:** Paperclip Coolify Deployer

**Repo/Website Link:** https://github.com/antongulin/coolify-paperclip-deployer

**Description:**
This is a complete deployment guide + an AI agent skill for self-hosting [Paperclip](https://github.com/paperclipai/paperclip) — an open-source AI agent orchestration dashboard — on your own Coolify v4 server.

I deployed Paperclip manually, hit every sharp edge, documented every fix, and wrapped the entire workflow into a reusable skill for OpenCode. Instead of 30 minutes of manual clicking and SSHing, you paste one prompt into any AI assistant with Coolify MCP access:

> "Deploy Paperclip on my Coolify server."

The agent then handles project creation, the critical `master` branch gotcha (Paperclip uses `master`, not `main`), setting `PAPERCLIP_PUBLIC_URL` and `BETTER_AUTH_SECRET`, mounting `/paperclip` persistent storage, fixing volume permissions (`chown 1000:1000`), disabling broken health checks, deploying, and running `docker exec ... paperclipai onboard` to generate your CEO invite URL.

**Key features:**
- 8-phase step-by-step deployment workflow
- All gotchas documented (`master` branch, UID mismatch, missing `curl` in image)
- Exact environment variable table
- Persistent storage config (`/paperclip` mount)
- Post-deploy onboarding command

**What problem it solves:**
Deploying Paperclip on Coolify isn't officially supported in the marketplace. Without this guide, you're trial-and-erroring through branch errors, permission denied on Docker volumes, health check failures, and invite links that break because of a mismatched `PAPERCLIP_PUBLIC_URL`.

**Deployment:**
Paperclip itself deploys via Dockerfile on Coolify v4. The repo includes:
- `GUIDE.md` — full manual deployment instructions
- `paperclip-coolify-deployer/SKILL.md` — the AI skill for OpenCode
- Source repo (what gets deployed): https://github.com/paperclipai/paperclip
- Requirements: Coolify v4 server, 2 GB RAM, 2 CPU cores, 10 GB disk

**AI Involvement:**
I did every deployment step manually first — the AI did not deploy it for me. After I got it working, I used the [opencode-skill-creator](https://github.com/antongulin/opencode-skill-creator) to generate test cases, benchmark the skill against a baseline, and optimize the description until it triggered reliably. Without the skill, AI agents hallucinate the wrong "Paperclip" (the 2013 Rails `thoughtbot/paperclip` gem). With the skill: 100% pass rate. Without it: 17%. The tool is free and open source.
