# Module 05: Cross-Repo Development with AI

> Most real work spans multiple repositories. AI tools are built around single repos. This module bridges that gap.

---

## 5.1 The Problem

You have a frontend repo, a backend repo, a shared library, and an infrastructure repo. You need to:
- Add a field to the API → update the frontend → update the shared types → deploy
- Debug an issue that spans the backend and a microservice
- Refactor a shared library without breaking 4 downstream consumers

AI coding agents are scoped to **one working directory**. They don't natively "see" across repos. So how do you connect the dots?

---

## 5.2 The Cross-Repo Strategy Stack

```
┌──────────────────────────────────────────┐
│  5. Orchestration Scripts                 │  ← automate multi-repo workflows
├──────────────────────────────────────────┤
│  4. Multi-Instance Agent Coordination     │  ← agents in each repo, you bridge
├──────────────────────────────────────────┤
│  3. Shared Context via Memory & Docs      │  ← knowledge that spans repos
├──────────────────────────────────────────┤
│  2. Cross-Repo CLAUDE.md System           │  ← teach agents about the ecosystem
├──────────────────────────────────────────┤
│  1. Project Topology Map                  │  ← know your own landscape first
└──────────────────────────────────────────┘
```

---

## 5.3 Layer 1: Project Topology Map

Before any AI can help across repos, **you** need a clear map.

### Create a Topology Document

Create a central document (could live in this learning project or a dedicated "meta" repo):

```markdown
# My Project Ecosystem

## Repos & Roles
| Repo | Purpose | Language | Depends On | Depended On By |
|------|---------|----------|------------|----------------|
| myapp-api | REST API backend | TypeScript/Express | shared-types, auth-lib | myapp-web, mobile-app |
| myapp-web | React frontend | TypeScript/Next.js | shared-types | — |
| shared-types | Shared TS interfaces & validators | TypeScript | — | myapp-api, myapp-web, mobile-app |
| auth-lib | Authentication library | TypeScript | — | myapp-api |
| infra | Terraform + CI/CD configs | HCL/YAML | — | all repos (deployment) |

## Data Flow
User → myapp-web → myapp-api → database
                  ↘ shared-types ↙

## Common Cross-Repo Tasks
- Adding a new API field: shared-types → myapp-api → myapp-web
- Auth changes: auth-lib → myapp-api → integration tests
- Deploy: infra → CI/CD triggers across repos
```

**Why this matters:** When you tell an AI agent "I'm adding a `profileImage` field to the User model," it needs to know which repos are affected and in what order. This map is the starting point.

---

## 5.4 Layer 2: Cross-Repo CLAUDE.md System

Each repo gets its own `CLAUDE.md`, but they should **reference each other**.

### Pattern: The Ecosystem-Aware CLAUDE.md

In `myapp-api/CLAUDE.md`:
```markdown
# myapp-api

## Ecosystem Context
This is the backend API. It is part of a multi-repo system:
- **shared-types** (../shared-types): All TypeScript interfaces live here.
  When adding/changing API response shapes, the types MUST be updated
  in shared-types first, then consumed here.
- **myapp-web** (../myapp-web): The frontend consumer of this API.
  Breaking API changes require coordinated updates.
- **auth-lib** (../auth-lib): Provides JWT validation middleware.
  Imported as `@myorg/auth-lib`.

## Cross-Repo Rules
- NEVER define response types locally — import from @myorg/shared-types
- API changes must be backward-compatible unless coordinated with frontend
- After changing an endpoint, check: does myapp-web call this? (grep the frontend repo)

## How to Check Other Repos
- Types: `ls ../shared-types/src/`
- Frontend API calls: `grep -r "api/endpoint" ../myapp-web/src/`
- Auth middleware: `cat ../auth-lib/src/middleware.ts`
```

### Pattern: The Sibling-Aware Setup

If your repos live as siblings in the same parent directory:
```
~/projects/
├── myapp-api/
├── myapp-web/
├── shared-types/
├── auth-lib/
└── infra/
```

Claude Code can read files from sibling repos using relative paths (`../myapp-web/src/api/users.ts`). This is the simplest cross-repo technique — just tell the agent where to look.

### Pattern: Global CLAUDE.md

