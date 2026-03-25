# Lab 05: Onboarding an Unknown Project

> Estimated time: 2-3 hours

## Objective

Practice the full onboarding workflow on a real open-source project you've never seen before. By the end, you'll have a CLAUDE.md, a mental model, and AI-assisted dev capability.

---

## Setup: Pick Your Project

Choose ONE open-source project you've never worked with. Recommendations by difficulty:

**Easy (small, well-structured):**
- `express` (Node.js web framework — you know what it does, but have you read its source?)
- `fastapi` (Python web framework)
- `chi` (Go HTTP router)

**Medium (larger, real-world patterns):**
- `cal.com` (scheduling app — Next.js + tRPC + Prisma)
- `maybe` (personal finance app — Rails)
- `hono` (edge web framework — TypeScript)

**Hard (complex, multi-component):**
- `sentry` (error tracking — Python/Django + React)
- `supabase` (open source Firebase — multiple services)

Clone it:
```bash
git clone [repo-url] ~/lab-05-project
cd ~/lab-05-project
```

---

## Exercise 1: The 5-Minute Briefing (Reconnaissance)

Open Claude Code and run the reconnaissance sequence from Module 06:

```
I just cloned this project and I've never seen it before.
Give me a 5-minute briefing:
1. Language, framework, tech stack
2. Directory structure (top 2 levels, explain each)
3. Entry point — how does this app start?
4. Build/test/run commands
5. How many tests? What framework?
6. Recent git activity (last 10 commits, active contributors)
7. The 5 most-changed files (core of the project)
```

**Record:**
- How long did this take?
- What did you learn that surprised you?
- What's still unclear?

---

## Exercise 2: Trace a Request

```
Pick the simplest API endpoint or user-facing feature in this project.
Trace the full lifecycle of a request/interaction:
1. Where does it enter? (route handler, controller, etc.)
2. What middleware/hooks does it pass through?
3. Where is the business logic?
4. How does it touch the database (if applicable)?
5. How does the response get formed and sent?

Show me the actual file paths and line numbers for each step.
```

**Draw your own diagram** (pen and paper or text). Then compare:
```
Here's my understanding of the request flow: [your diagram]
Am I right? What did I miss?
```

---

## Exercise 3: Get It Running

```
Help me get this project running on my machine.
1. Read ALL setup docs (README, CONTRIBUTING, docs/)
2. Read the Dockerfile and CI config for the "real" setup steps
3. Give me exact commands in order
4. Warn me about likely gotchas on Windows
```

**Track every issue you hit.** These become your "Gotchas" section later.

If you get stuck:
```
I'm getting this error: [paste error]
I've already tried: [what you tried]
Diagnose this — is it environment, config, or a real bug?
```

---

## Exercise 4: Run the Tests

```
Run the test suite. Give me a report:
1. Total tests, passing, failing, skipped
2. Categorize failures: environment issue vs. real bug vs. flaky
3. How long do tests take?
4. Are there different test suites (unit, integration, e2e)?
```

---

## Exercise 5: Create the CLAUDE.md

Now the core deliverable — create a CLAUDE.md from what you've learned:

```
Based on everything we've explored, create a CLAUDE.md for this project.
Include every section from the template in Module 06.
Be honest about what's uncertain — mark those as "TODO: verify".
```

**Review the CLAUDE.md yourself.** Does it capture what you now know? Add anything Claude missed from your personal experience getting it running.

---

## Exercise 6: The Trust Test

Now test your setup with 3 escalating tasks:

**Task 1 — Trivial:**
```
Find a typo, outdated comment, or minor code style issue. Fix it. Run tests.
```

**Task 2 — Small:**
```
Find the simplest test file. Add one more test case that tests an untested edge case. Run tests.
```

**Task 3 — Real:**
```
Pick a small, self-contained feature or improvement:
- Add input validation to an endpoint that lacks it
- Add error handling to a function that doesn't handle failures
- Add a missing test for a critical code path

Implement it. Run tests. Review the diff carefully.
```

**For each task, record:**
- Did the AI follow the project's conventions?
- Did it read files before editing?
- Did you catch anything in review?
- Did tests pass?

---

## Exercise 7: The Onboarding Brief

Fill out the complete Project Onboarding Brief template from Module 06, Section 6.12.

This is your deliverable — a document that would let anyone (human or AI) understand this project in 10 minutes.

---

## Bonus: Speed Run

Time yourself doing Exercises 1-5 on a SECOND project. Compare:
- How much faster was the second time?
- What parts of the process have you internalized?
- What would you change about your approach?

---

## Reflection

Journal entry:
1. What was the hardest part of onboarding an unknown project with AI?
2. At what point did you feel "I understand this project enough to contribute"?
3. How does AI-assisted onboarding compare to your usual approach?
4. What would you add to the Module 06 onboarding process?
