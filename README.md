# Coolify Paperclip Deployer

> One-line AI agent prompt to deploy Paperclip on Coolify — fully automated and battle-tested.

## What is this?

This repository contains an **OpenCode-compatible skill** and a companion **step-by-step installation guide** for deploying [Paperclip](https://github.com/paperclipai/paperclip) — an open-source AI agent orchestration platform — on a self-hosted [Coolify](https://coolify.io/) v4 server.

After following this guide, you will have:
- **Paperclip** running in a Docker container on your own server
- A public URL to access it (e.g. `http://paperclip.YOUR_IP.sslip.io`)
- Persistent storage that survives restarts
- A fully configured instance ready to onboard your first "CEO" admin

## How to use it

Copy-paste **one of the following** into any AI coding assistant (OpenCode, Claude, Cursor, etc.) that supports skill loading and MCP tools (specifically the Coolify MCP):

### Option A — Quick (natural language)
```
Use the skill from https://github.com/antongulin/coolify-paperclip-deployer to deploy Paperclip on my Coolify server.
```

### Option B — Explicit (with the guide URL)
```
Read the full guide at https://github.com/antongulin/coolify-paperclip-deployer/blob/main/GUIDE.md and use the paperclip-coolify-deployer skill to deploy Paperclip on my Coolify server.
```

### Option C — Local install (for OpenCode users)
If you use **OpenCode** (or any agent that supports skill directories), clone this repo into your skills folder:

**Global (all projects):**
```bash
git clone https://github.com/antongulin/coolify-paperclip-deployer.git ~/.config/opencode/skills/paperclip-coolify-deployer
```

**Project-only:**
```bash
git clone https://github.com/antongulin/coolify-paperclip-deployer.git .opencode/skills/paperclip-coolify-deployer
```

Then simply ask your agent:
```
Deploy Paperclip on my Coolify server.
```

## What's inside

```
coolify-paperclip-deployer/
├── README.md                    ← This file
├── GUIDE.md                     ← The complete beginner-friendly guide
└── paperclip-coolify-deployer/
    ├── SKILL.md                 ← The OpenCode skill (what the agent reads)
    ├── benchmark.json           ← Quantitative evaluation results
    └── evals/
        ├── evals.json           ← Test prompts & assertions
        └── trigger-evals.json   ← Trigger-optimization queries
```

## Why a "skill" matters

A skill is not just documentation. It is a **structured prompt** that tells the AI agent *exactly* how to handle a complex, multi-step task — with all the real-world gotchas baked in.

For Paperclip on Coolify, there are at least **8 critical gotchas** that the skill prevents:
1. Using `master` branch instead of `main` (Paperclip repo uses `master`)
2. Generating a 64-character `BETTER_AUTH_SECRET`
3. Setting `PAPERCLIP_PUBLIC_URL` to exactly match the browser URL
4. Mounting `/paperclip` persistent storage
5. Fixing host volume permissions (`chown -R 1000:1000`)
6. Disabling Coolify health checks (container has no `curl`)
7. Running `pnpm paperclipai onboard` inside the container
8. Extracting the CEO invite URL

Without the skill, an AI agent will often **hallucinate** the wrong "Paperclip" (the deprecated Rails `thoughtbot/paperclip` gem for file uploads) and give you a completely broken deployment.

## Benchmark results

This skill was built using the OpenCode **skill-creator** workflow with real evaluations:

| Task | With Skill | Without Skill |
|------|-----------|---------------|
| Full deployment guide | 7/7 (100%) | 2/7 (29%) |
| Vague deployment request | 6/6 (100%) | 1/6 (17%) |
| Branch-error troubleshooting | 3/3 (100%) | 3/3 (100%) |

**Without the skill**, the baseline hallucinated the Rails gem and gave wrong commands. **With the skill**, every test passed.

## How this skill was made

This skill was created using the OpenCode **skill-creator** skill — a structured, iterative process for building and validating AI agent skills:

1. **Capture intent** — Translated a personal battle-tested guide into a skill
2. **Write SKILL.md** — Drafted the full 8-phase deployment workflow with all gotchas
3. **Create evals** — Built 3 realistic test scenarios with 16 assertions
4. **Run baseline vs skill** — Spawned 6 parallel runs to compare with/without the skill
5. **Grade & aggregate** — Scored every assertion, built `benchmark.json`
6. **Launch viewer** — Reviewed results in the eval viewer with side-by-side diffs
7. **Optimize description** — Improved the skill description for better triggering
8. **Validate & package** — Ran `skill_validate` and prepared for sharing

The entire workflow is documented in the [OpenCode skill-creator repository](https://github.com/antongulin/opencode-skill-creator).

## Prerequisites

- A running Coolify v4 server (green dot in sidebar)
- Docker / Buildx on the server
- At least **2 GB RAM, 2 CPU cores, 10 GB disk**
- AI agent with access to Coolify MCP tools (`coolify_*`)

## Credits & Resources

- **Paperclip Repo:** [github.com/paperclipai/paperclip](https://github.com/paperclipai/paperclip)
- **Paperclip Docs:** [paperclip.ing/docs](https://paperclip.ing/docs)
- **Coolify Docs:** [coolify.io/docs](https://coolify.io/docs)
- **Community:** [Paperclip Discord](https://discord.gg/m4HZY7xNG3)

---

Made with ❤️ using the [OpenCode skill-creator](https://github.com/antongulin/opencode-skill-creator).