Claude Code supports a global `~/.claude/CLAUDE.md` that applies to ALL projects. Use it for ecosystem-wide context:

```markdown
# Global Claude Configuration

## My Project Ecosystem
I work on a multi-repo system. Repos are in ~/projects/:
- myapp-api, myapp-web, shared-types, auth-lib, infra
- shared-types must be updated before api or web for type changes
- All repos use pnpm, TypeScript strict mode, Vitest for testing

## Cross-Repo Conventions
- Branch naming: feature/JIRA-123-description
- All repos share the same PR template
- Changes spanning repos: create linked PRs with "Part 1 of N" in title
```

---

## 5.5 Layer 3: Shared Context via Memory & Documentation

### Claude Code Memory Spans Conversations, Not Repos

Claude Code's memory is project-scoped (stored per working directory). But you can create cross-cutting memories:

**Technique: Narrate cross-repo decisions**

When working in repo A and you make a decision that affects repo B, tell Claude:
```
Remember: we changed the User type to include profileImage (string | null).
This needs to be updated in myapp-web and shared-types repos too.
The API endpoint PATCH /users now accepts profileImage.
```

When you later open Claude Code in repo B, you can ask:
```
Check your memory — were there any recent changes in myapp-api that affect this repo?
```

### The "Context Bridge" Document

For complex cross-repo changes, create a temporary bridge document:

```markdown
# Cross-Repo Change: Add Profile Image

## Status: In Progress

## Change Sequence
1. [x] shared-types: Add `profileImage: string | null` to User interface
2. [x] myapp-api: Add PATCH /users/profileImage endpoint, add DB migration
3. [ ] myapp-web: Add image upload component, update profile page
4. [ ] infra: Add S3 bucket for image storage, update CORS

## Key Decisions
- Images stored in S3, URLs in database
- Max size: 5MB, formats: jpg/png/webp
- Resize to 256x256 on upload (server-side)

## API Contract
PATCH /users/:id/profile-image
Content-Type: multipart/form-data
Body: { image: File }
Response: { profileImage: "https://s3.../image.webp" }
```

Put this in a shared location (or just paste relevant parts when switching repos). It gives each agent the full picture.

---

## 5.6 Layer 4: Multi-Instance Agent Coordination

### Pattern 1: Sequential Repo Hopping

The simplest approach — work in one repo at a time, carry context manually.

```
Terminal 1 (shared-types):
  "Add profileImage: string | null to the User interface. Export it."

Terminal 2 (myapp-api):
  "Read ../shared-types/src/user.ts to see the updated User type.
   Add a PATCH /users/:id/profile-image endpoint that accepts an image upload."

Terminal 3 (myapp-web):
  "Read ../myapp-api/src/routes/users.ts to see the new endpoint.
   Add a profile image upload component that calls PATCH /users/:id/profile-image."
```

**Pros:** Simple, full control, no conflicts
**Cons:** Slow, manual context transfer

### Pattern 2: Parallel Terminals

Run Claude Code in multiple terminals simultaneously, one per repo.

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Terminal 1       │  │ Terminal 2       │  │ Terminal 3       │
│ ~/shared-types   │  │ ~/myapp-api      │  │ ~/myapp-web      │
│                  │  │                  │  │                  │
│ Claude: updating │  │ Claude: waiting  │  │ Claude: waiting  │
│ User interface   │  │ for types...     │  │ for API...       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │                    │                    │
         ├── Step 1: types ───┤                    │
         │                    ├── Step 2: API ─────┤
         │                    │                    ├── Step 3: frontend
```

**Key:** The dependency order still matters. Start the downstream repos once upstream changes are committed/saved.

### Pattern 3: The Monorepo Agent Trick

If your repos are siblings, you can cheat — open Claude Code in the **parent** directory:

```bash
cd ~/projects
claude
```

Now the agent can see ALL repos:
```
I need to add a profileImage field across the system:
1. Read shared-types/src/user.ts and add profileImage: string | null
2. Read myapp-api/src/routes/users.ts and add the upload endpoint
3. Read myapp-web/src/pages/profile.tsx and add the upload UI

