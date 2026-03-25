# Module 01: Foundations

## 1.1 What is AI-Native Coding?

**Traditional coding**: Human writes every line. Tools assist (syntax highlighting, linting, autocomplete).

**AI-assisted coding**: Human writes most code. AI suggests completions (Copilot-style). Human is still the "typist."

**AI-native coding**: Human defines intent, constraints, and architecture. AI agents generate, test, and iterate on implementations. Human reviews, steers, and decides.

The key shift: **you stop being the writer and become the editor-in-chief.**

### The Spectrum

```
Manual Coding → Autocomplete → Copilot → Chat-based → Agent-based → Autonomous
     ↑                                                        ↑
  You write everything                              You describe what you want,
                                                    AI builds it, you review
```

Most developers in 2026 are somewhere between "Chat-based" and "Agent-based." The frontier is pushing toward autonomous — where agents handle entire feature branches with minimal supervision.

---

## 1.2 How Large Language Models Work (For Developers)

You don't need a PhD in ML, but you need accurate mental models. Here's what matters:

### The Core Mechanism: Next-Token Prediction

LLMs predict the most likely next token given all previous tokens. That's it. Everything else — reasoning, coding, creativity — emerges from this single mechanism applied at massive scale.

```
Input tokens: "def fibonacci(n):\n    if n <= 1:\n        return"
Model predicts: " n" (with high probability)
```

### Why This Matters for Coding

1. **LLMs are pattern matchers, not compilers.** They can generate code that *looks* right but has subtle logic errors. Always verify.
2. **Context is everything.** The model only knows what's in its context window. If you don't provide relevant code/docs, it will hallucinate plausible-but-wrong details.
3. **Probability, not certainty.** The same prompt can yield different outputs. This isn't a bug — it's the nature of the system.

### Key Technical Concepts

| Concept | What It Means | Why You Care |
|---------|---------------|--------------|
| **Tokens** | Chunks of text (~4 chars each). The unit of input/output. | Longer context = more cost, but also more awareness. |
| **Context Window** | Max tokens the model can "see" at once (e.g., 200K for Claude). | Determines how much code/docs you can feed in one shot. |
| **Temperature** | Controls randomness. 0 = deterministic, 1 = creative. | Lower for code generation, higher for brainstorming. |
| **System Prompt** | Instructions that shape the model's behavior for a conversation. | This is how tools like Claude Code configure the agent. |
| **Fine-tuning vs. Prompting** | Fine-tuning changes model weights; prompting guides behavior at inference time. | You'll mostly prompt. Fine-tuning is for platform builders. |

### The Transformer Architecture (Simplified)

```
Input Text
    ↓
Tokenization (text → numbers)
    ↓
Embedding (numbers → vectors with meaning)
    ↓
Attention Layers (×many) ← This is where the magic happens
    │   "Which parts of the input are relevant to each other?"
    ↓
Output Probabilities
    ↓
Token Selection (sampling/greedy)
    ↓
Output Text
```

**Attention** is the breakthrough: it lets the model weigh the relevance of every token to every other token. When generating code, it can "attend" to a function signature 500 lines above to produce the correct implementation below.

---

## 1.3 How Claude Works Specifically

Claude is built by Anthropic using the transformer architecture with several key design choices:

### Claude's Distinguishing Characteristics

1. **Constitutional AI (CAI)**: Claude is trained with a set of principles ("constitution") that guide behavior. This is why it tends to be careful, thorough, and will push back on harmful requests.

2. **Long Context**: Claude supports up to 200K tokens of context — roughly 150K words or ~500 pages of code. This means you can feed entire codebases.

3. **Tool Use / Function Calling**: Claude can invoke external tools (read files, run commands, search the web, call APIs). This is what makes "agent" mode possible — Claude doesn't just generate text, it *takes actions*.

4. **Extended Thinking**: Claude can engage in step-by-step reasoning before responding. For complex coding tasks, this means more accurate solutions.

### Claude's Model Family (as of 2026)

| Model | Strengths | Best For |
|-------|-----------|----------|
| **Opus 4.6** | Highest capability, deep reasoning | Complex architecture, multi-file refactors, subtle bugs |
| **Sonnet 4.6** | Fast + capable balance | Day-to-day coding, feature implementation |
| **Haiku 4.5** | Fastest, lowest cost | Quick edits, simple generation, high-volume tasks |

### How Claude Code Works

Claude Code is a CLI-based coding agent. Here's the architecture:

