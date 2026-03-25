# Module 03: Maximizing Agent Capabilities

> Getting 10x output from AI agents is not about prompt tricks — it's about workflow design.

---

## 3.1 The Agent Capability Stack

Think of agent productivity as a stack. Each layer multiplies the one below it:

```
┌─────────────────────────────┐
│  5. Multi-Agent Orchestration │  ← 10x multiplier
├─────────────────────────────┤
│  4. Autonomous Workflows      │  ← works while you sleep
├─────────────────────────────┤
│  3. Tool & Context Mastery    │  ← right info at right time
├─────────────────────────────┤
│  2. Clear Intent Expression   │  ← precise instructions
├─────────────────────────────┤
│  1. Project Configuration     │  ← CLAUDE.md, memory, structure
└─────────────────────────────┘
```

Most people optimize layer 2 (better prompts) while ignoring layer 1 (project setup). This is like tuning an engine in a car with flat tires.

---

## 3.2 Layer 1: Project Configuration

### CLAUDE.md — Your Most Important File

A great CLAUDE.md turns a generic AI agent into a team-specific coding partner.

**Structure for maximum impact:**

```markdown
# Project Name

## Quick Reference
- Language: TypeScript 5.x / Node 20
- Package manager: pnpm
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`
- Single test: `pnpm test -- --grep "test name"`

## Architecture
- Monorepo with packages in `packages/`
- API: Express + TypeORM in `packages/api/`
- Web: Next.js 14 in `packages/web/`
- Shared types: `packages/types/`

## Conventions
- Use barrel exports (index.ts) for public APIs
- Error handling: Result<T, E> pattern (see packages/types/result.ts)
- Database: migrations in packages/api/migrations/, always test with `pnpm db:test`
- Tests: co-locate with source files as `*.test.ts`
- No default exports except for Next.js pages

## Critical Rules
- NEVER modify `packages/api/migrations/` without explicit approval
- ALWAYS run `pnpm typecheck` before committing
- Database schema changes require a migration, never modify entities directly

## Common Tasks
- Adding an API endpoint: see packages/api/src/routes/README.md for pattern
- Adding a web page: use `packages/web/src/app/` App Router conventions
```

### Memory System — Practical Playbook

Claude Code maintains persistent memory between conversations in `~/.claude/projects/<project>/memory/`. This is your highest-leverage tool for agent consistency across sessions.

**The memory architecture:**
```
~/.claude/projects/<project-hash>/memory/
├── MEMORY.md          ← Index (always in context, keep under 200 lines)
├── user_*.md          ← Who you are (role, expertise, preferences)
├── feedback_*.md      ← Corrections & confirmed approaches
├── project_*.md       ← Decisions, context, ongoing work
└── reference_*.md     ← Pointers to external systems
```

**Each memory file has frontmatter:**
```markdown
---
name: Auth migration decision
description: Why we chose event sourcing for orders — compliance-driven, not tech debt
type: project
---

We chose event sourcing for the order service because legal requires
a complete audit trail of all order state changes.

