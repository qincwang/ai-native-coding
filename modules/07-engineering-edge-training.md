# Module 07: Engineering Edge Training

> The skills AI can't replace, trained deliberately — even when your day job is "regular" work.

This is a **living module**. It grows as you complete exercises, discover new questions, and push into harder territory. Each section has starter exercises, space for your own additions, and a progression system.

---

## How This Module Works

Each skill area follows the same structure:
1. **Starter Questions** — curated exercises to begin with
2. **Your Questions** — problems you discover and add yourself
3. **Practice Log** — track what you did and what you learned
4. **Level Progression** — how you know you're getting better

**Weekly rhythm:** Pick ONE exercise from ONE area. Spend 30-60 minutes. Write what you learned in the practice log. That's it. Consistency beats intensity.

---

## Area 1: System Design Under Real Constraints

> The skill: given a vague need and real-world constraints (budget, team size, timeline, traffic), design a system that works. Not the "perfect" system — the right system for the situation.

### Why It Matters

AI can generate any architecture you describe. It cannot decide WHICH architecture fits your constraints. That judgment — "we should use Postgres, not DynamoDB, because our team knows SQL and we don't need the scale" — is irreplaceable.

### Starter Questions

Work through these in order. For each one, spend 30-60 minutes. Write your design BEFORE looking anything up. Then research how it's actually done. The gap is your learning.

---

#### SQ-1.1: Redesign Your Own Project

```
Take the project you work on daily. Answer these:

1. Draw the current architecture (boxes and arrows, keep it simple).
   - What are the components? How do they communicate?
   - Where does data live? How does it flow?

2. Now stress-test your drawing:
   - What's the single point of failure? If ONE thing dies, what breaks?
   - Where's the bottleneck if traffic 10x's?
   - What happens if the database is unreachable for 60 seconds?

3. If you were starting this project from scratch today, knowing what you know:
   - What would you keep the same?
   - What would you change?
   - What's one thing that's over-engineered? Under-engineered?
```

**With AI:**
```
Here's my current architecture: [describe it]
Ask me 10 hard questions about this design — things that would expose
weaknesses, hidden assumptions, or scaling limits.
Don't answer them — just ask. I want to think through them myself.
```

---

#### SQ-1.2: The "10x Traffic" Exercise

```
Take any web application you use (not your own — pick something you're
a user of). Start simple:

Scenario: This app currently serves 10,000 users. Design it to serve 100,000.

1. What are the likely components? (Web server, database, cache, CDN, etc.)
2. What breaks first at 10x? (Database connections? API rate? Memory?)
3. What's the cheapest fix for each bottleneck?
4. What changes if you go to 1,000,000 users?

Constraint: Your team is 3 engineers and you have 6 weeks.
```

**The constraint is the point.** The "perfect" design with unlimited resources is boring. The interesting question is: what do you cut? What do you defer? What's the minimum viable architecture?

---

#### SQ-1.3: The Trade-off Matrix

```
You're building a notification system (email + push + in-app).
Design it THREE different ways:

Option A: Simplest possible (get it working in a week)
Option B: Scalable (handle 1M notifications/day)
Option C: Resilient (zero message loss, even during outages)

For each, answer:
- What components?
- What are you explicitly sacrificing?
- When would you choose this option?
- What's the migration path from A → B → C?
```

**This trains the most important design skill: knowing what to sacrifice.** Every real design is a set of trade-offs. "We can have fast OR consistent, not both." "We can have simple OR scalable, not both." Practicing this explicitly builds the judgment muscle.

---

#### SQ-1.4: The Postmortem Redesign

```
Find a public postmortem (suggestions below). Read it. Then answer:

1. Before reading the root cause: given the symptoms, where would YOU look first?
2. After reading it: could this architecture have prevented the issue?
3. Design an alternative architecture that makes this class of failure impossible.
4. What does your alternative sacrifice? (cost, complexity, speed, etc.)

Postmortem sources:
- https://github.com/danluu/post-mortems (curated list)
- Search "[company name] engineering blog postmortem"
- Cloudflare, GitHub, Google, AWS all publish detailed ones
```

---

#### SQ-1.5: Design for Your Non-Technical Stakeholder

```
Pick a system you understand. Now explain the architecture to:
1. A product manager who asks "why can't we just add real-time updates?"
2. A CEO who asks "why does infrastructure cost $X/month?"
3. A new engineer who asks "why didn't we just use [trendy technology]?"

For each, your explanation should:
- Use no jargon (or define it immediately)
- Focus on trade-offs, not implementation details
- End with "here's what we'd need to change and what it would cost"
```

