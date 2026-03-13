---
title: "Give Every Microservice the Same AI Brain: Sharing Agent Configurations with Git Submodules"
description: "Stop copy-pasting your .agent folder into every repo. Learn how to use git submodules to share AI agent skills, workflows, and rules across a microservice architecture — one source of truth, zero drift."
focus_keyphrase: "git submodules shared AI agent configuration"
keywords: ["git submodules shared AI agent configuration", "AI agent microservices", ".agent folder git submodule", "Claude Agent Skills shared repo", "coding assistant configuration management"]
author: ""
date: "2026-03-13"
---

# Give Every Microservice the Same AI Brain: Sharing Agent Configurations with Git Submodules

You spent a weekend crafting the perfect AI Agent setup — custom skills for unit testing, code review workflows, a `.agent/` folder that turns your coding assistant into a team-wide automation engine. Then reality hits: you have **12 microservices** and you just copy-pasted that folder into service #3, already forgetting which version service #1 is running.

**Your AI agent has amnesia.** Every repo has a slightly different brain, and "just update all of them" becomes the new technical debt nobody signed up for.

There is a better way — and it starts with `git submodule add`.

In this post you will learn:

1. Why **AI agent configuration drift** is a real problem in polyrepo architectures
2. How to extract your `.agent/` folder into a **standalone repository**
3. How to import it as a **git submodule** so every microservice shares one brain
4. The update workflow, CI/CD considerations, and common gotchas

---

## The Problem — Agent Config Drift in Polyrepo Architectures

Modern AI coding assistants — Claude Code, GitHub Copilot, Cursor — rely on **project-level configuration folders** to understand your team's conventions. The folder names vary (`.agent/`, `.cursor/`, `.github/agents/`), but the idea is the same: a directory of files that transforms a general-purpose AI into a specialist that knows *your* stack.

Here is what a typical `.agent/` folder looks like:

```
.agent/
├── skills/
│   ├── dotnet-unit-testing/
│   │   └── SKILL.md
│   ├── dotnet-code-review/
│   │   └── SKILL.md
│   └── tech-blog-writing/
│       └── SKILL.md
└── workflows/
    ├── create-blog-workflow.md
    └── dotnet-feature-workflow.md
```

This folder is gold — it encodes weeks of trial-and-error tuning into repeatable, automated behavior. But the moment you have more than one repository, a painful pattern emerges:

| Repo | `.agent/` version | Last updated |
|---|---|---|
| `orders-api` | v3 (latest skills) | Yesterday |
| `payments-api` | v1 (missing code-review skill) | 3 months ago |
| `notifications-api` | v2 (outdated workflow) | 6 weeks ago |
| `gateway-api` | ❌ (never added) | N/A |

Every repo slowly drifts. A new hire clones `payments-api` and wonders why their AI assistant cannot review code. A senior engineer updates the testing skill in `orders-api` but forgets to propagate it. **The "single source of truth" has become 12 slightly different sources of confusion.**

---

## Why Git Submodules Are the Right Tool Here

Git submodules let you embed one Git repository inside another as a subdirectory. The parent repo stores a **pointer to a specific commit** in the submodule — not a copy of the files. This makes submodules ideal for content that is:

- **Shared across many repos** — one central repo, many consumers
- **Versioned independently** — each microservice controls *when* to upgrade
- **Relatively stable** — skills and workflows change weekly, not hourly

You may already be wondering: *"Why not just use a monorepo or a package manager?"* Fair question. Here is how submodules stack up against the alternatives:

| Approach | Pros | Cons | Best for |
|---|---|---|---|
| **Git submodules** | Pinned commits, independent versioning, no build step | Manual update workflow, learning curve | Config files, skills, templates |
| **Git subtrees** | Simpler clone (no `--recurse`), embedded history | Harder to push changes upstream, noisy history | Small, rarely-changed shared code |
| **Private packages (npm/NuGet)** | Proper dependency management, semver built-in | Overkill for config files, requires registry | Shared libraries, SDKs |
| **Monorepo** | Everything in one place, atomic cross-repo changes | Scaling complexity, tooling overhead | Tightly coupled services |

