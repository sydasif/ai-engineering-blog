---
title: "Getting Started with Claude Code: Your First AI Coding Partner"
date: 2026-05-28
tags: [ai, claude-code, tutorial, developer-tools]
description: "Learn how to install and use Claude Code, an agentic CLI for AI-assisted development."
---

# Getting Started with Claude Code: Your First AI Coding Partner

## Introduction

Claude Code is a pair programmer that never gets tired, explains every suggestion, and reads your entire project at once. But it's not a chat interface with a coding plugin. It's an agentic CLI that runs in your terminal, reads and writes files, manages Git, and reasons across your full codebase.

This guide covers what sets Claude Code apart, how to install it, and how to run your first prompt. By the end you'll know how to treat it as a collaborator rather than an autocomplete.

---

## Core Concepts

### What Claude Code Actually Is

Claude Code is Anthropic's official command-line interface for AI-assisted development. It runs in your terminal and can:

- Read and write files directly in your project
- Execute commands
- Manage Git workflows — branches, commits, pull requests
- Reason across up to 1 million tokens of codebase context
- Connect to external services via MCP (Model Context Protocol)
- Surface its reasoning in plain English before acting

It runs on Sonnet 4.6 by default for Pro users, Opus 4.6 on Max plans. Before taking any file-altering action, it asks for confirmation and shows its plan — that's the Constitutional AI framework in practice.

### How Claude Code Compares

| Feature | Claude Code                                    | Typical autocomplete tools       |
| ------- | ---------------------------------------------- | -------------------------------- |
| Type    | Agentic CLI                                    | IDE plugin or chat interface     |
| Focus   | Multi-file reasoning and agentic tasks         | Predicting the next line         |
| Context | Up to 1M tokens                                | Current file or limited window   |
| Actions | Reads/writes files, runs commands, manages Git | Suggests code you paste manually |
| Safety  | Confirms before file changes                   | Basic filtering                  |

Claude Code handles large-scale refactoring, cross-file bug fixes, and infrastructure tasks end to end. It's a different category from autocomplete.

---

## Installation and Your First Prompt

### Step 1 — Choose Your Access Method

Three ways to use Claude Code:

1. **Claude Web (claude.ai)** — No installation. Good for experimenting with prompts.
2. **Claude Code CLI** — The core tool. Runs from any terminal, works over SSH, scriptable in CI/CD pipelines.
3. **IDE Extension** — A VS Code extension (also works in Cursor, Windsurf, and Kiro) that surfaces inline diffs and accept/reject controls without leaving your editor.

#### Installing the CLI

Install Node.js 18+ first, then:

```bash
npm install -g @anthropic-ai/claude-code
```

Run `claude` in your terminal and complete the OAuth flow to authenticate. Verify with:

```bash
claude doctor
```

#### Setting Up in VS Code

Open the Extensions view (`Ctrl+Shift+X` on Windows/Linux, `Cmd+Shift+X` on Mac), search for **Claude Code**, and click Install. Sign in with your Anthropic account on first launch.

The extension also installs the first time you run `claude` from VS Code's integrated terminal. It works identically in Cursor, Windsurf, and other VS Code forks.

Once installed, click the Spark icon in the sidebar. You get real-time inline diffs, plan review before Claude touches any files, `@`-mention support for specific files and line ranges, and conversation history across tabs.

#### Setting Up in Cursor

Run Claude Code from Cursor's integrated terminal for heavy agentic tasks. Use Cursor's Composer for interactive editing. The two tools operate at different speeds and task scales — they complement each other.

#### Setting Up in Zed

Since Zed 1.0, Claude Code is native to Zed's Agent Panel via ACP (Agent Client Protocol). No separate installation. Zed is built in Rust with GPU-accelerated rendering.

**Pick your entry point:** VS Code if you're already there. Cursor for deep project-wide IDE features alongside agentic work. Zed for raw performance.

---

### Step 2 — Your First Prompt

Open your chosen interface and send this:

> _"Hello Claude! Please write a Python program that prints 'Hello, Claude!' and then explains, in a comment, what each line of the code does."_

Claude responds:

```python
# Assign the string 'Hello, Claude!' to the variable 'message'
message = 'Hello, Claude!'

# Use the print() function to display the value of 'message' in the console
print(message)
```

It explained each line without being asked. That's the default behavior — Claude assumes you want to understand the code, not just copy it.

Now push further:

> _"Rewrite it in JavaScript and explain the differences."_

Claude produces a JavaScript version and explains that `const` replaces Python's implicit assignment and `console.log()` replaces `print()`:

```javascript
// Assign the string 'Hello, Claude!' to the variable 'message' using const
const message = "Hello, Claude!";

// Use console.log() to display the value in the console
console.log(message);
```

### Step 3 — Iterate

> _"Update the Python version to ask for my name, greet me personally, and show the current time."_

Claude adds `input()`, imports `datetime`, and returns a runnable script:

```python
import datetime

# Ask the user for their name
name = input("What is your name? ")

# Greet the user by name
print(f"Hello, {name}!")

# Display the current system time
current_time = datetime.datetime.now().strftime("%H:%M:%S")
print(f"The current time is {current_time}.")
```

Each follow-up request revises Claude's reasoning and output. Treat it as a conversation, not a one-shot query.

---

## Best Practices

### For Beginners

- Start with the web interface — no setup, immediate feedback.
- Ask Claude to explain every decision. The reasoning is often more useful than the code.
- Make small, specific requests. Vague prompts produce vague output.

### For Experienced Developers

- Install the CLI and IDE extension. You'll cut context switching significantly.
- Use chain-of-thought prompts: ask Claude to "think step by step" before writing code on complex problems.
- Put your full codebase in context. Claude reads 1M tokens — use it on large files and documentation.
- Describe Git tasks in plain language. Claude Code creates branches, writes commit messages, and opens pull requests.

### Common Mistakes

- **Vague bug reports** produce vague fixes. Instead of "fix this bug," say: "The function should return X but returns Y. Here's the code and the error."
- **Ignoring Claude's explanations** wastes the tool's main advantage. Read the reasoning.
- **Single-prompt expectations** don't match how Claude works best. Iterate.
- **Using Claude Code for line completions** is like using a chainsaw to slice bread. Save it for multi-file, multi-step tasks.

---

## Conclusion

Claude Code is an agentic CLI that reads your codebase, executes commands, manages your Git workflow, and explains every decision. You now know:

- What it is and how it differs from autocomplete tools
- How to install the CLI and configure VS Code, Cursor, or Zed
- How to run and iterate on your first prompt

**Next step:** Install the CLI, send that first "Hello, Claude!" prompt, then try something harder — ask it to refactor a real file, add error handling, or commit a change. The scope of what it can do autonomously becomes clear quickly.

> **Acknowledgment:** Based on official Anthropic documentation and community research. Technical details reflect the state of Claude Code as of May 2026.