**This sounds soft but it's a career accelerator.** Engineers who can explain technical trade-offs to non-technical people get invited into strategic decisions. Those who can't stay in the implementation trench.

---

### Your Questions (Add Your Own)

As you encounter design challenges — at work, in side projects, in things you read — add them here:

```
#### SQ-1.X: [Title]
[Date added: YYYY-MM-DD]
[Context: why this question interests you]
[The question / exercise]
[What you learned: fill in after completing]
```

### Practice Log

| Date | Exercise | Time Spent | Key Insight | Confidence (1-5) |
|------|----------|-----------|-------------|-------------------|
| | | | | |

### Level Progression

```
Level 1: Can draw the architecture of a system you use daily
Level 2: Can identify single points of failure and bottlenecks
Level 3: Can design 2-3 alternatives with clear trade-off reasoning
Level 4: Can design for specific constraints (team size, budget, timeline)
Level 5: Can review someone else's design and find the weaknesses they missed
```

---

## Area 2: Navigating Ambiguity

> The skill: turning vague, contradictory, or incomplete requirements into a clear technical plan — without waiting for someone to hand you a spec.

### Why It Matters

AI needs clear instructions. Customers, product managers, and business stakeholders give you unclear instructions. The person who bridges that gap — who turns "users are churning" into "we need an onboarding flow with these 5 steps" — is the most valuable person on the team.

### Starter Questions

---

#### AQ-2.1: The Reverse Spec

```
Pick a feature in a product you use daily (e.g., Spotify's "Discover Weekly",
GitHub's pull request review, Uber's surge pricing).

Write the ORIGINAL product spec that you think led to this feature being built.
Include:
1. The problem statement (1-2 sentences — what user pain does this solve?)
2. The requirements (what must be true for this to be "done"?)
3. The edge cases (what weird situations does this handle?)
4. The non-goals (what does this deliberately NOT do?)
5. The technical constraints you'd guess (latency, data freshness, etc.)

Then use the actual feature and test your spec against reality:
- What did you miss?
- What did the builders include that you wouldn't have thought of?
- What would you change about their implementation?
```

**Why this works:** It reverses the usual process. Instead of "spec → build," you're doing "product → spec." This trains the translation muscle in both directions.

---

#### AQ-2.2: The Vague Request Drill

```
A stakeholder sends you these actual messages. For each one, write:
(a) 3 clarifying questions you'd ask BEFORE doing anything
(b) Your best-guess spec if you couldn't ask anyone (you're on your own)
(c) The riskiest assumption in your spec

Message 1: "Can we add analytics to the dashboard?"

Message 2: "Users are complaining the app is slow."

Message 3: "We need to support multiple languages by Q3."

Message 4: "Can you make the search better?"

Message 5: "Legal says we need to be GDPR compliant."
```

**The pattern to internalize:**
- "Analytics" means what, specifically? Page views? User behavior funnels? Revenue metrics?
- "Slow" where? For whom? Under what conditions? Compared to what?
- "Multiple languages" — UI translations? Content? User-generated content? Right-to-left support?

Every vague word hides 5 decisions. Your job is to find them.

---

#### AQ-2.3: The Spec That Kills Scope Creep

```
You're building a "user settings page." Simple, right?

Write the COMPLETE spec. Include every decision:
1. What settings can users change? (list exhaustively)
2. How are settings saved? (auto-save? save button? per-field?)
3. What validation exists? (email format? password strength?)
4. What happens on error? (inline? toast? modal?)
5. What's the mobile experience?
6. Do settings sync across devices?
7. Can admins override user settings?
8. What's the default for each setting?
9. What happens to existing users when we add a new setting?
10. What settings should NOT be user-configurable? Why?

Now circle the 3 decisions that would cause the most debate on your team.
Those are the high-ambiguity decisions. Practice writing a one-paragraph
argument for each side.
```

---

#### AQ-2.4: Translate Business to Technical

