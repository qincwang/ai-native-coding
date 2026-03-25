# Lab 06: Custom Commands, Hooks & MCP Skills

> Estimated time: 60-90 minutes

## Objective

Build your own Claude Code customizations: create custom slash commands, set up hooks for automated guardrails, and build a working MCP server that extends Claude Code with new capabilities.

---

## Setup

1. Open a terminal
2. Create a lab project:
   ```bash
   mkdir ~/lab-06-custom-claude && cd ~/lab-06-custom-claude
   npm init -y
   mkdir -p .claude/commands scripts mcp-servers/project-tools
   ```
3. Create a basic CLAUDE.md:
   ```bash
   echo "# Lab 06 Project\n\n## Build\n- npm test\n\n## Conventions\n- TypeScript with ESM\n- Tests with Node built-in runner" > CLAUDE.md
   ```
4. Start Claude Code: `claude`

---

## Exercise 1: Explore Built-In Commands

Before creating your own, get familiar with what's already available.

1. Type `/help` in Claude Code and read the output
2. Try `/init` — observe how it analyzes your project and suggests a CLAUDE.md
3. Try `/compact` — notice how it summarizes and compresses your conversation

**Observe:** How does each command behave differently from a normal prompt? What makes them feel like "commands" rather than conversations?

---

## Exercise 2: Create Your First Custom Command

Create a command that reviews code for common issues.

**Create the file `.claude/commands/check-quality.md`:**

```markdown
Review all source files in this project for quality issues.

Check for:
1. Functions longer than 30 lines
2. Deeply nested code (more than 3 levels)
3. Missing error handling on async operations
4. Magic numbers (unexplained numeric literals)
5. Console.log statements that should be removed

For each issue:
- State the file and line
- Explain why it's a problem
- Suggest a fix

If the code is clean, say so and explain what you checked.
```

Now test it:
```
/project:check-quality
```

**Observe:** How does Claude handle this compared to if you typed the same instructions as a regular message? (Hint: it should behave the same — commands are prompts, not magic.)

---

## Exercise 3: Create a Command with Arguments

Create a command that generates boilerplate for new modules.

**Create `.claude/commands/new-module.md`:**

```markdown
Create a new module: $ARGUMENTS

1. Create a source file in src/ with the appropriate name
2. Include:
   - TypeScript with ESM imports
   - A main exported class or function matching the module name
   - Input validation using type guards
   - JSDoc comments on public APIs
3. Create a matching test file in src/__tests__/
4. Include at least 3 meaningful test cases
5. Run the tests to verify they pass

Follow the patterns in existing source files if any exist.
```

Test it:
```
/project:new-module UserValidator - validates user input with email, age, and username fields
```

**Observe:** How well does `$ARGUMENTS` capture your intent? Try running it again with different arguments.

---

## Exercise 4: Create a Personal Command

Create a command in your user directory that works across all projects.

```bash
mkdir -p ~/.claude/commands
```

**Create `~/.claude/commands/summarize-changes.md`:**

```markdown
Summarize what has changed in this project recently.

1. Run `git log --oneline -10` to see recent commits
2. Run `git diff --stat HEAD~3` to see files changed in last 3 commits
3. Check for any uncommitted changes with `git status`

Present a brief summary:
- What was done (past 10 commits, grouped by theme)
- What's in progress (uncommitted changes)
- Any files that look like they need attention (large diffs, conflicts)

Keep it to 10 lines or less.
```

Test it from any project:
```
/user:summarize-changes
```

**Journal question:** What other personal commands would save you time across projects?

---

## Exercise 5: Set Up a Pre-Edit Hook

Create a hook that prevents edits to specific files.

**Create `scripts/guard-config.sh`:**

```bash
#!/bin/bash
# Block edits to config files without explicit mention in the prompt

FILE_PATH=$(echo "$1" | jq -r '.file_path // empty' 2>/dev/null)

if [[ "$FILE_PATH" == *"package.json"* ]] || [[ "$FILE_PATH" == *".env"* ]]; then
  echo "BLOCKED: This hook prevents direct edits to $FILE_PATH."
  echo "If you need to modify this file, ask the user to do it manually."
  exit 1
fi
```

Make it executable:
```bash
chmod +x scripts/guard-config.sh
```

**Create `.claude/settings.json`:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/guard-config.sh \"$TOOL_INPUT\""
      }
    ]
  }
}
```

Now test it by asking Claude to modify `package.json`:
```
Add a "description" field to package.json
```

**Observe:** Does the hook block the edit? What message does Claude see? How does it adapt?

---

## Exercise 6: Set Up a Post-Edit Hook

Create a hook that runs a check after every file edit.

**Create `scripts/post-edit-check.sh`:**

```bash
#!/bin/bash
# After any edit, check if the file has syntax issues

FILE_PATH=$(echo "$1" | jq -r '.file_path // empty' 2>/dev/null)

if [[ "$FILE_PATH" == *.js ]] || [[ "$FILE_PATH" == *.ts ]]; then
  # Quick syntax check using Node
  node --check "$FILE_PATH" 2>&1
  if [ $? -ne 0 ]; then
    echo "WARNING: Syntax error detected in $FILE_PATH"
    echo "Please fix before continuing."
  fi