For a folder of markdown files and YAML configs, submodules hit the sweet spot: **lightweight, pinned, and zero build infrastructure required.**

---

## Step-by-Step — Extracting and Sharing Your `.agent/` Folder

### Step 1: Create the Central `agent-config` Repository

Start by creating a dedicated repository that will hold your shared agent configuration. This becomes the **single source of truth**.

```bash
# On GitHub / GitLab / Azure DevOps, create a new repo called "agent-config"
# Then clone it locally:
git clone https://github.com/your-org/agent-config.git
cd agent-config
```

### Step 2: Move Your Skills and Workflows Into It

Copy the contents of your best `.agent/` folder into the new repository. Keep the internal directory structure intact — the AI tools expect specific paths.

```
agent-config/          ← this IS the .agent/ folder content
├── skills/
│   ├── dotnet-unit-testing/
│   │   ├── SKILL.md
│   │   └── PATTERNS.md
│   ├── dotnet-code-review/
│   │   └── SKILL.md
│   └── tech-blog-writing/
│       └── SKILL.md
└── workflows/
    ├── create-blog-workflow.md
    └── dotnet-feature-workflow.md
```

Commit and push:

```bash
git add .
git commit -m "feat: initial agent config with skills and workflows"
git push origin main
```

> **Tip:** Tag your first stable version so microservices can pin to it.
>
> ```bash
> git tag v1.0.0
> git push origin v1.0.0
> ```

### Step 3: Add the Submodule to Each Microservice

In every microservice repository, add `agent-config` as a submodule mounted at the `.agent` path:

```bash
cd /path/to/orders-api

# Add the submodule — this creates the .agent/ directory
git submodule add https://github.com/your-org/agent-config.git .agent

# Commit the submodule reference
git commit -m "feat: add shared agent config as submodule"
git push
```

After this command, Git creates two things:

1. **`.gitmodules`** — a file in the repo root that tracks submodule URLs:

   ```ini
   [submodule ".agent"]
       path = .agent
       url = https://github.com/your-org/agent-config.git
   ```

2. **A pointer entry** — the `.agent` directory now points to a specific commit hash in `agent-config`, not a branch.

### Step 4: Verify the Setup

Run a quick check to confirm everything is wired correctly:

```bash
# View submodule status
git submodule status

# Expected output:
#  a1b2c3d4 .agent (v1.0.0)
```

Open your AI coding assistant. It should automatically discover the skills and workflows inside `.agent/` — just as if the folder were native to the repo.

---

## Keeping Every Brain in Sync — The Update Workflow

When you add a new skill or update a workflow in `agent-config`, each microservice needs to **pull the new version explicitly**. This is a feature, not a bug — it prevents surprise breakage.

### Updating a Single Microservice

```bash
cd /path/to/orders-api

# Fetch the latest from the submodule's remote
cd .agent
git fetch origin
git checkout v1.1.0    # or: git checkout main
cd ..

# Commit the updated pointer in the parent repo
git add .agent
git commit -m "chore: update agent config to v1.1.0"
git push
```

### Updating All Microservices at Once

For teams with many repos, script it:

```bash
#!/bin/bash
# update-agent-config.sh
REPOS=("orders-api" "payments-api" "notifications-api" "gateway-api")
TARGET_TAG="v1.1.0"

for repo in "${REPOS[@]}"; do
  echo "Updating $repo..."
  cd "/path/to/$repo"
  git submodule update --init
  cd .agent
  git fetch origin
  git checkout "$TARGET_TAG"
  cd ..
  git add .agent
  git commit -m "chore: update agent config to $TARGET_TAG"
  git push
done
```

### Pinning Strategy: Tags vs. Branches

| Strategy | When to use |
|---|---|
| **Pin to a tag** (e.g., `v1.2.0`) | Production repos, stable environments. You decide exactly when each repo upgrades. |
| **Pin to `main`** branch | Development environments where you always want the latest agent config. |

**Recommendation:** Use **semantic version tags** for production and `main` for active development branches. This gives you the best of both worlds — stability where it matters, speed where it helps.

---

## CI/CD Considerations

Submodules require one extra flag in your CI pipelines. Without it, your builds will see an empty `.agent/` directory.

