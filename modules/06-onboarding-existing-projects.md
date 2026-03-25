# Module 06: Onboarding Existing Projects into AI-Native Development

> You just inherited 3 repos, a wiki nobody updates, a CI pipeline held together with tape, and zero AI tooling. Now what?

---

## 6.1 The Reality

Most projects you'll encounter are NOT AI-ready. They have:
- Sparse or outdated documentation
- No CLAUDE.md
- Inconsistent coding patterns
- Incomplete test coverage
- Tribal knowledge locked in people's heads
- Build steps that "just work" on Dave's machine

This is normal. The process of making a project AI-ready is also the process of making it **well-understood** — which benefits everyone, not just the AI.

**The key insight: AI onboarding is project onboarding on steroids.** Everything you do to help the AI understand the project also helps new human developers. You're investing in comprehensibility.

---

## 6.2 The Onboarding Phases

```
Phase 1: Reconnaissance     → Understand what exists (don't change anything)
Phase 2: Foundation          → Get the basics running and documented
Phase 3: AI Infrastructure   → CLAUDE.md, memory, conventions
Phase 4: Trust Building      → Small wins that prove the setup works
Phase 5: Full Integration    → AI-native workflow for daily development
```

**Critical rule:** Don't try to "fix everything first." You'll never start. The goal is a working AI setup in days, not a perfect codebase in months.

---

## 6.3 Phase 1: Reconnaissance (Day 1)

Your first day with an unknown project. Don't write code. Don't set up tools. Just explore.

### Step 1: The 5-Minute Scan

Open Claude Code in the project root and ask:

```
I just inherited this project and I'm not familiar with it.
Give me a quick overview:
1. What language(s) and frameworks does this use?
2. What's the directory structure? (top 2 levels)
3. Is there a README, docs folder, or wiki?
4. What build system / package manager?
5. What does the CI/CD look like? (.github/workflows, Jenkinsfile, etc.)
6. What test framework? How many tests exist?
7. What's the git activity like? (recent commits, active branches)
```

This single prompt gives you more context in 60 seconds than an hour of manual exploration.

### Step 2: The Dependency Map

```
Analyze the dependency graph of this project:
1. What are the key external dependencies? Group by purpose (web framework, database, testing, etc.)
2. Are there any internal/private packages? What do they do?
3. Are dependencies up to date? Any obviously outdated or deprecated ones?
4. What's the overall tech stack story this tells?
```

### Step 3: The Architecture Scan

```
I need to understand the architecture of this project. Analyze:
1. What's the entry point? How does the app start?
2. What are the major modules/domains? How do they relate?
3. Where does data flow in and out? (APIs, databases, queues, files)
4. What patterns are used? (MVC, service layer, repository pattern, etc.)
5. Where is configuration managed? (env vars, config files, etc.)

Draw me an ASCII diagram of the high-level architecture.
```

### Step 4: The "Where Is Everything?" Map

```
Help me build a map of where important things live in this codebase:
- Where are API route/endpoint definitions?
- Where are database models/schemas?
- Where is authentication/authorization handled?
- Where are tests? How are they organized?
- Where is business logic (vs. framework glue)?
- Where is configuration/environment handling?
- Where are the build/deploy scripts?
- Any unusual or non-standard locations I should know about?
```

### Step 5: The Pain Points Scan

```
Look for red flags in this codebase:
1. Files over 500 lines (complexity hotspots)
2. TODO/FIXME/HACK comments (known technical debt)
3. Commented-out code blocks (uncertainty/abandoned work)
4. Files with no test coverage that probably should have tests
5. Any hardcoded secrets, URLs, or credentials (security issues)
6. Inconsistent patterns (different ways of doing the same thing)

Don't fix anything. Just report what you find.
```

### What to Write Down After Phase 1

Create a personal notes file (not committed — this is for you):

```markdown
# Project: [name] — First Impressions (YYYY-MM-DD)

## What I understand
- [things that are clear]

## What I don't understand
- [things that are confusing — these become your learning priorities]

## What worries me
- [red flags, risks, tech debt]

## Key people to ask
- [who built this? who knows the most? check git blame]

## First instinct for improvements
- [hold these — don't act yet]
```

---

## 6.4 Phase 2: Foundation (Days 2-3)

Now you make the project runnable and testable — the minimum bar for AI-native work.

### Step 1: Can I Build It?

