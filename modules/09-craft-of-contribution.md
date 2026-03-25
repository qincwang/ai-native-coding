# Module 09: The Craft of Good Contributions in the AI Era

> When anyone can generate code, what makes a contribution valuable? When AI writes the implementation, what's left for you? More than you think — but it's different than before.

---

## 9.1 How AI Changed Open Source

### What Already Happened

**The contribution bar dropped massively.** Someone who couldn't write Go can now contribute to a Go project. AI translates intent across languages. This means more contributors, but also more low-quality PRs. Maintainers are drowning in AI-generated PRs that "look right" but miss the project's design philosophy.

**Issue-to-PR speed collapsed.** What used to take days — read the codebase, understand the patterns, write a fix — now takes hours. This sounds good until you realize that speed without understanding creates churn. PRs get opened faster, but review burden increases because the submitter often doesn't deeply understand what they changed.

**Documentation got better and worse simultaneously.** AI generates docs easily, so more projects have them. But AI-generated docs can be subtly wrong, and nobody fact-checks them as carefully as hand-written docs. The signal-to-noise ratio shifted.

**New categories of projects exploded:**
- MCP servers (hundreds of community-built integrations)
- AI coding tool plugins and extensions
- Wrapper projects (thin layers over AI APIs)
- "Vibe coded" projects — shipped fast, abandoned fast

### What's Changing Right Now

**Maintainer burnout is accelerating.** The volume of AI-generated issues, PRs, and questions overwhelmed many maintainers. Some projects now explicitly ban AI-generated PRs without human understanding. Others require contributors to explain their changes in their own words.

**Code review became the bottleneck.** When anyone can generate code, the scarce resource shifts to people who can review it well. The best open-source contributors in 2026 aren't the fastest coders — they're the most thoughtful reviewers.

**Fork-and-customize replaced contribute-upstream.** It's now trivial to fork a project and have AI customize it for your needs. This reduces the pressure to contribute features back upstream. Some ecosystems are fragmenting because of this.

### The Deeper Shifts Most People Miss

**Open source is becoming a design competition, not a coding competition.** The projects that win aren't the ones with the most code — they're the ones with the clearest architecture, the best CLAUDE.md, the most reviewable patterns. Because if AI can implement anything, the differentiator is *what* to implement and *how* it's organized.

**Licensing conversations got more complicated.** AI trained on open-source code generates code that resembles its training data. This raised legal questions that still aren't fully resolved. Some projects added explicit AI-related clauses to their licenses.

**The "bus factor" improved and degraded.** AI makes it easier for new contributors to understand a codebase (ask Claude to explain it). But it also means fewer people develop deep understanding, because they can get surface-level answers without doing the work. The knowledge gets wider but shallower.

**Small maintainers got superpowers.** A single developer maintaining a library can now respond to issues, triage bugs, and even generate fix PRs at a pace that used to require a team. The solo maintainer with good AI workflows is more productive than a small team without them.

### How Standing Out Shifted

| Old way to stand out | New way to stand out |
|---|---|
| Submit the most PRs | Submit the most thoughtful PRs with clear explanations |
| Write features fast | Review others' PRs carefully (especially AI-generated ones) |
| Know the codebase by heart | Write great CLAUDE.md and contributing guides |
| Fix obscure bugs | Design clean APIs and architecture that others can extend |
| Respond to issues quickly | Triage and clarify vague issues into actionable specs |

---

## 9.2 What Makes a Good PR in 2026

It's not about the code anymore. AI can produce clean, passing, well-tested code. A good PR is now defined by what's AROUND the code.

### 1. The "Why" Is Clear and Honest

The PR description doesn't say "add user endpoint." It says:

> "Users currently can't update their email without contacting support. This adds a self-service endpoint. I chose PATCH over PUT because we only allow partial updates, consistent with how /profile works."

That reasoning is the value. AI can write the endpoint. It can't decide whether the endpoint should exist, or whether PATCH is the right verb for this project's philosophy.

**Bad PR description:**
```
Added PATCH /users/:id/email endpoint with validation and tests.
```

**Good PR description:**
```
## Why
Users must currently contact support to change their email address.
This creates ~40 support tickets/week (see issue #234).

## What
Self-service email update via PATCH /users/:id/email.

## Design Decisions
- PATCH not PUT: consistent with existing partial-update pattern (/profile, /settings)
- Requires re-authentication: email is a sensitive field, matching how /password works
- Sends confirmation to both old and new email: prevents account takeover

## What I Considered but Didn't Do
- Email change queue with admin approval: overkill for this use case
- Allowing batch email changes via admin API: separate concern, filed as #267

## Test Plan
- Happy path: change email with valid input
- Auth: rejected without re-authentication
- Validation: rejected with invalid email format
- Conflict: rejected if email already in use
- Notification: both old and new addresses receive emails
```

