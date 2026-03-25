# Lab 03: Multi-Agent Project Orchestration

> Estimated time: 90-120 minutes

## Objective

Learn to use multiple AI agents working on the same project simultaneously. Experience the power — and coordination challenges — of parallel AI work.

---

## Setup

Create a new project for a simple task management API:
```bash
mkdir ~/lab-03-taskapi && cd ~/lab-03-taskapi
npm init -y
npm install express
npm install -D typescript @types/express @types/node tsx
npx tsc --init
```

Start Claude Code: `claude`

---

## Exercise 1: Scout-Build-Verify Pattern

### Step 1: Scout
```
I want to build a task management REST API with Express + TypeScript.
Before writing any code, research and tell me:
1. What's the recommended project structure for an Express + TS API in 2026?
2. What's the best pattern for error handling?
3. What testing approach should we use?
Don't write any code yet — just give me recommendations.
```

**Your job:** Read the recommendations. Decide which to follow.

### Step 2: Build
```
Build the task management API with this structure:
[paste your chosen structure]

Endpoints:
- POST /tasks { title, description, priority } → created task
- GET /tasks?status=open&priority=high → filtered list
- GET /tasks/:id → single task
- PATCH /tasks/:id { status, title, ... } → updated task
- DELETE /tasks/:id → 204

Use in-memory storage. Include proper error handling and input validation.
```

### Step 3: Verify
```
Review the implementation you just created:
1. Run any tests
2. Check for missing error handling
3. Verify all edge cases are covered
4. Look for any inconsistencies in the API contract
Report issues but don't fix them yet — I want to review the report first.
```

**Your job:** Review the verify report. Decide which issues to fix and in what order.

---

## Exercise 2: Parallel Workers (The Core Multi-Agent Pattern)

This is the key exercise. You'll use Claude Code's Agent tool to spawn parallel workers.

### The Task

Your task API needs 3 independent features:
1. **Search**: Full-text search across task titles and descriptions
2. **Pagination**: `GET /tasks?page=1&limit=10` with total count
3. **Export**: `GET /tasks/export?format=csv` to download tasks as CSV

### Sequential Approach (Baseline)
Time yourself implementing all 3 features one at a time:
```
Add full-text search to GET /tasks: ?q=searchterm should filter by title and description (case-insensitive partial match)
```
Then:
```
Add pagination to GET /tasks: ?page=1&limit=10. Response should include { tasks, total, page, limit, totalPages }
```
Then:
```
Add CSV export: GET /tasks/export?format=csv should return a CSV file with all tasks
```

**Record the total time.**

### Parallel Approach
Now try asking Claude to spawn sub-agents for each:
```
I need 3 independent features added to the task API. Please use sub-agents to work on all 3 in parallel:

1. Agent A: Add full-text search to GET /tasks — ?q=searchterm filters by title and description (case-insensitive)
2. Agent B: Add pagination to GET /tasks — ?page=1&limit=10, response includes total count and page metadata
3. Agent C: Add CSV export — GET /tasks/export?format=csv returns all tasks as downloadable CSV

Each feature is independent and touches different parts of the code. Use worktree isolation so they don't conflict.
```

**Observe:**
- How did Claude handle the parallel work?
- Were there any merge conflicts?
- How does the total time compare?

---

## Exercise 3: The Review Agent Pattern

After building features, use a dedicated "review" pass:

```
Act as a senior code reviewer. Review ALL changes made in this session:
1. Code quality: naming, structure, readability
2. Error handling: are all error paths covered?
3. Security: any input validation gaps?
4. Performance: any obvious N+1 or memory issues?
5. API consistency: do all endpoints follow the same patterns?

Format as a code review with file:line references and severity (critical/warning/nit).
```

**Your job:** Triage the review. Fix criticals, consider warnings, ignore nits (or fix them if you agree).

---

## Exercise 4: Competing Approaches

A task API needs a filtering system. Try two approaches simultaneously:

```
I need to add advanced filtering to GET /tasks. Implement two different approaches so I can compare:

Approach A: Query string based — /tasks?priority=high&status=open&created_after=2026-01-01
Approach B: POST body based — POST /tasks/search { filters: { priority: "high", status: "open", createdAfter: "2026-01-01" } }

Use separate agents in worktrees for each. Include tests.
```

**Your decision:** Which approach is better for your use case? Why?

**Factors to evaluate:**
- Cacheability (GET is cacheable, POST is not)
- Complexity of filter expressions
- REST convention compliance
- Client-side usability

---

## Exercise 5: Orchestrated Pipeline

Build a mini CI pipeline using Claude:

```
Run this pipeline in sequence:
1. Lint check — run any linting and report issues
2. Type check — run tsc --noEmit and report issues
3. Test — run all tests and report results
4. Security scan — check for common vulnerabilities in the code
5. Summary — give me a single pass/fail status with key issues

If any step fails, still continue to the next step (I want full results).
```

---

## Reflection

Journal entry:
1. When did parallel agents save time vs. add complexity?
2. How did you handle conflicts between parallel agent outputs?
3. What's your strategy for deciding when to go sequential vs. parallel?
4. How did the review agent's findings compare to what you'd catch yourself?