```
Help me get this project building on my machine:
1. Read any setup instructions (README, CONTRIBUTING, docs/)
2. What environment/tools do I need? (Node version, Python version, Docker, etc.)
3. What are the exact commands to install dependencies and build?
4. Try to identify any missing setup steps that aren't documented.
```

If the build fails (it often will):
```
The build is failing with this error: [paste error].
This is an existing project I inherited — I didn't break this.
Help me diagnose: is this a missing dependency, environment issue, or config problem?
```

**Pro tip:** Use Claude to read Dockerfiles, CI configs, and Makefiles — these often contain the "real" setup steps that the README forgot to mention.

```
Read the Dockerfile and the CI workflow files.
These often have the most accurate build instructions.
What steps do they run that aren't in the README?
```

### Step 2: Can I Test It?

```
How do I run tests in this project?
1. What test command(s) exist? (check package.json scripts, Makefile, etc.)
2. Run the tests — what's the current state? All passing?
3. How many tests are there? Roughly what coverage?
4. Are there different test suites? (unit, integration, e2e)
```

If tests fail:
```
These tests were failing before I touched anything. Categorize the failures:
1. Environment/config issues (might work in CI but not locally)
2. Genuinely broken tests (likely pre-existing bugs)
3. Flaky tests (sometimes pass, sometimes fail)
Don't fix them yet — just categorize.
```

### Step 3: Can I Run It?

```
How do I run this project locally?
1. What services does it depend on? (database, Redis, queue, etc.)
2. Is there a docker-compose.yml or similar for local dependencies?
3. What environment variables are needed? (check .env.example, CI config, Docker)
4. What's the command to start the dev server?
5. How do I verify it's running correctly?
```

### Step 4: Document What You Learned

This is where you start creating value. Everything you struggled with — document it.

```
Based on everything we've discovered about building, testing, and running this project,
draft a "Getting Started" section that would have saved me time.
Include exact commands, environment requirements, and common pitfalls.
Don't create a new file — just show me the content. I'll decide where it goes.
```

---

## 6.5 Phase 3: AI Infrastructure (Days 3-5)

Now you set up the AI coding environment. This is where the leverage starts compounding.

### Step 1: Create the CLAUDE.md

This is the single highest-ROI action. Ask Claude to help draft it based on what it's learned:

```
Based on everything you've observed about this project, draft a CLAUDE.md file.
Include:
1. Quick reference (language, framework, build/test/lint commands)
2. Architecture overview (2-3 sentences + the key directories)
3. Coding conventions you've observed (naming, patterns, file organization)
4. Known gotchas or non-obvious things
5. Common tasks and how to do them
6. Things to avoid (based on red flags we found)

Be honest about what you're not sure about — mark those as "TODO: verify with team".
```

**Example output for a messy project:**

```markdown
# CLAUDE.md — ProjectX

## Quick Reference
- Language: Python 3.11 / FastAPI
- Package manager: pip + requirements.txt (no poetry/pipenv)
- Build: N/A (no build step)
- Test: `pytest tests/`
- Lint: `ruff check .` (configured in pyproject.toml)
- Run locally: `uvicorn src.main:app --reload` (needs PostgreSQL + Redis running)

## Architecture
FastAPI monolith with service-layer pattern. API routes in `src/routes/`,
business logic in `src/services/`, database models in `src/models/`.
Background jobs via Celery + Redis in `src/tasks/`.

## Conventions (Observed)
- Route handlers are thin — they validate input and call a service
- Services raise custom exceptions (see src/exceptions.py)
- Database access through SQLAlchemy ORM, async sessions
- Pydantic schemas in `src/schemas/` — one file per domain
- Tests mirror src/ structure: `tests/routes/`, `tests/services/`

## Gotchas
- Some tests require a running PostgreSQL (not mocked) — use docker-compose
- The `src/legacy/` folder uses a completely different pattern — don't follow it
- Environment variables are loaded in `src/config.py` — always add new ones there
- TODO: verify with team — is `src/utils/helpers.py` still used? Looks dead.

## Things to Avoid
- Don't add new code to `src/legacy/` — it's being phased out
- Don't use raw SQL — use SQLAlchemy ORM
- Don't skip type hints — existing code is ~70% typed, goal is 100%
```

### Step 2: Set Up Memory for the Project

Tell Claude important context that doesn't belong in CLAUDE.md:

```
Remember these things about this project:
- It's owned by the Platform team, originally built by [person] who left
- The legacy/ folder is from v1, scheduled for removal in Q3
- Tests marked @pytest.mark.slow hit real databases and take 10+ minutes
- The CI pipeline is in GitHub Actions, deploys to AWS ECS
- There's a Confluence wiki at [URL] but it's mostly outdated
```

