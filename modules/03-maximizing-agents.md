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

### Memory System

Claude Code maintains persistent memory between conversations. Use it for:
- Your role and expertise level
- Feedback on agent behavior (what to do/avoid)
- Project context that isn't in code (deadlines, team decisions)
- External resource locations

**Pro tip:** After a productive session, explicitly tell Claude "Remember that X approach worked well for Y" — this gets saved and improves future sessions.

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

## Self-Check Questions

- [ ] What are the 5 layers of the agent capability stack?
- [ ] Write a CLAUDE.md for a project you're working on
- [ ] Name 3 multi-agent patterns and when to use each
- [ ] How would you set up a task that runs while you sleep?
- [ ] What's the difference between "tell" and "teach" approaches?

→ **Next: [Module 04 - Career Growth & Evergreen Learning](04-career-growth.md)**
