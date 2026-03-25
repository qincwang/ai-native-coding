# Lab 01: Your First AI-Native Development Cycle

> Estimated time: 45-60 minutes

## Objective

Experience the full AI-native workflow: **Context → Intent → Generation → Verification → Integration** by building a small utility from scratch using Claude Code.

---

## Setup

1. Open a terminal
2. Create a new project directory:
   ```bash
   mkdir ~/lab-01-calculator && cd ~/lab-01-calculator
   npm init -y
   ```
3. Start Claude Code: `claude`

---

## Exercise 1: Context Loading via CLAUDE.md

Create a `CLAUDE.md` file for this tiny project. Tell Claude:

```
Create a CLAUDE.md file for this project. It's a Node.js CLI calculator
that supports basic math operations. We use:
- Node.js 20+
- No external dependencies (stdlib only)
- Tests with Node's built-in test runner (node --test)
- ESM modules
```

**Observe:** How does Claude set up the file? What structure does it choose?

---

## Exercise 2: Intent Expression

Now ask Claude to build the calculator. Try these prompts in order and observe the difference in output quality:

**Prompt A (vague):**
```
Build a calculator
```

**Prompt B (specific):**
```
Build a CLI calculator (calc.js) that:
- Takes input like: node calc.js "2 + 3"
- Supports: +, -, *, /, ** (power), % (modulo)
- Handles operator precedence correctly
- Returns just the numeric result to stdout
- Exits with code 1 and an error message for invalid input
```

**Journal question:** How did the outputs differ? What made Prompt B better?

---

## Exercise 3: Test-Driven Iteration

```
Write tests for the calculator in calc.test.js using Node's built-in test runner.
Include tests for:
- Basic operations (2+2, 10-3, etc.)
- Operator precedence (2+3*4 should be 14)
- Power and modulo
- Error cases (invalid input, division by zero)
Then run the tests.
```

**Observe:** Does Claude write tests first, then check if they pass? Does it fix failures automatically?

---

## Exercise 4: Review & Feedback Loop

After Claude generates the code:

1. Read the diff carefully
2. Give specific feedback:
   ```
   The calculator works but I want it to also support parentheses: "( 2 + 3 ) * 4"
   ```
3. Watch how Claude iterates

**Journal question:** How many iterations did it take? What was your feedback pattern?

---

## Exercise 5: Verification Checklist

Run through this checklist:
- [ ] All tests pass (`node --test calc.test.js`)
- [ ] Handles edge cases (empty input, spaces, very large numbers)
- [ ] Code is readable without comments explaining obvious things
- [ ] Error messages are helpful

---

## Reflection

Write a journal entry in `~/ai-native-coding/journal/` answering:
1. What surprised you about the AI-native workflow?
2. Where did you add the most value (vs. where the AI added value)?
3. What would you do differently next time?