### Step 3: Set Up Git Hooks for AI Feedback

The agent needs fast feedback loops. Set up pre-commit hooks if they don't exist:

```
Check if this project has pre-commit hooks or a pre-commit config.
If not, suggest a minimal setup that runs:
1. Linting (whatever linter the project already uses)
2. Type checking (if applicable)
3. Fast unit tests (not the slow integration ones)

Keep it minimal — only things that already exist in the project's toolchain.
Don't add new tools.
```

### Step 4: Map the Multi-Repo Landscape (If Applicable)

If this project is part of a larger ecosystem:

```
This project seems to depend on / be depended on by other repos.
Help me map the ecosystem:
1. Check package.json / requirements.txt for internal/private packages
2. Check for references to other services (API URLs, service names)
3. Check CI/CD for cross-repo triggers or deployment dependencies
4. Check for shared config or infrastructure references
```

Then create the topology map from Module 05.

---

## 6.6 Phase 4: Trust Building (Week 1-2)

Don't start with "rewrite the auth system." Start with small, safe, verifiable tasks that build confidence in the AI setup.

### The First Win Ladder

**Win 1: Fix a typo or update a comment**
The smallest possible change. Proves the edit → test → commit loop works.

```
Find any typos, outdated comments, or incorrect documentation in the codebase.
Fix the most obvious one. Run tests to make sure nothing breaks.
```

**Win 2: Add a missing test**
Find untested code and add a test for it. Pure addition — no existing code changes.

```
Find a function in src/services/ that has no corresponding test.
Write a test for it. Run the full test suite to verify.
```

**Win 3: Small bug fix**
Find a real issue and fix it. This tests the full workflow.

```
Look at recent git commits with "fix" in the message to understand common bug patterns.
Check if there are any open issues or TODOs that reference bugs.
Pick the simplest one and fix it with a test.
```

**Win 4: Add a small feature following existing patterns**
The real test — can the AI extend the codebase meaningfully?

```
[Describe a small, well-defined feature]
Follow the existing patterns in the codebase. Read similar implementations first.
Write tests. Run the full suite.
```

**Win 5: Refactor one module**
Now you're testing the AI's understanding of the codebase's design.

```
The [specific module] has [specific issue — duplication, complexity, etc.].
Refactor it to [specific improvement].
Don't change behavior — all existing tests must still pass.
Add tests if the refactored code has paths that weren't tested before.
```

### Tracking Trust Metrics

Keep a simple log:

```markdown
## AI Trust Log

| Date | Task | Outcome | Issues Found in Review | Trust Delta |
|------|------|---------|----------------------|-------------|
| 03-25 | Fix typo in README | Clean | None | +1 |
| 03-25 | Add test for UserService.create | Clean after 1 iteration | Missed edge case, caught in review | +0.5 |
| 03-26 | Fix pagination bug | Required 2 iterations | First attempt broke another test | +0.5 |
| 03-27 | Add email notification feature | Clean after feedback | Followed wrong pattern initially, self-corrected | +1 |
```

---

## 6.7 Phase 5: Full Integration (Week 2+)

Once trust is established, AI becomes part of the daily workflow.

### Daily Pattern

```
Morning:
  1. Pull latest changes
  2. Open Claude Code
  3. "What changed since yesterday? Summarize recent commits."
  4. Start working on your tasks with AI assistance

During work:
  5. Use AI for implementation (describe intent, review output)
  6. Use AI for debugging ("this test fails with X, the relevant code is in Y")
  7. Use AI for code review ("review my changes before I commit")

End of day:
  8. Set up overnight tasks for well-defined, low-risk work
  9. "Summarize what we accomplished today" (for your own records)
```

### Continuous CLAUDE.md Improvement

Your CLAUDE.md should evolve weekly as you learn more:

```
Update CLAUDE.md with what I've learned this week:
- The auth middleware in src/middleware/auth.ts expects X header format
- Integration tests need REDIS_URL set even for non-Redis tests (known quirk)
- The /admin routes have a completely different error format (legacy, don't follow)
```

### Team Adoption (If Applicable)

When you're ready to share:

1. **Commit the CLAUDE.md** — it helps everyone, including humans
2. **Show, don't tell** — demo a task done with AI in a team meeting
3. **Start with willing adopters** — find the teammate who's curious, not skeptical
4. **Share your trust log** — concrete evidence of what works and what doesn't