```
Practice translations. For each business statement, write the technical
implications:

"We're launching in Europe next quarter."
→ What does this mean technically? (data residency, GDPR, i18n, latency,
  payment providers, timezone handling, etc.)

"We want to let users invite their team."
→ What does this mean technically? (multi-tenancy, permissions model,
  invitation flow, email service, edge cases around existing accounts, etc.)

"We need to cut infrastructure costs by 30%."
→ What does this mean technically? (profiling, right-sizing, reserved instances,
  caching, eliminating waste, feature flags to sunset unused features, etc.)

Pick one and write a full technical plan. Include effort estimate, risks,
and what you'd cut if the timeline was halved.
```

---

#### AQ-2.5: The Stakeholder Simulation

```
Use Claude as a deliberately vague stakeholder:

Prompt to Claude:
"You are a product manager. You want a 'reporting feature' for our SaaS
application. You have a general idea but haven't thought through the details.
When I ask you questions, answer like a real PM — sometimes clearly,
sometimes vaguely, sometimes changing your mind. Don't volunteer information
I don't ask about. Occasionally introduce a new requirement mid-conversation."

Then interview Claude. Try to extract a complete, buildable spec through
questions alone. Track:
- How many questions did it take?
- What requirements did you almost miss?
- When did the PM "change their mind" and how did you handle it?
```

**This is the exercise that most closely simulates real work.** The skill isn't getting the right answer — it's asking the right questions.

---

### Your Questions (Add Your Own)

```
#### AQ-2.X: [Title]
[Date added: YYYY-MM-DD]
[Context: why this question interests you]
[The question / exercise]
[What you learned: fill in after completing]
```

### Practice Log

| Date | Exercise | Time Spent | Key Insight | Confidence (1-5) |
|------|----------|-----------|-------------|-------------------|
| | | | | |

### Level Progression

```
Level 1: Can identify that a requirement is vague (know what you don't know)
Level 2: Can generate clarifying questions that uncover hidden decisions
Level 3: Can write a complete spec from a vague request with reasonable assumptions
Level 4: Can identify the highest-risk assumptions and de-risk them first
Level 5: Can lead a spec process — drive from "vague idea" to "team-aligned plan"
```

---

## Area 3: Debugging Complex Systems

> The skill: given symptoms (errors, slowness, wrong data), systematically find the root cause — even across multiple components, services, and layers.

### Why It Matters

AI can grep for errors and suggest fixes for known patterns. But real debugging requires forming hypotheses, ruling out causes, and following evidence through layers of indirection. The person who can say "I think this is a connection pool exhaustion issue, let me check the metrics" saves hours compared to the person who stares at the error message.

### Starter Questions

---

#### DQ-3.1: Build a System, Then Break It

```
Build a minimal multi-component system with Docker Compose:

docker-compose.yml:
- A web server (Node/Python/Go — your choice) on port 3000
- A PostgreSQL database
- A Redis cache

The app should:
- GET /users → read from database (cache in Redis for 60 seconds)
- POST /users → write to database (invalidate Redis cache)

This is intentionally simple. The value is in what you do AFTER building it.

Now introduce these failures ONE AT A TIME. For each:
- Observe the symptoms (what error? what behavior?)
- Diagnose WITHOUT looking at what you changed
  (pretend a teammate broke it and you're investigating)
- Document: symptoms → hypothesis → investigation steps → root cause

Failure 1: Stop the Redis container. What happens to GET /users?
Failure 2: Set the database password wrong. What error do users see?
Failure 3: Add a 5-second sleep to the database query. What happens under load?
Failure 4: Fill up disk space in the Postgres container. What breaks?
Failure 5: Introduce a connection pool limit of 2. Hit the API with 10 concurrent requests.
```

**The training is in the diagnosis, not the fix.** Force yourself to form a hypothesis BEFORE looking at logs. "I think this is a database connection issue because..." Then check if you're right.

---

#### DQ-3.2: The Mystery Bug Game

```
Use Claude as a debugging partner. Give it this prompt:

"You are simulating a production system. A bug exists, but you won't tell
me what it is directly. Here's the system:
- Node.js API server with Express
- PostgreSQL database
- Redis cache
- The system processes e-commerce orders

The bug: some orders are being processed twice, but only during peak hours
(not during normal traffic). Users are being charged twice occasionally.

When I ask to check something (logs, metrics, code, database), give me
realistic output that would help me diagnose. Include some red herrings.
Don't tell me the answer until I correctly identify the root cause."

Then debug it:
1. What's your first hypothesis?
2. What would you check first? (logs? metrics? code? database?)
3. What do the results tell you?
4. Refine your hypothesis. Repeat.

Track how many steps it takes you to find the root cause.
```

---

