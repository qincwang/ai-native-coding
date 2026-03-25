# Lab 02: Prompt Engineering for Code Generation

> Estimated time: 60-90 minutes

## Objective

Develop intuition for what makes effective prompts by experimenting with different styles and measuring outcomes.

---

## Exercise 1: The Prompt Spectrum

Pick a task: **Build a URL shortener API** (in-memory, no database).

Write 5 different prompts, from vague to precise, and try each one:

### Level 1 — One-liner
```
Build a URL shortener
```

### Level 2 — With tech stack
```
Build a URL shortener API using Express.js with in-memory storage
```

### Level 3 — With endpoints
```
Build a URL shortener API:
- POST /shorten { url } → { shortCode, shortUrl }
- GET /:code → 302 redirect to original URL
- GET /stats/:code → { url, clicks, created }
Express.js, in-memory storage, no database needed.
```

### Level 4 — With constraints
```
Build a URL shortener API:
- POST /shorten { url } → { shortCode, shortUrl }
  - Validate URL format, return 400 for invalid
  - Generate 6-char alphanumeric codes
  - Reject duplicate URLs, return existing code
- GET /:code → 302 redirect to original URL
  - 404 for unknown codes
  - Increment click counter
- GET /stats/:code → { url, clicks, createdAt }
Express.js, in-memory Map, port 3000.
No external dependencies except express.
```

### Level 5 — With examples & non-goals
```
[Level 4 prompt, plus:]

Example:
  POST /shorten {"url": "https://example.com"} → {"shortCode": "abc123", "shortUrl": "http://localhost:3000/abc123"}
  GET /abc123 → 302 → https://example.com

Non-goals:
- No authentication
- No database (in-memory only)
- No rate limiting
- No custom short codes
```

**For each level, record:**
- How many files were created?
- Did it work on first try?
- How many bugs/issues?
- Did it match your expectations?

---

## Exercise 2: The "Teach" vs "Tell" Experiment

### Task: Add input validation to an existing Express app

**Tell approach:**
```
Add this validation to POST /shorten:
- Check url is a string
- Check url starts with http:// or https://
- Check url length < 2048
- Return 400 with { error: "message" } if invalid
```

**Teach approach:**
```
Our validation philosophy: validate at the boundary, fail fast with helpful errors.
Add input validation to POST /shorten. The validation should:
- Reject anything that isn't a valid HTTP(S) URL
- Use standard HTTP status codes
- Return error messages that tell the caller exactly what's wrong and how to fix it
Look at how we handle errors elsewhere in the codebase for style guidance.
```

**Compare:** Which produces more thoughtful code? Which is easier to review?

---

## Exercise 3: Context Injection Patterns

### Pattern A: Read-first
```
Read src/routes/users.ts. Now add a similar route for products.
```

### Pattern B: Reference pattern
```
Add a products route following the same pattern as src/routes/users.ts.
Products have: id, name, price, category, inStock.
```

### Pattern C: Anti-pattern (no context)
```
Add a products route. Products have: id, name, price, category, inStock.
```

**Which produces code most consistent with the existing codebase?**

---

## Exercise 4: Negative Prompting

Sometimes telling the AI what NOT to do is as important as what to do.

### Without constraints:
```
Refactor the URL shortener to use TypeScript
```

### With negative constraints:
```
Refactor the URL shortener to use TypeScript:
- Don't add any new features
- Don't change the API contract
- Don't add unnecessary type utilities or generics
- Don't change the project structure
- Keep it simple — basic types, no type gymnastics
- All existing tests must still pass
```

**Observe:** How much "scope creep" does the unconstrained version introduce?

---

## Exercise 5: Build Your Prompt Template

Based on your experiments, create your personal prompt template:

```markdown
## [Task Title]

### What
[1-2 sentence description of the desired outcome]

### Requirements
- [specific requirement 1]
- [specific requirement 2]

### Constraints
- [what NOT to do]
- [boundaries to respect]

### Context
- [relevant files to read]
- [patterns to follow]

### Done When
- [measurable success criteria]
```

Save this template in `~/ai-native-coding/references/prompt-template.md`.

---

## Reflection

Journal entry:
1. At which prompt level did output quality plateau (more detail stopped helping)?
2. What was the most surprising difference between "tell" and "teach"?
3. What does your personal prompt template look like?