---

## 6.8 The "I Don't Know Anything" Playbook

For the worst case — you've been dropped into a project with zero context.

### Step 1: Let the AI explore for you

```
I have zero context about this project. I need you to be my guide.
Explore the codebase systematically and give me a briefing:

1. Start with the most important files: README, package.json/requirements.txt,
   Dockerfile, CI config, main entry point
2. Identify the tech stack and framework
3. Map the top-level directory structure and explain each folder
4. Find the 5 most-changed files (git log) — these are likely the core
5. Find the 5 largest files — these are likely the most complex
6. Check for any onboarding docs, CONTRIBUTING.md, or wiki references

Present this as a briefing I can read in 5 minutes.
```

### Step 2: Understand by tracing a request

```
Trace the lifecycle of a typical request through this codebase:
1. Where does a request enter? (HTTP server, main handler, etc.)
2. What middleware/interceptors does it pass through?
3. How does it reach the business logic?
4. How does it interact with the database?
5. How does the response get sent back?

Pick a simple GET endpoint as the example. Show me the actual file paths and functions.
```

### Step 3: Understand by reading tests

Tests are often the best documentation:

```
Read the test files and tell me:
1. What do the tests reveal about how the system is supposed to behave?
2. What use cases are tested? (These are the "happy paths" the system supports)
3. What edge cases are tested? (These are the known tricky situations)
4. What's NOT tested? (These are the risky areas)
```

### Step 4: Understand by reading git history

```
Analyze the git history:
1. Who are the main contributors? (git shortlog -sn)
2. What are the most recent 20 commit messages? (what's being worked on)
3. What files change together most often? (these are coupled)
4. Are there any long-lived branches? (ongoing work)
5. When was the last release/tag? What was in it?
```

### Step 5: Build your mental model incrementally

```
Based on everything we've explored, I think the architecture is:
[write your understanding, even if rough]

Am I right? What am I missing? Correct my mental model by reading
the actual code in the areas I'm wrong about.
```

This dialogue — stating your understanding and having the AI correct it — is the fastest way to learn a codebase. It's active learning.

---

## 6.9 Handling Specific Challenges

### Challenge: No Tests

```
This project has few/no tests. Before I can use AI confidently, I need baseline coverage.

1. Identify the 5 most critical modules (most used, most changed, most complex)
2. For each, write basic "does it not crash" tests — import it, call main functions
   with typical inputs, assert basic output shape
3. These aren't thorough tests — they're a safety net for AI-assisted changes
4. Run all tests to establish the baseline.
```

Then gradually add meaningful tests as you make changes.

### Challenge: No Linter / No Type Checking

```
This project has no linting or type checking configured.
Add the minimal linter setup for [language]:
- Use the most standard/popular linter (ESLint, Ruff, etc.)
- Start with default rules — don't customize yet
- Add it as a dev dependency, not globally
- Create a minimal config file
- Run it and show me the results (but don't fix everything — just establish the baseline)
```

### Challenge: "It Works on Dave's Machine"

```
I can't get this project running locally. Let's debug systematically:

1. Read the Dockerfile / docker-compose.yml — what does the production/CI environment look like?
2. Compare that to my local environment (I'm on [OS/version])
3. Check for hardcoded paths, OS-specific commands, or assumptions
4. Check for required environment variables that aren't documented
5. Check for required services (databases, queues) that aren't documented
```

### Challenge: Outdated Documentation

```
The documentation appears outdated. Help me audit it:
1. Compare the README setup instructions to what actually works
2. Check if API documentation matches actual endpoints (read routes, compare to docs)
3. Check if architecture docs match actual structure
4. Flag everything that's wrong

Then help me update the most critical parts — setup instructions and architecture overview.
```

### Challenge: Multiple Repos, No Map

```
I have access to these repos: [list them]
I don't know how they relate to each other. Help me figure it out:

For each repo:
1. What does it do? (read the entry point and README)
2. What does it depend on? (check for references to other repos)
3. What depends on it? (check for package names, API endpoints others might call)
4. How is it deployed? (check CI/CD)

Then draw me a dependency diagram of the whole ecosystem.
```

### Challenge: Proprietary or Internal Frameworks

```
This project uses an internal framework/library called [name] that you won't know about.
Help me understand it:
1. Find where it's imported and how it's used in this codebase
2. Read its source code (if available as a dependency or in a sibling repo)
3. Look for any internal documentation, README, or comments
4. Show me the most common usage patterns in this codebase
5. Summarize what it does and how to use it based on the evidence
```