Start with shared-types, then work through each repo in dependency order.
```

**Pros:** Single context window sees everything. True cross-repo reasoning.
**Cons:** Large context usage. CLAUDE.md setup is trickier (need one at parent level). May confuse per-repo tooling (package.json, tsconfig, etc.).

**When to use:** Tightly coupled repos that frequently change together. Especially effective for type-sharing systems.

### Pattern 4: Sub-Agent Per Repo

Use Claude Code's Agent tool to spawn a sub-agent for each repo:

```
I need to add profileImage across 3 repos. Spawn agents for each:

Agent 1 (shared-types): "cd ~/projects/shared-types && add profileImage: string | null to User interface in src/user.ts, run build"

Agent 2 (myapp-api): "cd ~/projects/myapp-api && read ../shared-types/src/user.ts, then add PATCH /users/:id/profile-image endpoint with tests"

Agent 3 (myapp-web): "cd ~/projects/myapp-web && read ../myapp-api/src/routes/users.ts, then add profile image upload component"
```

**Note:** For dependent changes, Agent 1 should complete before Agent 2 starts reading its output. Independent parts can run in parallel.

---

## 5.7 Layer 5: Orchestration Scripts

For repeated cross-repo workflows, automate them.

### The Cross-Repo Task Runner

Create a simple script that drives Claude across repos:

```bash
#!/bin/bash
# cross-repo-change.sh — Orchestrate a change across multiple repos

CHANGE_DESCRIPTION="$1"
REPOS_DIR=~/projects

echo "=== Step 1: Update shared-types ==="
cd "$REPOS_DIR/shared-types"
claude --print "Read CLAUDE.md, then: $CHANGE_DESCRIPTION — only the shared-types part. Run build when done."

echo "=== Step 2: Update API ==="
cd "$REPOS_DIR/myapp-api"
claude --print "Read ../shared-types/src/ to see recent changes. Then: $CHANGE_DESCRIPTION — only the API part. Run tests."

echo "=== Step 3: Update Frontend ==="
cd "$REPOS_DIR/myapp-web"
claude --print "Read ../myapp-api/src/routes/ to see recent changes. Then: $CHANGE_DESCRIPTION — only the frontend part. Run tests."

echo "=== Done ==="
```

Usage:
```bash
./cross-repo-change.sh "Add profileImage field to User"
```

### GitHub Actions: Cross-Repo PR Coordination

```yaml
# In shared-types repo: .github/workflows/notify-dependents.yml
name: Notify Dependent Repos
on:
  push:
    branches: [main]
    paths: ['src/**']

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger downstream updates
        run: |
          # Trigger workflow in myapp-api
          gh workflow run update-types.yml --repo myorg/myapp-api
          # Trigger workflow in myapp-web
          gh workflow run update-types.yml --repo myorg/myapp-web
```

### The Cross-Repo PR Pattern

When a change spans repos, create linked PRs:

```
shared-types PR #42:  "Add profileImage to User type [1/3]"
myapp-api PR #87:     "Add profile image endpoint [2/3] — depends on shared-types#42"
myapp-web PR #156:    "Add profile image upload UI [3/3] — depends on myapp-api#87"
```

Ask Claude to create these with the right cross-references:
```
Create a PR for the changes in this repo. Title it "[2/3] Add profile image endpoint".
In the body, note that this depends on shared-types PR #42 and is required by myapp-web (PR forthcoming).
```

---

## 5.8 Advanced: MCP Servers for Cross-Repo Intelligence

### What MCP Enables

MCP servers can give your AI agent superpowers across repos:

**Custom MCP Server ideas:**
- **Repo Registry**: An MCP server that knows your entire repo ecosystem — dependencies, owners, build status
- **Type Checker**: An MCP server that validates type compatibility across repos
- **Change Propagator**: An MCP server that, given a type change, identifies all downstream files that need updating
- **Cross-Repo Search**: An MCP server that searches across all your repos simultaneously

### Example: Simple Cross-Repo Search MCP Server

A lightweight MCP server that searches across multiple repos:

```typescript
// Conceptual — the idea, not production code
server.tool("cross_repo_search", async ({ query, repos }) => {
  const results = [];
  for (const repo of repos) {
    const matches = await ripgrep(query, path.join(REPOS_DIR, repo));
    results.push({ repo, matches });
  }
  return results;
});

