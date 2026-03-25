# Lab 04: Building an Autonomous Coding Pipeline

> Estimated time: 2-3 hours (including setup and overnight run)

## Objective

Set up workflows where AI agents do productive work with minimal or no supervision — including while you're sleeping.

---

## Exercise 1: The Overnight Feature Builder

### Setup: A Real-ish Project

Create a project with enough structure to be meaningful:

```bash
mkdir ~/lab-04-webapp && cd ~/lab-04-webapp
```

Ask Claude to scaffold:
```
Scaffold a simple Express + TypeScript web app with:
- src/routes/ for API routes
- src/models/ for data models (in-memory)
- src/middleware/ for middleware
- tests/ for tests
- A user model with CRUD endpoints already working
- A CLAUDE.md with build/test/lint commands
- At least 5 passing tests

This is a foundation for autonomous agent work — make sure tests are solid.
```

### The Autonomous Task

Create a file `tasks.md` with well-defined tasks:

```markdown
# Tasks for Autonomous Agent

## Task 1: Add a "projects" resource
- Model: { id, name, description, ownerId (references user), status: active|archived, createdAt }
- CRUD endpoints: POST/GET/GET:id/PATCH/DELETE at /projects
- Validate ownerId references an existing user
- Write tests for all endpoints and edge cases
- Follow the same patterns as the user routes

## Task 2: Add request logging middleware
- Log: timestamp, method, path, status code, response time (ms)
- Log to stdout in JSON format
- Add to all routes
- Write a test that verifies logging output

## Task 3: Add a health check endpoint
- GET /health returns { status: "ok", uptime: seconds, timestamp: ISO string }
- No authentication required
- Write a test
```

### Run It

```
Read tasks.md and implement all 3 tasks in order.
For each task:
1. Read the relevant existing code first
2. Implement the feature
3. Run all tests (existing + new)
4. Only move to the next task if all tests pass
5. If tests fail, fix them before moving on

Do not ask me questions — make reasonable decisions based on the existing codebase patterns.
```

**Then step away.** Come back and review the results.

### Review Checklist
- [ ] All 3 tasks completed?
- [ ] All tests pass?
- [ ] Code follows existing patterns?
- [ ] No unnecessary changes to existing code?
- [ ] Error handling is consistent?

---

## Exercise 2: Scheduled Maintenance Agent

Set up a recurring task that keeps your codebase healthy:

```
Set up a process that does the following (I want to be able to run this weekly):

1. Check for any TODO/FIXME/HACK comments in the codebase
2. List any tests that are skipped or disabled
3. Check if any dependencies have known vulnerabilities (npm audit)
4. Check for unused exports or dead code
5. Generate a report in maintenance-report.md

Format the report with sections, counts, and specific file:line references.
```

**Automation:** You can schedule this with Claude Code's scheduled tasks or a simple cron:
```bash
# Weekly maintenance check (Sunday midnight)
0 0 * * 0 cd ~/lab-04-webapp && claude --print "Run the maintenance check and save results to maintenance-report.md"
```

---

## Exercise 3: PR Review Agent

Simulate an automated PR review workflow:

### Step 1: Create a branch with changes
```bash
git checkout -b feature/add-comments
```

Ask Claude to add a commenting system (deliberately skip some things — we want the reviewer to catch issues):
```
Add a comments system. Comments belong to users and projects.
Quick implementation — don't worry about pagination or validation for now.
Just get the basic CRUD working.
```

### Step 2: Run the review agent
```
Review all changes on the current branch compared to main.
Act as a thorough PR reviewer. Check for:

1. **Correctness**: Logic errors, missing error handling
2. **Consistency**: Does new code match existing patterns?
3. **Security**: Input validation, injection risks
4. **Testing**: Are new features tested? Edge cases covered?
5. **Performance**: Any obvious bottlenecks?
6. **Missing pieces**: What was skipped that should be addressed?

Format as a PR review with:
- Overall assessment (approve / request changes)
- Critical issues (must fix)
- Suggestions (should fix)
- Nits (optional)
```

**Observe:** Does the agent catch the deliberately skipped validation and pagination?

---

## Exercise 4: The Full Autonomous Cycle

Combine everything into a single autonomous workflow:

```
I'm going to describe a feature. I want you to:
1. Plan the implementation (show me the plan, then proceed)
2. Implement it with tests
3. Run all tests (fix any failures)
4. Self-review the code (check for issues)
5. Fix any issues found in self-review
6. Run tests again
7. Give me a final summary

Feature: Add a "tags" system
- Users can create tags (name, color)
- Projects can have multiple tags
- GET /projects can filter by tag: ?tag=important
- Tags endpoint: CRUD at /tags
- Many-to-many relationship between projects and tags
```

**Measure:**
- How many iterations did the agent do?
- Did self-review catch real issues?
- What was the quality of the final output?

---

## Exercise 5: Designing for Autonomy

Based on everything you've learned, create an "Autonomous Agent Playbook" for your real projects:

```markdown
# Autonomous Agent Playbook

## Pre-requisites for autonomous work
- [ ] CLAUDE.md is comprehensive and up-to-date
- [ ] Test suite has > X% coverage on critical paths
- [ ] CI/CD pipeline catches type errors, lint issues, test failures
- [ ] Code patterns are consistent (agent can learn by reading)

## Task classification
| Task Type | Autonomy Level | Example |
|-----------|---------------|---------|
| Bug fix with failing test | High — agent can fix and verify | "Fix the 500 error in /users/:id" |
| New endpoint following pattern | High — if pattern is clear | "Add DELETE /projects/:id" |
| New feature with spec | Medium — review the plan first | "Add comments system per spec" |
| Refactor | Medium — review approach, then let it run | "Extract auth to middleware" |
| Architecture change | Low — plan together, implement together | "Switch from REST to GraphQL" |
| Security-sensitive work | Low — review every change | "Implement payment processing" |

## Overnight task template
[Task description]
- Success criteria: [measurable]
- Scope boundary: [what NOT to touch]
- Fallback: [what to do if stuck — stop, not guess]
- Verification: [how to prove it worked]
```

Save this to `~/ai-native-coding/references/autonomous-playbook.md`.

---

## Reflection

Journal entry:
1. What types of tasks are safe to run autonomously today?
2. What's your personal trust threshold — when do you need to watch vs. walk away?
3. How does test coverage affect your trust in autonomous work?
4. What would need to change for you to let agents merge code without review?