The second version captures decisions, alternatives considered, and the relationship to the rest of the system. This is what a maintainer needs to evaluate the PR — not just "does the code work?" but "is this the right change?"

### 2. The Change Respects the Repo's Trajectory

This is where most AI-generated PRs fail. They solve the immediate problem in a way that's technically correct but architecturally wrong.

```
The repo is migrating from callbacks to async/await.
70% of the codebase is already converted.

Your PR adds a new feature using callbacks because
"that's what the adjacent file uses."

Technically correct. Philosophically wrong.
You just moved the project backward.
```

A good PR reads the direction the codebase is going, not just where it is today.

**How to detect trajectory:**
- Read the last 20 commit messages — what patterns are emerging?
- Read open PRs — what direction is active work going?
- Read closed/rejected PRs — what direction was explicitly turned down?
- Check for migration guides, RFCs, or ADRs in the docs
- Look for `TODO: migrate`, `@deprecated`, `legacy` markers in the code

### 3. The Scope Is Intentional

A good PR changes exactly what it should and nothing else. Not "I was in the area so I also refactored X and updated Y and fixed Z." Each of those should be separate.

AI loves to over-deliver. It "helpfully" cleans up adjacent code, adds error handling to functions you didn't ask about, refactors imports. Good contributors constrain scope deliberately.

**Rule of thumb:** If you can't describe the PR in one sentence without using "and," it should probably be multiple PRs.

### 4. Edge Cases Are Addressed, Not Just the Happy Path

AI produces code that handles the described scenario. A good PR also handles:
- What if this input is null/undefined/empty?
- What if the database is temporarily unavailable?
- What if two users hit this endpoint simultaneously?
- What if this data already exists?
- What if the user doesn't have permission?
- What happens at scale — 1 request vs. 10,000?

The mark of a good contributor isn't that the code works — it's that the contributor *thought about what happens when it doesn't work.*

### 5. The Tests Prove Behavior, Not Implementation

**Bad test (implementation-coupled):**
```javascript
test('creates user', () => {
  const spy = jest.spyOn(database, 'save');
  createUser({ name: 'Alice' });
  expect(spy).toHaveBeenCalledOnce();
  expect(spy).toHaveBeenCalledWith({ name: 'Alice', role: 'user' });
});
```

**Good test (behavior-proving):**
```javascript
test('created user can be retrieved with correct data', async () => {
  await createUser({ name: 'Alice', email: 'alice@example.com' });
  const user = await getUser('alice@example.com');
  expect(user.name).toBe('Alice');
  expect(user.role).toBe('user'); // default role
});
```

AI tends to write implementation-coupled tests. Good PRs test outcomes — what the user/caller observes, not how the internals work.

---

## 9.3 What's "Human Effort" vs. "Throw to AI"

### Throw to AI Without Guilt

These are tasks where AI's speed matters and human judgment adds little:

- Writing the boilerplate (route handler, model definition, test scaffolding)
- Generating repetitive variations (5 similar endpoints, CRUD operations)
- Writing the initial test suite for existing code
- Formatting, renaming, mechanical refactors
- Documentation for code that already exists
- Converting between formats (SQL to ORM, REST to GraphQL schema)
- Fixing lint errors, type errors, import ordering
- Generating migration files from schema changes
- Writing commit messages and PR descriptions (from diffs)
- Boilerplate config files (Dockerfile, CI YAML, tsconfig)

### Human Effort — This Is Where You Earn Your Place

These require judgment, context, and accountability that AI can't provide:

- **Deciding whether the feature should exist at all.** "Should we build this?" is a human question.
- **Choosing between architectural approaches and explaining why.** AI can present options. You choose and own the consequences.
- **Identifying what the ticket DOESN'T say but should.** The gaps in requirements are where bugs hide.
- **Noticing that the proposed solution conflicts with another team's work.** AI doesn't know what Team B is building in their repo.
- **Recognizing that "the simple fix" creates a maintenance trap.** "This works now but creates a coupling that will hurt us in 3 months."
- **Writing the PR description that captures intent and trade-offs.** The WHY, not the WHAT.
- **Reviewing someone else's AI-generated code with genuine critical thinking.** This is the most valuable contribution in 2026.
- **Saying "no" to a feature or approach.** AI is agreeable. Humans must be willing to push back.

### The Gray Zone (AI Drafts, You Refine)