```
You (human) ←→ Claude Code CLI ←→ Claude API
                    ↓
              Tool Execution Layer
              ├── File Read/Write
              ├── Bash Commands
              ├── Code Search (Glob/Grep)
              ├── Web Search/Fetch
              └── Sub-agents (parallel workers)
```

**The loop**:
1. You describe what you want
2. Claude reads relevant code to understand context
3. Claude plans an approach (sometimes with extended thinking)
4. Claude makes changes (edits files, runs commands)
5. Claude verifies (runs tests, checks output)
6. You review and provide feedback
7. Repeat until done

This is the **human-in-the-loop agent pattern** — the most productive paradigm in 2026.

---

## 1.4 The Standard AI-Native Workflow

### Phase 1: Context Loading

Before asking an AI agent to do anything, it needs context. The quality of output is directly proportional to the quality of context.

**What to provide:**
- The specific files involved
- Relevant documentation or specs
- Examples of desired behavior
- Constraints and requirements
- Existing patterns in the codebase

**How Claude Code does this automatically:**
- Reads `CLAUDE.md` files (project-level instructions)
- Uses memory from past conversations
- Searches the codebase with Glob/Grep before making changes
- Reads files before editing them

### Phase 2: Intent Expression

This is the art of telling the AI *what* you want without over-specifying *how*.

**Too vague:** "Fix the bug"
**Too specific:** "On line 47 of auth.py, change the == to !="
**Just right:** "Users are getting logged out after 5 minutes instead of 30. The session timeout logic is in auth.py. Fix the timeout calculation."

The sweet spot: **clear intent + relevant context + constraints**.

### Phase 3: Generation & Iteration

The agent generates a solution. Rarely is the first attempt perfect. The iteration loop:

```
Generate → Review → Feedback → Regenerate → Review → Accept
```

**Effective feedback examples:**
- "This works but it's not handling the edge case where X is null"
- "Good approach, but use the existing `validateInput()` helper instead of inline validation"
- "The logic is right but this needs to follow our existing pattern in `services/`"

### Phase 4: Verification

Never trust AI-generated code blindly. Verification methods:

1. **Read the diff** — understand what changed and why
2. **Run existing tests** — catch regressions immediately
3. **Write new tests** — for new functionality
4. **Manual testing** — for UI/UX changes
5. **Static analysis** — linters, type checkers
6. **Code review** — same as any other code

### Phase 5: Integration

Once verified, the code follows normal engineering practices: commit, PR, review, merge. The fact that an AI wrote it doesn't change the standard of quality.

---

## 1.5 Iteration Methods

### Method 1: Conversational Refinement
Keep talking to the agent in the same session. Each message adds context.
- Best for: exploratory work, unclear requirements
- Risk: context window bloat if conversation gets too long

### Method 2: Diff-Review-Feedback
Let the agent make changes, review the diff, give targeted feedback.
- Best for: implementation tasks with clear specs
- Risk: none — this is the gold standard workflow

### Method 3: Test-Driven AI Development
Write tests first, then let the agent implement until tests pass.
- Best for: well-defined functionality, APIs, algorithms
- Risk: tests themselves might be wrong — verify those too

### Method 4: Parallel Exploration
Spawn multiple agents to try different approaches simultaneously. Compare results.
- Best for: architectural decisions, performance optimization
- Risk: higher cost, need to evaluate multiple solutions

### Method 5: Progressive Delegation
Start by doing it yourself with AI assistance. As trust builds, delegate more.
- Best for: onboarding to a new codebase or team
- Risk: none — this is the safest learning path

---

## 1.6 Key Principles to Internalize

1. **AI amplifies your intent, not your laziness.** Vague inputs produce vague outputs. Clear thinking produces clear code.

2. **Context is the bottleneck, not intelligence.** Claude is smart enough. The question is whether it has enough information to make the right decision.

3. **Review is non-negotiable.** The fastest way to ship a bug is to blindly accept AI-generated code. You are the quality gate.

4. **Iteration is cheaper than perfection.** Don't spend 20 minutes crafting the perfect prompt. Spend 2 minutes on a good-enough prompt, then iterate.

5. **The agent is your pair programmer, not your replacement.** You bring domain knowledge, taste, and judgment. The agent brings speed, breadth, and tirelessness.

---

## Self-Check Questions

- [ ] Can you explain to a non-technical person how an LLM generates code?
- [ ] What is the difference between context window and training data?
- [ ] Why does the same prompt sometimes produce different outputs?
- [ ] What are the 5 phases of the standard AI-native workflow?
- [ ] Which iteration method would you use for a well-defined API endpoint?

→ **Next: [Module 02 - AI Coding Concepts & Terminology](02-ai-coding-concepts.md)**