#### DQ-3.3: The Postmortem Study Method

```
Find a public postmortem. Read ONLY the "Impact" and "Symptoms" sections.
STOP. Don't read the root cause yet.

Write down:
1. Your top 3 hypotheses for what caused this
2. For each hypothesis, what evidence would confirm or rule it out
3. What would you check first and why

Then read the actual root cause.

Score yourself:
- Was the real cause in your top 3? → Strong diagnostic instincts
- Was it in a category you considered? → Decent instincts, need more specifics
- Was it something you never considered? → This is a blind spot — study it

Repeat weekly. Your hit rate will improve.
```

**Suggested postmortems to start with:**
- GitHub's 2018 database incident (network partition)
- Cloudflare's 2019 regex outage (CPU spike from bad deploy)
- Stripe's 2019 database migration (connection pool exhaustion)
- Any from https://github.com/danluu/post-mortems

---

#### DQ-3.4: The Observability Drill

```
Take your current project (even if it's simple). Answer:

1. If this breaks in production right now, how would you know?
   - Are there health checks?
   - Are there error alerts?
   - Are there logs? Where? In what format?
   - Is there any metrics/monitoring?

2. If a user reports "it's slow," what would you check?
   - Can you see request latency? Per-endpoint?
   - Can you see database query times?
   - Can you see memory/CPU usage?

3. If the answer to most of these is "no" — set up the basics:
   - Add request logging with timing (method, path, status, ms)
   - Add a health check endpoint
   - Add error logging that captures stack traces

This isn't exciting work, but it's the foundation of all debugging.
The best debuggers aren't smarter — they have better observability.
```

---

#### DQ-3.5: Read Code You Didn't Write

```
Pick an open-source project. Find a bug report in their issue tracker
that has been fixed. Read these in order:

1. The bug report (symptoms only)
2. Form your hypothesis
3. The discussion thread (watch how others diagnosed it)
4. The fix (the actual code change)

Answer:
- Was the fix where you expected?
- Was it simpler or more complex than you imagined?
- What debugging technique did the fixer use that you wouldn't have?

Do this once a week. After 10 bugs, you'll have a much broader
mental library of "things that go wrong and how to find them."
```

---

### Your Questions (Add Your Own)

```
#### DQ-3.X: [Title]
[Date added: YYYY-MM-DD]
[Context: why this question interests you]
[The question / exercise]
[What you learned: fill in after completing]
```

### Practice Log

| Date | Exercise | Time Spent | Key Insight | Confidence (1-5) |
|------|----------|-----------|-------------|-------------------|
| | | | | |

### Level Progression

```
Level 1: Can read an error message and find the relevant code
Level 2: Can form a hypothesis before diving into code
Level 3: Can systematically rule out causes (not just guess-and-check)
Level 4: Can debug across multiple components (frontend + API + database)
Level 5: Can diagnose issues from symptoms alone, before seeing any code
```

---

## Monthly Review Template

Every month, review your progress across all three areas:

```markdown
## Monthly Review — YYYY-MM

### Exercises Completed
- Area 1 (System Design): [count]
- Area 2 (Ambiguity): [count]
- Area 3 (Debugging): [count]

### Current Levels
- System Design: Level _/5
- Navigating Ambiguity: Level _/5
- Debugging: Level _/5

### Biggest Insight This Month
[one thing that changed how you think]

### What I'm Struggling With
[be honest — this is where growth happens]

### Focus for Next Month
[pick ONE area to emphasize]

### Questions I've Added
[list any new questions you wrote for yourself]
```

---

## How AI Accelerates This Training

Use AI not to skip the hard thinking, but to create richer training scenarios:

| Training Need | How AI Helps |
|---|---|
| System design review | "Critique this design. What would a principal engineer challenge?" |
| Ambiguity practice | AI simulates vague stakeholders you can interview |
| Debugging practice | AI simulates production systems with hidden bugs |
| Postmortem study | AI can explain unfamiliar infrastructure concepts in postmortems |
| Feedback on your thinking | "Here's my reasoning for choosing X. What am I not considering?" |

**The pattern:** Use AI to generate the PROBLEMS, not the solutions. You solve them. AI tells you what you missed. That's the training loop.

---

*This module grows with you. Add questions. Fill in practice logs. Update your levels honestly. Come back monthly.*

→ **Back to [README](../README.md) | Previous: [Module 06](06-onboarding-existing-projects.md)**