- Error handling strategy — AI writes it, you verify it matches the project's philosophy
- Test cases — AI generates them, you ask "what's missing? what edge case did it overlook?"
- API design — AI proposes, you evaluate against existing patterns and user needs
- Code review comments — AI flags issues, you decide which matter and how to communicate them
- Documentation — AI drafts, you verify accuracy and add the "why" behind design decisions

### The Decision Framework

```
Can AI do this correctly without understanding     → Throw to AI
the broader context?                               (boilerplate, formatting,
                                                    mechanical tasks)

Does this require understanding the codebase's     → Gray zone
patterns but not its philosophy?                   (AI drafts, you refine)

Does this require judgment about what SHOULD       → Human effort
be built, not just HOW to build it?               (design, decisions, review)
```

---

## 9.4 Detecting Philosophical Misalignment with a Repo

This is the hardest skill and the most important one. Your code can pass all tests, follow all lint rules, and still be fundamentally wrong for the project.

### Signal 1: You're Fighting the Existing Patterns

If you find yourself writing "workarounds" or "special cases" to make your approach work with the existing code, your approach probably doesn't fit. Good design in the context of an existing project should feel like it slots in naturally.

```
Red flag: "I need to add a special adapter because the existing
          service layer doesn't support my approach"

Question: Should you change your approach to fit the service layer?
          Or does the service layer need to evolve?
          If the latter, that's a SEPARATE discussion/PR,
          not a silent refactor buried in your feature.
```

**How to check:**
```
Ask Claude: "I'm about to add [your approach]. Read 3-4 files that
do something similar in this project. Does my approach match how
they do it? If not, what's different and why might that matter?"
```

### Signal 2: You're Introducing a New Pattern When an Existing One Exists

Every repo has implicit conventions. If there are 15 files that do error handling with Result types and you add one that uses try-catch, you're misaligned — even if try-catch is "better" in isolation.

**The principle:** In a collaborative codebase, consistency is almost always more valuable than local optimization. One "better" pattern mixed with 15 "existing" patterns creates 2 patterns to maintain instead of 1.

**Exception:** When the project is deliberately migrating to the new pattern and your PR is part of that migration (see Signal about trajectory above).

**How to check:**
```
Before writing any code:
1. Grep for how the thing you're about to do is done elsewhere
2. Count the instances — if there's a dominant pattern, follow it
3. If there are multiple patterns, check which one appears in
   more recent files (that's likely the direction)
```

### Signal 3: Your Abstraction Level Doesn't Match

Some repos are deliberately low-level — explicit, verbose, no magic. Others are heavily abstracted — DSLs, metaprogramming, convention-over-configuration. Adding a clever abstraction to a deliberately explicit codebase is a philosophical mismatch even if it "works."

```
The repo uses raw SQL queries everywhere.
You add an ORM model.

Maybe the raw SQL is there for a reason — performance control,
query visibility, avoiding ORM edge cases. Or maybe it's just
legacy. You need to find out BEFORE the PR.
```

**Examples of abstraction-level mismatches:**
- Adding a factory pattern to a codebase that uses plain constructors
- Adding dependency injection to a codebase that uses direct imports
- Adding a custom DSL to a codebase that values explicit code
- Adding metaprogramming (decorators, macros) to a codebase that avoids magic
- Using inheritance in a codebase that prefers composition

**The test:** Would a maintainer look at your code and think "that's clever" or "that's clear"? In most open-source projects, "clear" wins.

### Signal 4: Your Change Assumes a Different Future Than the Maintainers Intend

This is the subtlest one. You add a plugin system because "obviously this should be extensible." But the maintainers deliberately kept it simple because extensibility adds maintenance burden they can't afford. Your "improvement" is philosophically opposed to the project's values.

**Common examples:**
- Adding configurability to something deliberately hardcoded
- Adding extensibility to something deliberately simple
- Adding abstraction to something deliberately concrete
- Adding features to something deliberately minimal
- Adding backward compatibility layers when the project wants to move forward

**How to check:**
```
Read the project's:
1. CONTRIBUTING.md — explicit philosophy statements
2. Closed PRs that were REJECTED — WHY were they rejected?
   This is gold. The rejections tell you the project's boundaries.
3. Issue discussions — how do maintainers talk about the future?
4. Architecture Decision Records (ADRs) — if they exist
5. The README's "Non-goals" section — if it exists
6. The maintainer's comments on similar PRs — tone, priorities, concerns
```

### Signal 5: You're Optimizing for a Different Audience

A library designed for beginners values clarity over cleverness. A library designed for performance values speed over readability. A library designed for correctness values type safety over ergonomics.

If your PR optimizes for the wrong value, it's misaligned.

