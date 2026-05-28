---
title: "Prompting Foundations for Developers: How to Talk to Claude So It Listens"
date: 2026-05-28
tags: [ai, claude-code, prompting, developer-experience]
description: "Master the art of prompting Claude Code to get production-ready results instead of generic output."
---

# Prompting Foundations for Developers: How to Talk to Claude So It Listens

## Introduction

You've installed Claude Code. You've sent a few prompts. But when you ask for something real — "write a login system" — the output feels generic, incomplete, or misses the point entirely.

The problem is almost always the prompt. That's fixable.

This chapter covers the exact structures and habits that turn vague requests into production-ready code: what Claude needs to know before it writes anything, how to trigger its reasoning instead of its pattern-matching, how to feed it multiple files without losing context, and how to lock in project-wide standards with `CLAUDE.md`.

---

## What Claude Needs Before It Writes

Before Claude produces a single line, it resolves three things from your prompt:

1. **What to build.** "A login system" could mean a simple password check, a full OAuth flow, or a JWT-based API endpoint. Claude picks one. If you didn't specify, it guessed.

2. **How it should behave.** Should it block on errors or return structured error responses? Log failures? Validate inputs? Hash passwords? Silence on these means Claude chooses defaults — often the simplest ones.

3. **What the result should look like.** A single function? A module? A class with docstrings? Raw code or code with explanation?

Leave any of these unanswered and Claude fills the gap. The output won't be wrong exactly — it just won't be what you had in mind.

A useful mental check before sending: _What, How, Format._ Three seconds to answer those three questions will consistently improve what comes back.

---

## Writing Prompts That Work

### Weak vs. Strong

**Weak:**

> "Write a function to handle user authentication."

Claude produces a bare-bones username/password check. No error handling, no hashing, no structure. Technically functional, not shippable.

**Strong:**

> "Write a Python function using FastAPI that handles user authentication with JWT tokens. Verify credentials against the database, issue tokens that expire in 15 minutes, and return structured JSON responses. Include docstrings and a comment explaining each step."

Four additions: framework (FastAPI), language (Python), expected behavior (JWT, expiration, JSON responses), output format (docstrings, comments). Claude now knows what "done" looks like.

### Chain-of-Thought Prompting

Claude reasons more carefully when you ask it to reason before it codes. This is chain-of-thought prompting — and it's worth knowing how it fits alongside Claude's newer capabilities.

**Without it:**

> "Write a FastAPI endpoint that calculates factorial."

Claude often produces a recursive function with no input validation. It works for positive integers. It breaks on negatives and very large values.

**With it:**

> "Before coding, outline your approach for a FastAPI endpoint that accepts a number and returns its factorial. Then write the full implementation with input validation, error handling, and comments."

Claude first lists its plan — input schema, recursion vs. iteration, edge cases (negative numbers, zero, large values), JSON output — then writes code that matches it. The result is safer and easier to debug.

**One important nuance:** Chain-of-thought prompting was the standard approach before Claude's _extended thinking_ feature. Extended thinking (available on Sonnet 4.6 and Opus 4.6/4.7) gives Claude a separately-budgeted reasoning pass before generating output — you can inspect that reasoning trace in the API response. For complex coding tasks, extended thinking generally produces better results than manual chain-of-thought prompting. If you're on a plan that supports it, enable it for architecture decisions, complex debugging, and multi-step refactors. Manual chain-of-thought remains useful when extended thinking isn't available or when you need the reasoning to appear inline in Claude's response.

For either approach, Anthropic's docs note one consistent finding: always have Claude output its thinking. If the reasoning doesn't appear in the response, no reasoning step ran.

### Multi-File Prompts

Real projects span multiple files. Claude handles them well — if you structure the input clearly.

**Bad approach:** Dump five files into a prompt with no labels. Claude loses track of which code belongs to which file and what the relationship between them is.

**Good approach:** Label each file with a clear header and state your goal upfront.

```
Claude, I'll share two FastAPI files. Analyze how validation is handled
across both and refactor them to remove redundancy.

## FILE: validators.py
[code here]

## FILE: main.py
[code here]

Provide the complete corrected code for both files.
```