### GitHub Actions

```yaml
- uses: actions/checkout@v4
  with:
    submodules: recursive   # ← this is the key line
```

### Azure DevOps

```yaml
steps:
  - checkout: self
    submodules: true
```

### GitLab CI

```yaml
variables:
  GIT_SUBMODULE_STRATEGY: recursive
```

### Local Clone (New Team Members)

When someone clones a repo for the first time, they need to initialize submodules:

```bash
# Option A: Clone with submodules in one step
git clone --recurse-submodules https://github.com/your-org/orders-api.git

# Option B: If already cloned without submodules
git submodule init
git submodule update
```

> **Pro tip:** Add this to your repo's `README.md` or `CONTRIBUTING.md` so no one gets an empty `.agent/` folder on their first day.

---

## Gotchas and Solutions

### 1. "My `.agent/` Folder is Empty After Clone!"

**Cause:** The developer cloned without `--recurse-submodules`.

**Fix:**

```bash
git submodule init && git submodule update
```

**Prevention:** Add a post-clone check to your `Makefile` or `package.json` scripts:

```json
{
  "scripts": {
    "postinstall": "git submodule init && git submodule update"
  }
}
```

### 2. "Git Shows My Submodule as Modified But I Didn't Change Anything"

**Cause:** The submodule's `HEAD` drifted (e.g., someone ran `cd .agent && git pull` without committing the pointer in the parent).

**Fix:**

```bash
git submodule update --init
```

### 3. "My IDE Doesn't Detect Skills Inside the Submodule"

**Cause:** Some tools index files before submodules are initialized, or do not follow symlinks.

**Fix:** Ensure submodules are initialized before opening the project. Most AI coding assistants (Claude Code, Cursor) read from the filesystem at runtime, so they will see the `.agent/` contents as long as the files are physically present.

### 4. "I Need Per-Service Overrides"

**Cause:** One microservice needs an extra skill that doesn't apply to others.

**Fix:** Use a layered approach:

```
orders-api/
├── .agent/                    ← shared (submodule)
│   ├── skills/
│   └── workflows/
└── .agent-local/              ← service-specific overrides
    └── skills/
        └── orders-domain/
            └── SKILL.md
```

Many AI tools support merging multiple config directories. If yours doesn't, reference the local folder from the shared workflow files.

---

## The Architecture at a Glance

Here is the final picture — one central `agent-config` repo powering every microservice in your fleet:

```
                    ┌──────────────────────┐
                    │   agent-config repo  │
                    │   (single source)    │
                    │                      │
                    │  skills/             │
                    │  workflows/          │
                    │  tag: v1.2.0         │
                    └──────┬───────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
   ┌────────▼─────┐ ┌─────▼──────┐ ┌─────▼──────────┐
   │  orders-api  │ │payments-api│ │notifications-api│
   │              │ │            │ │                 │
   │  .agent/ ──► │ │ .agent/ ►  │ │  .agent/ ►     │
   │  (v1.2.0)   │ │ (v1.1.0)  │ │  (v1.2.0)      │
   └──────────────┘ └────────────┘ └─────────────────┘
        submodule       submodule        submodule
```

Each microservice pins to the version it has tested. Upgrades are deliberate, not accidental.

---

## What to Do Next

You now have a blueprint for eliminating agent configuration drift across your entire microservice fleet. Here are three concrete next steps:

1. **Create the `agent-config` repository** — Extract your best `.agent/` folder today. Tag it `v1.0.0`.
2. **Add it as a submodule to one service** — Start with your most actively developed microservice. Verify your AI assistant discovers the skills automatically.
3. **Update your CI pipeline** — Add `submodules: recursive` to your checkout step. This is a one-line change that prevents empty-folder surprises.

Once you have one service working, rolling out to the rest is `git submodule add` away.

---

*Sources Consulted:*
- [Git Submodules Documentation](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [GitHub Actions: Checkout with Submodules](https://github.com/actions/checkout#checkout-submodules)
- [Claude Agent Skills Documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Azure DevOps: Checkout Submodules](https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/pipeline-options-for-git?view=azure-devops#checkout-submodules)