| Project Values | Your PR Should Optimize For |
|---|---|
| Simplicity (e.g., Express, Hono) | Minimal API surface, obvious behavior |
| Performance (e.g., Fastify, Drizzle) | Speed, zero unnecessary allocation |
| Correctness (e.g., Zod, TypeScript) | Type safety, exhaustive handling |
| Beginner-friendliness (e.g., Create React App) | Clear naming, helpful errors, guided experience |
| Flexibility (e.g., Fastify plugins) | Composable, non-opinionated |

---

## 9.5 The Pre-PR Alignment Test

Run this checklist before submitting any PR to a project you don't maintain:

### 1. Pattern Check
```
Read 5 files that do something similar to what you're adding.
Does your code look like it belongs in the same project?
□ Yes → proceed
□ No → adjust to match, or discuss the deviation in the PR description
```

### 2. Trajectory Check
```
Read the last 20 commits and any open PRs.
Does your change move in the same direction as recent work?
□ Yes → proceed
□ No → ask yourself: am I intentionally proposing a new direction?
       If so, open an issue to discuss BEFORE writing the PR.
```

### 3. Rejection History Check
```
Search closed/rejected PRs for similar proposals.
Has this been tried and rejected before?
□ No similar PRs → proceed
□ Similar PR was rejected → read WHY. Address those concerns or don't submit.
```

### 4. Scope Check
```
Can you describe this PR in one sentence without "and"?
□ Yes → proceed
□ No → split into multiple PRs
```

### 5. The Maintainer Thought Experiment
```
If you deleted your code and gave the same task to the maintainer,
would they write something structurally similar?
□ Yes → you're aligned
□ "Similar but they'd do X differently" → adjust X
□ "They'd approach it completely differently" → stop and reconsider
□ "They wouldn't build this at all" → open an issue first, not a PR
```

### 6. The Philosophy Statement
```
Can you explain your design choice in one sentence that a maintainer
would nod at?
□ Yes → proceed
□ The explanation requires "I know the project does Y, but I think Z
  is better because..." → you're proposing a philosophical change.
  That needs a discussion BEFORE a PR.
```

---

## 9.6 Good Contributions Beyond Code

In the AI era, some of the highest-value contributions aren't code at all:

### Triage Issues
Turn vague bug reports into actionable specs. Reproduce the bug, narrow down the cause, label it correctly. This saves maintainer hours.

### Review PRs
Thoughtful code review — especially of AI-generated PRs — is the scarcest resource. A review that catches a philosophical mismatch or a missed edge case is worth more than 10 AI-generated features.

### Write CLAUDE.md / Contributing Guides
Make the project AI-native-friendly. Write clear build instructions, document conventions, explain the "why" behind patterns. This multiplies the effectiveness of every future contributor (human and AI).

### Close Stale Issues
Many projects have hundreds of open issues, many of which are duplicates, already fixed, or no longer relevant. Systematically triaging and closing these is unglamorous but valuable.

### Answer Questions
In discussions, Stack Overflow, Discord — helping others understand the project builds community and reduces maintainer burden.

---

## 9.7 The Contribution Quality Ladder

```
Level 1: "It works"
  AI can do this. Code compiles, tests pass.
  Value: minimal. Thousands of people can produce this now.

Level 2: "It fits"
  Code follows existing patterns and conventions.
  Value: moderate. Shows you read the codebase, not just the ticket.

Level 3: "It's thoughtful"
  Edge cases handled. Clear PR description with reasoning.
  Non-obvious decisions explained. Tests prove behavior.
  Value: high. This is what maintainers want to review.

Level 4: "It moves the project forward"
  Change aligns with the project's trajectory. Design is consistent
  with the project's philosophy. Trade-offs are acknowledged.
  Value: very high. This is what earns you trust and commit access.

Level 5: "It makes the project better beyond the code"
  Improves docs, testing infrastructure, developer experience.
  Makes future contributions easier. Mentors other contributors.
  Value: exceptional. This is how you become a core maintainer.
```

Most AI-generated PRs cap at Level 1. Most human PRs without thought cap at Level 2. Your goal is Level 3 minimum, Level 4 when you know the project well.

---

## Self-Check Questions

- [ ] What's the difference between a Level 1 and Level 3 PR?
- [ ] Name 3 signals of philosophical misalignment with a repo
- [ ] What would you check before submitting a PR to a project you've never contributed to?
- [ ] Which parts of your last PR could AI have written? Which parts required your judgment?
- [ ] Find a rejected PR in an open-source project — why was it rejected? Would you have caught the issue?

→ **Back to [README](../README.md) | Previous: [Module 08](08-systems-components-deep-dive.md)**