Claude reads the structure, understands the relationship between modules, and refactors across both in one pass. The 1M-token context window on Sonnet 4.6 and Opus 4.6/4.7 means you can feed in large codebases this way — many files, full documentation, test suites.

### CLAUDE.md: Project-Wide Standards

The blog's original framing of "system prompts" is accurate for API usage, but in Claude Code the standard mechanism is `CLAUDE.md` — a file you place in your project root (or in subdirectories for local rules). Claude reads it automatically at session start.

A `CLAUDE.md` might contain:

```markdown
# Project Standards

- Language: Python 3.12, FastAPI 0.115
- Always include docstrings on public functions
- Use Pydantic v2 models for request/response schemas
- Never use eval() or exec()
- All database queries go through the repository layer — no raw SQL in route handlers
- Secrets come from environment variables only
```

Claude automatically merges multiple `CLAUDE.md` files based on directory structure — global rules in the root, local overrides in subdirectories. This mirrors how senior engineers think: project-wide constellations plus file-specific constraints.

For API usage, system prompts work the same way — set them once, and every request in the session inherits the standards.

**On security guardrails specifically:** treat prompts as logs. Assume they can be inspected. Don't put secrets or credentials in prompts. Lock down tool permissions to what Claude needs for the task. For anything touching authentication or access control, keep a human in the review loop.

---

## Best Practices

### For Beginners

- Run the _What, How, Format_ check before every prompt.
- Paste actual error messages when debugging. Claude diagnoses errors well when it sees the full stack trace, not a paraphrase of it.
- Ask Claude to explain its reasoning: "Why did you choose an iterative approach over recursion?" The answer teaches you how it's modeling the problem.

### For Experienced Developers

- Commit a `CLAUDE.md` to every project. Keep it short and explicit — specific rules outperform long documentation.
- Label files clearly in multi-file prompts. State the goal at the top, then paste the files.
- Use extended thinking for complex tasks. Set effort to `xhigh` for coding and agentic work — Anthropic's own docs recommend this as the default for most development use cases.
- For very long autonomous runs, use sub-agents: "You just wrote the payment processor. Now use a sub-agent to run a security review on that code." This keeps the main conversation focused while the review runs in a separate context.

### Common Prompting Errors

| Problem                              | Likely Cause                                              | Fix                                                          |
| ------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| Generic output                       | Prompt missing framework, format, or behavior constraints | Add specific language, library, and output requirements      |
| Output cuts off                      | Hitting max token limit                                   | Ask for output in sections, or summarize then expand         |
| Claude ignores part of the prompt    | Too too many instructions at once                         | Break into sequential prompts                                |
| Wrong imports or syntax              | Missing version context                                   | Specify library versions: "FastAPI 0.115, Pydantic v2"       |
| Claude contradicts earlier decisions | Context drift                                             | Re-anchor: "We're still using FastAPI 0.115 with PostgreSQL" |

---

## Prompts as Dialogue

The developers who get the most out of Claude treat prompting as a conversation, not a one-shot command. Start broad enough to test Claude's interpretation of the problem. Refine based on what comes back. Ask it to defend its choices.

Each iteration improves the output. Over time you develop a sense for when to be specific and when to leave room for Claude to interpret — when a tight spec produces better code and when a looser framing surfaces a better architecture. Anthropic's own best practices guide puts it plainly: sometimes a vague prompt is exactly right, because you want to see how Claude interprets the problem before constraining it.

---

## Conclusion

Prompting is a skill with a short learning curve and a long ceiling. The foundations: answer _What, How, Format_ before you send anything; use chain-of-thought or extended thinking for tasks that require reasoning; label multi-file inputs clearly, store project standards in `CLAUDE.md` so you don't repeat yourself every session.

**Try this:** Take a piece of code you wrote recently. Write a weak prompt — just "improve this function." Then write a strong one using the three questions. Compare what Claude returns. The difference will be obvious, and you won't go back to vague prompts.

> **Acknowledgment:** Based on official Anthropic documentation and community research. Technical details reflect the state of Claude Code as of May 2026.