fi
```

Add it to `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/guard-config.sh \"$TOOL_INPUT\""
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/post-edit-check.sh \"$TOOL_INPUT\""
      }
    ]
  }
}
```

**Observe:** Ask Claude to create a TypeScript file with a deliberate error. Does the hook catch it? How does Claude respond to the hook's feedback?

---

## Exercise 7: Build Your First MCP Server

Build a simple MCP server that provides project-related tools.

**Install dependencies:**

```bash
cd mcp-servers/project-tools
npm init -y
npm install @modelcontextprotocol/sdk zod
```

**Create `mcp-servers/project-tools/server.js`:**

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";
import { readFileSync, existsSync } from "fs";
import { execSync } from "child_process";

const server = new McpServer({
  name: "project-tools",
  version: "1.0.0",
});

// Tool 1: Get project stats
server.tool(
  "project_stats",
  "Get statistics about the current project (file counts, line counts, languages)",
  {},
  async () => {
    try {
      const files = execSync('find . -name "*.ts" -o -name "*.js" | grep -v node_modules', {
        encoding: "utf-8",
      }).trim().split("\n").filter(Boolean);

      let totalLines = 0;
      for (const file of files) {
        if (existsSync(file)) {
          const content = readFileSync(file, "utf-8");
          totalLines += content.split("\n").length;
        }
      }

      return {
        content: [{
          type: "text",
          text: `Project Stats:\n- Source files: ${files.length}\n- Total lines: ${totalLines}\n- Files: ${files.join(", ")}`,
        }],
      };
    } catch {
      return {
        content: [{ type: "text", text: "No source files found." }],
      };
    }
  }
);

// Tool 2: Check TODO items
server.tool(
  "find_todos",
  "Find all TODO and FIXME comments in the codebase",
  {
    pattern: z.string().optional().describe("Additional pattern to search for (default: TODO|FIXME)"),
  },
  async ({ pattern }) => {
    const searchPattern = pattern || "TODO|FIXME";
    try {
      const result = execSync(
        `grep -rn "${searchPattern}" --include="*.ts" --include="*.js" . | grep -v node_modules`,
        { encoding: "utf-8" }
      );
      return {
        content: [{ type: "text", text: result || "No TODOs found." }],
      };
    } catch {
      return {
        content: [{ type: "text", text: "No matches found." }],
      };
    }
  }
);

// Tool 3: Quick dependency check
server.tool(
  "check_deps",
  "Check if project dependencies are installed and list them",
  {},
  async () => {
    try {
      const pkg = JSON.parse(readFileSync("./package.json", "utf-8"));
      const deps = Object.keys(pkg.dependencies || {});
      const devDeps = Object.keys(pkg.devDependencies || {});
      const hasNodeModules = existsSync("./node_modules");

      return {
        content: [{
          type: "text",
          text: [
            `Dependencies (${deps.length}): ${deps.join(", ") || "none"}`,
            `Dev dependencies (${devDeps.length}): ${devDeps.join(", ") || "none"}`,
            `node_modules: ${hasNodeModules ? "installed" : "NOT installed — run npm install"}`,
          ].join("\n"),
        }],
      };
    } catch {
      return {
        content: [{ type: "text", text: "No package.json found." }],
      };
    }
  }
);

const transport = new StdioServerTransport();
await server.connect(transport);
```

Add `"type": "module"` to the MCP server's `package.json`:
```bash
cd mcp-servers/project-tools
node -e "const p=JSON.parse(require('fs').readFileSync('package.json'));p.type='module';require('fs').writeFileSync('package.json',JSON.stringify(p,null,2))"
```

**Register the server in `.claude/settings.json`:**

Update your settings to include:

```json
{
  "mcpServers": {
    "project-tools": {
      "command": "node",
      "args": ["./mcp-servers/project-tools/server.js"]
    }
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/guard-config.sh \"$TOOL_INPUT\""
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "command": "bash ./scripts/post-edit-check.sh \"$TOOL_INPUT\""
      }
    ]
  }
}
```

Restart Claude Code and test:
```
Use the project_stats tool to tell me about this project
```

```
Find all TODOs in the codebase
```

**Observe:** Do the MCP tools appear alongside Claude's built-in tools? How does Claude decide when to use them?

---

## Exercise 8: Combine Everything

Now put it all together. Create a custom command that uses your hooks and MCP server.

**Create `.claude/commands/health-check.md`:**

```markdown
Run a full health check on this project.

1. Use the `project_stats` tool to get an overview
2. Use the `check_deps` tool to verify dependencies
3. Use the `find_todos` tool to find outstanding work
4. Run any available tests
5. Check git status for uncommitted changes

Present a dashboard-style summary:
- Project size and health
- Dependency status
- Outstanding TODOs
- Test results
- Git cleanliness

Flag anything that needs attention.
```

Test it:
```
/project:health-check
```

**Observe:** Watch how Claude orchestrates built-in tools, MCP tools, and shell commands together in a single workflow — all triggered by your custom command.

---

## Reflection

Write a journal entry in `~/ai-native-coding/journal/` answering:

1. Which customization (commands, hooks, or MCP) do you see yourself using most? Why?
2. What command would save your team the most time if you created it tomorrow?
3. How does the combination of commands + hooks + MCP change what "using an AI coding tool" means?
4. What MCP server would be most valuable for your specific work? What tools would it expose?