**Why:** Compliance requirement from legal team, not a technical preference.
**How to apply:** When making order-service architecture decisions, prioritize
auditability over simplicity.
```

**What to save (and what not to):**

| Save | Don't Save |
|------|------------|
| Your role and expertise level | Code patterns (derivable from codebase) |
| "Don't mock the database" (corrections) | Git history (use `git log`) |
| "Bundled PRs worked well here" (confirmed approaches) | Fix recipes (the fix is in the code) |
| "Auth rewrite driven by compliance" (project context) | Anything already in CLAUDE.md |
| "Bugs tracked in Linear project X" (external references) | Temporary/in-progress work details |

**Strategic memory patterns:**

**Pattern 1: The Day-One Dump**
First time using Claude Code on a project? Spend 5 minutes front-loading context:
```
Remember these things about this project:
- I'm a backend engineer, new to this codebase
- This is a fintech app, compliance matters more than speed
- The legacy/ folder is being phased out — never add to it
- Tests marked @slow hit real databases
- Bugs go in Linear project "PAYMENTS"
- Team lead is Sarah, she reviews all auth-related PRs
```

**Pattern 2: The Correction Loop**
When Claude does something wrong, correct it AND make it stick:
```
Don't use try-catch for expected errors in this project — we use the Result pattern.
Remember this for future conversations.
```
This becomes a feedback memory. Claude won't make the same mistake next session.

**Pattern 3: The Cross-Session Bridge**
At the end of a long session with important decisions:
```
Remember what we decided today:
- API pagination uses cursor-based, not offset-based
- Rate limiting will be per-API-key, not per-IP
- We're deferring the caching layer until after launch
```

**Pattern 4: The Recall Check**
Start a new session by asking what Claude remembers:
```
What do you remember about this project and my preferences?
```
If something's missing or outdated, correct it on the spot.

**Memory vs. CLAUDE.md — decision guide:**
```
Is this about the project (for everyone)?  → CLAUDE.md
Is this about YOU or YOUR decisions?       → Memory
Is this a build command or convention?     → CLAUDE.md
Is this a preference or correction?        → Memory
Should it be committed to git?             → CLAUDE.md
Is it personal to your workflow?           → Memory
```

**Maintenance:** Review memory files monthly. Projects evolve. Decisions get reversed. Stale memory is worse than no memory — it makes the agent confidently wrong.

### Git Hooks & CI Integration

Set up your environment so the agent gets immediate feedback:
- Pre-commit hooks: linting, formatting, type checking
- Test runners: the agent can run tests and see results
- CI status: the agent can check if CI passes

---

## 3.3 Layer 2: Clear Intent Expression

### The Intent Specification Framework

**Bad:** "Add user authentication"
**Good:**

```
Add JWT-based authentication to the API:

Requirements:
- POST /auth/login accepts { email, password } and returns { token, refreshToken }
- POST /auth/refresh accepts { refreshToken } and returns new { token }
- Middleware: protect all /api/* routes except /auth/*
- Token expiry: 15 min access, 7 day refresh
- Store refresh tokens in the database (not in-memory)

Constraints:
- Use the existing User model in packages/api/src/entities/user.ts
- Follow the existing route pattern in packages/api/src/routes/
- Use bcrypt for password hashing (already in package.json)
- Write tests for all endpoints

Non-goals:
- No OAuth/social login (yet)
- No email verification (separate task)
```

### The RICE Framework for Agent Tasks

| Element | Description | Example |
|---------|-------------|---------|
| **R**equirements | What must be true when done | "All tests pass, endpoint returns correct status codes" |
| **I**nput | What context/files the agent needs | "See `src/auth/` for existing patterns" |
| **C**onstraints | What the agent must NOT do | "Don't modify the User model schema" |
| **E**xamples | Concrete input/output pairs | "POST /login with valid creds → 200 + token" |

### Incremental vs. Big-Bang

**Incremental (preferred for most tasks):**
```
Step 1: "Create the auth middleware that validates JWT tokens"
Step 2: "Now add the /auth/login endpoint"
Step 3: "Add the /auth/refresh endpoint"
Step 4: "Write tests for all three"
```

**Big-Bang (okay for well-spec'd, isolated tasks):**
```
"Implement the full auth system per this spec: [detailed spec]"
```

**Rule of thumb:** If the task touches more than 5 files, go incremental. If it's self-contained, big-bang is fine.

---

## 3.4 Layer 3: Tool & Context Mastery

### Understanding What Claude Code Can Do

| Tool | What It Does | When to Leverage It |
|------|-------------|-------------------|
| **Read** | Read any file on disk | Before any edit — always have Claude read first |
| **Edit** | Surgical text replacement | Targeted changes to existing code |
| **Write** | Create/overwrite files | New files only |
| **Bash** | Run any shell command | Tests, builds, git, system commands |
| **Glob** | Find files by pattern | "Find all test files," "where are the routes?" |
| **Grep** | Search file contents | "Find where X is used," "who calls this function?" |
| **Agent** | Spawn sub-agents | Parallel research, independent tasks |
| **WebSearch/Fetch** | Access the internet | Documentation, API references, latest versions |
| **MCP tools** | External integrations | Notion, GitHub, Slack, custom tools |

### Context Management Strategies

**Strategy 1: Front-load critical context**
Put the most important information at the beginning of your message. The model pays more attention to the start.

**Strategy 2: Reference, don't paste**
Instead of pasting 500 lines of code, say "Read `src/auth/middleware.ts` — follow that pattern." The agent will read the actual current file content.

**Strategy 3: Use sub-agents for research**
When you need to explore a large codebase, spawn an Explore agent. It searches without filling your main context window with irrelevant results.

**Strategy 4: Start fresh when context gets stale**
If a conversation has been going for 50+ messages, start a new one. The agent's memory carries forward, but you get a clean context window.

---

## 3.5 Layer 4: Autonomous Workflows

### Working While You Sleep

This is the game-changer. Here's how to set up autonomous workflows:

#### Method 1: GitHub Copilot Coding Agent
- Assign a GitHub issue to `@copilot`
- It creates a branch, implements the feature, opens a PR
- You review when you wake up
- **Best for:** well-defined issues with clear acceptance criteria

#### Method 2: Claude Code Background Tasks
- Start Claude Code with a task and let it run
- Use `run_in_background` for long-running operations
- Set up scheduled tasks to run periodically
- **Best for:** maintenance, monitoring, repetitive tasks

#### Method 3: CI/CD-Triggered Agent Work
```yaml
# Example: Auto-fix lint errors on push
on: push
jobs:
  auto-fix:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: AI Lint Fix
        run: |
          claude --print "Fix all lint errors in the changed files.
          Run 'npm run lint' to see issues, fix them, and commit."
```

#### Method 4: Scheduled Agent Tasks
Use Claude Code's scheduled tasks or cron jobs:
- Nightly: "Review all TODOs in the codebase and create issues for stale ones"
- Weekly: "Check for dependency updates and create a PR if any are safe to upgrade"
- On PR: "Review this PR for security issues, test coverage, and convention compliance"

### Designing Tasks for Autonomous Execution

For a task to succeed without you:

1. **Clear success criteria**: "All tests pass" not "looks good"
2. **Bounded scope**: One feature, one fix — not "improve the app"
3. **Existing test coverage**: The agent needs automated feedback
4. **Well-documented patterns**: CLAUDE.md should cover conventions
5. **Safe defaults**: Agent should ask (or stop) if uncertain, not guess

### The Trust Ladder

```
Level 1: Agent suggests, you implement
Level 2: Agent implements, you review every line
Level 3: Agent implements + tests, you review the diff
Level 4: Agent implements + tests + commits, you review the PR
Level 5: Agent handles full issues, you review PRs in batches
Level 6: Agent merges low-risk PRs, you audit weekly
```

Most teams are at Level 3-4. Level 5-6 requires excellent test coverage, CI/CD, and organizational trust.

---

## 3.6 Layer 5: Multi-Agent Orchestration

### Pattern 1: Parallel Workers

Multiple agents work on independent tasks simultaneously.

```
You: "I need these 3 features built"

Agent A → Feature 1 (user profile page)
Agent B → Feature 2 (notification system)      [all in parallel]
Agent C → Feature 3 (search functionality)

You: Review all 3, provide feedback, agents iterate
```

**In Claude Code:** Use the Agent tool to spawn multiple sub-agents in a single message. Each gets its own context and works independently.

### Pattern 2: Pipeline (Sequential Specialists)

Each agent handles one phase, passing output to the next.

```
Architect Agent → Implementation Agent → Review Agent → Test Agent
   (designs)         (builds)             (critiques)    (verifies)
```

**In Claude Code:** You act as the orchestrator, passing output from one phase to the next. Or build a script that automates the pipeline.

### Pattern 3: Scout-Build-Verify

```
Scout Agent: "Explore the codebase and tell me how auth currently works"
    ↓ (you read the report)
Build Agent: "Implement the new auth system using this approach: [approach]"
    ↓ (you review the code)
Verify Agent: "Run all tests, check for security issues, verify the implementation"
```

**Why it works:** Each agent has a focused context window. The Scout doesn't need editing tools; the Builder doesn't need to explore everything.

### Pattern 4: Competing Approaches

Multiple agents try different solutions to the same problem.

```
Agent A: "Implement the search feature using Elasticsearch"
Agent B: "Implement the search feature using PostgreSQL full-text search"
Agent C: "Implement the search feature using in-memory with Fuse.js"

You: Compare results, pick the best approach (or hybrid)
```

**When to use:** Architectural decisions, performance-critical features, when you're genuinely unsure which approach is better.

### Pattern 5: Worktree Isolation

Each agent works in its own git worktree — a separate copy of the repo.

```
Main repo (your work)
├── Worktree A: Agent working on feature-x branch
├── Worktree B: Agent working on bugfix-y branch
└── Worktree C: Agent experimenting with refactor-z
```

**Benefits:**
- No conflicts between agents
- Each agent has a clean working state
- Easy to compare results
- Easy to discard failed experiments

**In Claude Code:** Use `isolation: "worktree"` when spawning agents.

---

## 3.7 Advanced Techniques

### The "Teach, Don't Tell" Technique

Instead of telling the agent exactly what to do, teach it your principles and let it apply them.

**Tell approach:** "Add error handling to every function in auth.ts using try-catch"

**Teach approach:** "Our error handling philosophy is: use Result<T, E> for expected failures, let unexpected errors bubble up to the global handler. Review auth.ts and apply this philosophy where it's missing."

The teach approach produces more thoughtful, context-aware results.

### The "Rubber Duck" Technique

Use the agent as a thinking partner, not just a doer.

"I'm trying to decide between approach A and approach B for the caching layer. Here's what I know: [context]. What trade-offs am I missing?"

Then decide yourself. The agent helps you think; you make the call.

### The "Checkpoint" Technique

For long tasks, set explicit checkpoints:

"Implement the payment system. After each of these checkpoints, stop and show me what you've done:
1. Database schema and models
2. API endpoints (no logic yet, just routing)
3. Business logic for charge/refund
4. Webhook handlers
5. Tests"

This prevents the agent from going too far in the wrong direction.

### The "Context Refresh" Technique

When a conversation gets long:
1. Ask the agent to summarize what's been done and what's left
2. Start a new conversation
3. Paste the summary and continue

This gives you a fresh, uncluttered context window while preserving progress.

---

## 3.8 Common Pitfalls

| Pitfall | Why It Happens | Fix |
|---------|---------------|-----|
| Agent goes in circles | Unclear success criteria | Define concrete done conditions |
| Agent makes breaking changes | Didn't understand existing code | Have it read files before editing |
| Agent ignores conventions | No CLAUDE.md or incomplete one | Write a thorough CLAUDE.md |
| Agent hallucinates imports | Working from memory, not reality | Ensure it greps for actual exports |
| Conversation gets slow | Context window is full | Start fresh with a summary |
| Agent is too cautious | Asking permission for everything | Adjust permission settings |
| Agent is too aggressive | Making changes without checking | Add checkpoints and reviews |

---

## 3.9 Claude Code Commands Reference

Claude Code comes with built-in **slash commands** — shortcuts you type directly in the conversation to trigger specific workflows. These are not prompts; they are first-class features that invoke specialized behavior.

### Built-In Slash Commands

| Command | What It Does | When to Use It |
|---------|-------------|----------------|
| `/help` | Show available commands and usage info | When you forget what's available |
| `/commit` | Stage changes, generate a commit message, and commit | After completing a piece of work |
| `/review-pr` | Review a pull request for issues and suggestions | Before merging a PR |
| `/clear` | Clear the conversation context | When context is stale or cluttered |
| `/compact` | Compress conversation history to free context space | During long sessions approaching context limits |
| `/init` | Generate a CLAUDE.md for the current project | First time setting up a project for Claude Code |
| `/fast` | Toggle fast mode (same model, faster output) | When speed matters more than thoroughness |
| `/config` | View or change Claude Code settings | Adjusting permissions, theme, model |
| `/memory` | View or edit your memory files | Reviewing what Claude remembers |
| `/loop` | Run a command on a recurring interval | Polling, monitoring, repeated checks |
| `/schedule` | Create scheduled remote agents (cron-based) | Automated recurring tasks |

### How Slash Commands Work Under the Hood

Slash commands are not magic — they expand into specialized prompts with specific tool access and instructions. When you type `/commit`:

```
1. Claude Code intercepts the "/" prefix
2. Loads the "commit" skill definition (prompt + constraints)
3. The skill tells Claude exactly how to:
   - Run git status and git diff
   - Analyze changes
   - Draft a commit message following conventions
   - Stage files and create the commit
4. Claude executes using its normal tools (Bash, Read, etc.)
```

This is the same mechanism you can use to create your own commands.

### Command Tips

**Chaining with arguments:** Some commands accept arguments.
```
/review-pr 142          ← review PR #142
/loop 5m /commit        ← auto-commit every 5 minutes
/schedule               ← interactive scheduled task setup
```

**Using `/compact` strategically:** Don't wait until you hit context limits. If you've finished one phase of work and are starting another, `/compact` gives you a clean slate while preserving key context.

**Using `/init` on existing projects:** Even if you already have a CLAUDE.md, running `/init` can suggest improvements based on what Claude discovers in the codebase.

---

## 3.10 Creating Custom Commands

This is one of Claude Code's most powerful and least-known features. You can create your own slash commands that anyone on your team can use.

### How Custom Commands Work

Custom commands live in `.claude/commands/` in your project root. Each command is a markdown file where the filename becomes the command name.

```
your-project/
├── .claude/
│   └── commands/
│       ├── review-security.md    ← becomes /project:review-security
│       ├── add-endpoint.md       ← becomes /project:add-endpoint
│       └── check-coverage.md     ← becomes /project:check-coverage
├── src/
└── CLAUDE.md
```

Commands in your project's `.claude/commands/` are prefixed with `/project:` when you invoke them.

You can also create **personal commands** that work across all projects:

```
~/.claude/commands/
├── morning-standup.md     ← becomes /user:morning-standup
├── end-of-day.md          ← becomes /user:end-of-day
└── quick-refactor.md      ← becomes /user:quick-refactor
```

Personal commands are prefixed with `/user:`.

### Anatomy of a Custom Command

A command file is just a markdown prompt. Here's a real example:

**`.claude/commands/review-security.md`**
```markdown
Review the current codebase for security vulnerabilities.

Focus on:
1. SQL injection in any database queries
2. XSS vulnerabilities in rendered output
3. Authentication/authorization bypasses
4. Hardcoded secrets or credentials
5. Insecure dependencies (check package.json)

For each issue found:
- State the file and line number
- Explain the vulnerability
- Provide a fix

If no issues are found, confirm the areas you checked.
```

That's it. When you type `/project:review-security`, Claude executes this prompt with full access to its tools.

### Using Arguments in Custom Commands

Commands can accept dynamic input using the `$ARGUMENTS` placeholder:

**`.claude/commands/add-endpoint.md`**
```markdown
Create a new REST API endpoint: $ARGUMENTS

Follow the existing patterns in src/routes/. Specifically:
1. Read an existing route file to understand the pattern
2. Create the new endpoint following that exact pattern
3. Add input validation using our Zod schemas
4. Write tests following the pattern in __tests__/
5. Add the route to the router in src/routes/index.ts

Run the tests to make sure everything passes.
```

Usage:
```
/project:add-endpoint GET /api/users/:id - returns user profile with email and role
```

The text after the command name replaces `$ARGUMENTS`.

### Real-World Custom Command Examples

**`.claude/commands/morning-review.md`** — Start your day:
```markdown
Good morning. Help me get up to speed:

1. Run `git log --oneline --since="yesterday" --all` to see what changed overnight
2. Check for any open PRs that need my review
3. Look at any TODO comments added in the last 24 hours
4. Summarize: what changed, what needs attention, what's blocked

Keep it brief — bullet points only.
```

**`.claude/commands/prep-pr.md`** — Pre-PR checklist:
```markdown
Prepare this branch for a pull request:

1. Run all tests and report results
2. Run the linter and fix any issues
3. Check for any files that shouldn't be committed (env files, logs, build artifacts)
4. Review the diff against main for any obvious issues
5. Suggest a PR title and description based on the changes

Do NOT commit or push — just report what you find.
```

**`.claude/commands/explain-area.md`** — Onboard yourself to unfamiliar code:
```markdown
Explain this area of the codebase: $ARGUMENTS

1. Find the relevant files
2. Read them and understand the architecture
3. Explain:
   - What this code does (purpose)
   - How data flows through it
   - Key abstractions and patterns used
   - How it connects to other parts of the system
   - Any potential gotchas or non-obvious behavior

Write the explanation for someone who is familiar with programming but new to this codebase.
```

### Making Commands Team-Wide

Since `.claude/commands/` lives in your project, commit it to git. Your entire team gets the same commands:

```bash
git add .claude/commands/
git commit -m "Add team custom commands for Claude Code"
```

This is powerful — you're essentially creating team-specific AI workflows that are version-controlled, reviewable, and evolve with your project.

---

## 3.11 Hooks: Event-Driven Automation

Hooks let you run shell commands automatically when Claude Code does something. They're configured in your settings and fire on specific events.

### What Hooks Do

```
Claude calls a tool → Hook fires → Shell command runs → Result shown to Claude
```

Hooks can:

- **Validate** actions before they happen (pre-hooks)
- **React** to actions after they happen (post-hooks)
- **Block** actions that violate your rules
- **Transform** behavior by injecting feedback

### Hook Configuration

Hooks are defined in your Claude Code settings (`settings.json`). You can set them at the project level (`.claude/settings.json`) or user level (`~/.claude/settings.json`).

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/check-edit-allowed.sh \"$TOOL_INPUT\""
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "command": "bash ./scripts/after-bash.sh \"$TOOL_INPUT\" \"$TOOL_OUTPUT\""
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "command": "bash ./scripts/notify.sh \"$MESSAGE\""
      }
    ]
  }
}
```

### Hook Events

| Event | When It Fires | Use Case |
| ----- | ------------- | -------- |
| `PreToolUse` | Before a tool executes | Block dangerous operations, validate inputs |
| `PostToolUse` | After a tool executes | Log actions, send notifications, trigger follow-ups |
| `Notification` | When Claude sends a notification | Alert you via Slack/email when a task completes |
| `Stop` | When Claude finishes a response | Run cleanup, validation, or formatting |

### Hook Matchers

The `matcher` field filters which tool triggers the hook:

- `"Edit"` — fires only on Edit tool calls
- `"Bash"` — fires only on Bash tool calls
- `"Write"` — fires only on Write tool calls
- `""` (empty) — fires on ALL tool calls

### Practical Hook Examples

### Example 1: Prevent edits to protected files

```bash
#!/bin/bash
# scripts/check-edit-allowed.sh
# PreToolUse hook for Edit — blocks changes to migration files

FILE_PATH=$(echo "$1" | jq -r '.file_path // empty')
if [[ "$FILE_PATH" == *"/migrations/"* ]]; then
  echo "BLOCKED: Cannot edit migration files without explicit approval."
  echo "Create a new migration instead: npm run migration:create"
  exit 1
fi
```

A non-zero exit code blocks the tool call. Claude sees the message and adjusts.

### Example 2: Auto-lint after file edits

```bash
#!/bin/bash
# scripts/after-edit.sh
# PostToolUse hook for Edit — runs linter on changed file

FILE_PATH=$(echo "$1" | jq -r '.file_path // empty')
if [[ "$FILE_PATH" == *.ts ]] || [[ "$FILE_PATH" == *.tsx ]]; then
  npx eslint --fix "$FILE_PATH" 2>&1
fi
```

### Example 3: Notify Slack when a long task finishes

```bash
#!/bin/bash
# scripts/notify-slack.sh
# Notification hook — posts to Slack

curl -s -X POST "$SLACK_WEBHOOK_URL" \
  -H 'Content-Type: application/json' \
  -d "{\"text\": \"Claude Code finished: $1\"}"
```

### Hooks vs. CLAUDE.md Rules

| Aspect | CLAUDE.md | Hooks |
| ------ | --------- | ----- |
| **Enforcement** | Soft (instructions Claude should follow) | Hard (shell commands that block/modify) |
| **Flexibility** | Claude interprets with judgment | Deterministic — code runs, pass/fail |
| **Scope** | Behavioral guidance | Specific tool-level actions |
| **When to use** | Conventions, preferences, patterns | Guardrails, automation, integration |

Use both together: CLAUDE.md for "prefer X over Y" guidance, hooks for "never allow Z" enforcement.

---

## 3.12 Building Custom Skills with MCP

Module 02 introduced MCP conceptually. Here's how to actually build and use MCP servers to extend Claude Code with new capabilities.

### What You're Building

An MCP server is a small program that exposes **tools** (functions Claude can call) and/or **resources** (data Claude can read) over a standardized protocol.

```
Claude Code ←→ MCP Protocol (JSON-RPC over stdio) ←→ Your MCP Server
                                                       ├── tool: search_jira
                                                       ├── tool: deploy_staging
                                                       └── resource: team_oncall
```

### Setting Up an MCP Server

### Step 1: Install the MCP SDK

```bash
# TypeScript (most common)
npm init -y
npm install @modelcontextprotocol/sdk

# Python
pip install mcp
```

### Step 2: Build a minimal server

Here's a complete TypeScript MCP server that provides a "lookup_user" tool:

```typescript
// server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "team-tools",
  version: "1.0.0",
});

// Define a tool
server.tool(
  "lookup_user",
  "Look up a team member by name or email",
  {
    query: z.string().describe("Name or email to search for"),
  },
  async ({ query }) => {
    // In reality, this would hit your company's API
    const users: Record<string, { role: string; team: string }> = {
      "sarah": { role: "Tech Lead", team: "Platform" },
      "james": { role: "Senior SWE", team: "Payments" },
    };

    const key = query.toLowerCase();
    const user = users[key];

    return {
      content: [
        {
          type: "text",
          text: user
            ? `${query}: ${user.role} on ${user.team}`
            : `No user found matching "${query}"`,
        },
      ],
    };
  }
);

// Start the server
const transport = new StdioServerTransport();
await server.connect(transport);
```

### Step 3: Register with Claude Code

Add the server to your Claude Code settings (`.claude/settings.json` for project-level, or `~/.claude/settings.json` for global):

```json
{
  "mcpServers": {
    "team-tools": {
      "command": "node",
      "args": ["./mcp-servers/team-tools/server.js"]
    }
  }
}
```

Now Claude Code can call `lookup_user` like any other tool.

### Python MCP Server Example

```python
# server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("deploy-tools")

@mcp.tool()
def check_deploy_status(environment: str) -> str:
    """Check the deployment status for a given environment.

    Args:
        environment: The environment to check (staging, production)
    """
    # In reality, hit your CI/CD API
    statuses = {
        "staging": "✓ Deployed v2.3.1 at 2026-03-25 14:30 UTC",
        "production": "✓ Deployed v2.3.0 at 2026-03-24 09:00 UTC",
    }
    return statuses.get(environment, f"Unknown environment: {environment}")

@mcp.tool()
def trigger_deploy(environment: str, version: str) -> str:
    """Trigger a deployment to the specified environment.

    Args:
        environment: Target environment (staging only — production requires manual approval)
        version: Version tag to deploy (e.g., v2.3.1)
    """
    if environment == "production":
        return "BLOCKED: Production deploys require manual approval in the CI dashboard."
    return f"Deploy triggered: {version} → {environment}. ETA: ~3 minutes."

if __name__ == "__main__":
    mcp.run(transport="stdio")
```

Register it:

```json
{
  "mcpServers": {
    "deploy-tools": {
      "command": "python",
      "args": ["./mcp-servers/deploy-tools/server.py"]
    }
  }
}
```

### Adding Resources to MCP Servers

Resources expose read-only data that Claude can access:

```typescript
server.resource(
  "oncall-schedule",
  "oncall://current",
  async (uri) => ({
    contents: [
      {
        uri: uri.href,
        mimeType: "text/plain",
        text: "Current oncall: Sarah (Platform), James (Payments). Escalation: #oncall-help in Slack.",
      },
    ],
  })
);
```

Claude can read this with: "Check who's on call right now."

### MCP Server Ideas for Your Team

| Server | Tools It Provides | Value |
| ------ | ----------------- | ----- |
| **Jira/Linear** | `search_tickets`, `create_ticket`, `update_status` | Link coding work to project tracking |
| **Deploy** | `check_status`, `trigger_deploy`, `rollback` | Deploy from the conversation |
| **Monitoring** | `check_alerts`, `query_metrics`, `get_logs` | Debug without leaving Claude Code |
| **Wiki/Docs** | `search_docs`, `get_runbook` | Access team knowledge during coding |
| **Feature Flags** | `list_flags`, `toggle_flag`, `check_flag` | Manage feature flags while implementing |
| **Database** | `run_query` (read-only!), `describe_table` | Explore data during development |

### MCP Security Best Practices

1. **Principle of least privilege**: Only expose what's needed. A deploy server shouldn't also read databases.
2. **Read-only by default**: Start with query/read tools. Add write/modify tools only when needed.
3. **Input validation**: Validate all inputs with schemas (Zod, Pydantic). Never pass raw input to shell commands or SQL.
4. **Audit logging**: Log every tool call. You want to know what Claude did and when.
5. **Environment separation**: Use different MCP configs for dev vs. prod access.

---

## 3.13 Putting It All Together: Your Custom Workflow

Here's how commands, hooks, and MCP servers combine into a personalized workflow:

```text
Custom Commands          → What Claude does (your team's playbooks)
Hooks                    → When and how Claude does it (guardrails & automation)
MCP Servers              → What Claude can access (your team's tools & data)
CLAUDE.md                → How Claude behaves (conventions & preferences)
Memory                   → What Claude remembers (your context & corrections)
```

### Example: A full team setup

```text
your-project/
├── .claude/
│   ├── settings.json          ← hooks + MCP server config
│   └── commands/
│       ├── deploy-staging.md  ← /project:deploy-staging
│       ├── review-security.md ← /project:review-security
│       └── morning-review.md  ← /project:morning-review
├── mcp-servers/
│   ├── deploy-tools/          ← deploy + rollback capabilities
│   └── monitoring/            ← metrics + alerts + logs
├── scripts/
│   ├── pre-edit-guard.sh      ← hook: block edits to protected files
│   └── post-edit-lint.sh      ← hook: auto-lint after edits
├── CLAUDE.md                  ← project conventions
└── src/
```

A new team member clones the repo, installs Claude Code, and immediately has:

- Team-specific commands (`/project:deploy-staging`)
- Guardrails that prevent common mistakes (hooks)
- Access to team tools (MCP servers)
- Coding conventions (CLAUDE.md)

This is how you scale AI-native development across a team.

---

## Self-Check Questions

- [ ] What are the 5 layers of the agent capability stack?
- [ ] Write a CLAUDE.md for a project you're working on
- [ ] Name 3 multi-agent patterns and when to use each
- [ ] How would you set up a task that runs while you sleep?
- [ ] What's the difference between "tell" and "teach" approaches?
- [ ] Name 5 built-in slash commands and when you'd use each
- [ ] Create a custom command that automates a task you do frequently
- [ ] Explain the difference between hooks and CLAUDE.md rules — when would you use each?
- [ ] Design an MCP server for a tool your team uses — what tools would it expose?
- [ ] How do custom commands, hooks, MCP servers, CLAUDE.md, and memory work together?

→ **Next: [Module 04 - Career Growth & Evergreen Learning](04-career-growth.md)**