---

## 6.10 The Gradual AI Readiness Scorecard

Rate your project on each dimension. Improve one level at a time.

### Build & Run (Foundation)
```
Level 0: Can't build locally
Level 1: Builds but requires tribal knowledge
Level 2: Documented setup, builds reliably
Level 3: One-command setup (docker-compose up or equivalent)
```

### Testing (Safety Net)
```
Level 0: No tests
Level 1: Some tests, many failing or skipped
Level 2: Tests pass, cover happy paths
Level 3: Comprehensive tests, including edge cases
Level 4: Tests serve as documentation — you can understand behavior by reading them
```

### Documentation (Context)
```
Level 0: No docs
Level 1: Outdated README
Level 2: Accurate README + architecture overview
Level 3: CLAUDE.md + up-to-date docs + documented conventions
Level 4: Self-documenting — clear naming, patterns, and CLAUDE.md that enables autonomous AI work
```

### Coding Patterns (Consistency)
```
Level 0: Every file does things differently
Level 1: Some patterns, inconsistently applied
Level 2: Clear patterns in most of the codebase
Level 3: Enforced patterns (linters, templates, code review norms)
```

### AI Readiness (Integration)
```
Level 0: No AI tooling
Level 1: CLAUDE.md exists, basic AI-assisted coding
Level 2: CLAUDE.md + memory + consistent patterns → reliable AI output
Level 3: Multi-agent workflows, autonomous tasks, CI-integrated AI
Level 4: AI is a core part of the development workflow for the whole team
```

**Target for your first week: Get every dimension to at least Level 2.**

---

## 6.11 The First-Week Checklist

### Day 1: Reconnaissance
- [ ] Run the 5-minute scan
- [ ] Map dependencies
- [ ] Understand architecture (even roughly)
- [ ] Identify where everything lives
- [ ] Write personal notes (what you understand, what you don't)

### Day 2: Foundation
- [ ] Get the project building
- [ ] Get tests running (note which ones fail)
- [ ] Get the app running locally
- [ ] Document what you learned (exact commands, pitfalls)

### Day 3: AI Setup
- [ ] Create CLAUDE.md
- [ ] Set up Claude Code memory with project context
- [ ] Verify the AI can: read code, run tests, make simple edits

### Day 4: First Wins
- [ ] Fix a trivial issue with AI assistance
- [ ] Add a test with AI assistance
- [ ] Do a small, well-defined task end-to-end with AI

### Day 5: Integration
- [ ] Refine CLAUDE.md based on what you've learned
- [ ] Set up one autonomous/scheduled task
- [ ] Create a topology map (if multi-repo)
- [ ] Start your trust log

---

## 6.12 Template: Project Onboarding Brief

Use this template to document each project you onboard. Save it in the repo or in your personal notes:

```markdown
# Project Onboarding Brief: [Project Name]
**Date:** YYYY-MM-DD
**Onboarded by:** [you]

## Overview
[2-3 sentences: what is this, who uses it, why does it exist]

## Tech Stack
- Language:
- Framework:
- Database:
- Package Manager:
- Test Framework:
- CI/CD:

## How to Build / Test / Run
```[commands]```

## Architecture (ASCII diagram)
```
[diagram]
```

## Key Directories
| Path | Purpose |
|------|---------|

## Conventions
- [observed pattern 1]
- [observed pattern 2]

## Gotchas & Tribal Knowledge
- [thing 1]
- [thing 2]

## Red Flags / Tech Debt
- [issue 1]
- [issue 2]

## Related Repos / Services
| Repo/Service | Relationship |
|--------------|-------------|

## AI Readiness Score
- Build & Run: Level _/3
- Testing: Level _/4
- Documentation: Level _/4
- Coding Patterns: Level _/3
- AI Readiness: Level _/4

## What I Still Don't Understand
- [question 1]
- [question 2]
```

---

## Self-Check Questions

- [ ] What are the 5 phases of onboarding a project for AI-native development?
- [ ] What's the first thing you should ask Claude about an unknown codebase?
- [ ] Why are tests the most critical prerequisite for AI-assisted coding?
- [ ] How would you handle a project with zero documentation?
- [ ] What's your AI Readiness Score target for week 1?

→ **Back to [README](../README.md) | Previous: [Module 05](05-cross-repo-development.md)**