server.tool("find_type_usages", async ({ typeName }) => {
  // Search all repos for imports/usages of a specific type
  const repos = await getRepoList();
  const usages = [];
  for (const repo of repos) {
    const matches = await ripgrep(`import.*${typeName}|: ${typeName}[\\s,;>]`, repo);
    usages.push({ repo, matches });
  }
  return usages;
});
```

Once connected via MCP, you can ask Claude:
```
Search across all my repos for anything that uses the User type.
I need to know everywhere that will be affected by adding profileImage.
```

---

## 5.9 Patterns by Repo Relationship

### Tightly Coupled Repos (shared types, API + client)

**Best approach:** Monorepo Agent Trick (open parent dir) or Sequential Repo Hopping with a bridge document.

**Key principle:** Changes must flow in dependency order. Types → backend → frontend.

### Loosely Coupled Repos (microservices communicating via APIs)

**Best approach:** Parallel Terminals with contract-first design.

**Key principle:** Define the API contract first (OpenAPI spec), then each repo can be updated independently against the contract.

```
Step 1: "Update the OpenAPI spec for the user-service to include profileImage"
Step 2 (parallel):
  - Agent A: "Implement the new endpoint in user-service per the OpenAPI spec"
  - Agent B: "Update the API client in notification-service per the OpenAPI spec"
  - Agent C: "Update the frontend to use the new field per the OpenAPI spec"
```

### Shared Library + Consumers

**Best approach:** Version-bump workflow.

```
Step 1: Update shared library, bump version, publish
Step 2: In each consumer, update dependency version, fix any breaking changes
```

Ask Claude:
```
I just published @myorg/shared-types@2.3.0 which adds profileImage to User.
Update the dependency in this repo and fix any type errors that result.
Run `pnpm install` then `pnpm typecheck` to find issues.
```

### Infrastructure + Application Repos

**Best approach:** Infra-first with environment variable contracts.

```
Step 1 (infra): "Add S3 bucket for profile images, output the bucket name as env var PROFILE_IMAGE_BUCKET"
Step 2 (api): "Read the infra changes to see the new env var. Add PROFILE_IMAGE_BUCKET to .env.example and use it in the upload handler."
```

---

## 5.10 Common Pitfalls in Cross-Repo AI Work

| Pitfall | What Happens | Fix |
|---------|-------------|-----|
| No dependency order | Agent updates frontend before types exist | Document the change sequence explicitly |
| Stale cross-references | Agent reads old version of sibling repo | Commit/save upstream changes before downstream reads |
| Context overload | Opening parent dir fills context with irrelevant repos | Use targeted file reads, not broad exploration |
| Inconsistent conventions | Each repo's agent follows different patterns | Global CLAUDE.md + per-repo CLAUDE.md |
| Breaking changes without coordination | API change breaks frontend | Use bridge documents, linked PRs |
| Forgetting a consumer | Shared type changes but one consumer isn't updated | Maintain the topology map, grep for usages |

---

## 5.11 Your Cross-Repo Checklist

### Setup (Do Once)
- [ ] Create a topology map of your repo ecosystem
- [ ] Add ecosystem context to each repo's CLAUDE.md
- [ ] Create a global `~/.claude/CLAUDE.md` with cross-cutting conventions
- [ ] Organize repos as siblings for easy relative-path access
- [ ] Document the dependency order for common changes

### Per Cross-Repo Change
- [ ] Create a bridge document describing the full change
- [ ] Identify the dependency order (which repo first?)
- [ ] Work upstream → downstream
- [ ] Verify each repo's tests pass before moving to the next
- [ ] Create linked PRs with clear cross-references
- [ ] Update the bridge document as you go

---

## Self-Check Questions

- [ ] What's the difference between the "monorepo agent trick" and "sub-agent per repo"?
- [ ] When would you use parallel vs. sequential cross-repo updates?
- [ ] How does a global CLAUDE.md help in a multi-repo setup?
- [ ] Design a cross-repo change sequence for adding a new field to a shared API
- [ ] What MCP server would be most valuable for your specific repo ecosystem?

→ **Back to [README](../README.md) | Previous: [Module 04](04-career-growth.md)**
