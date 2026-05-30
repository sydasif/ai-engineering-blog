---
title: "Debugging and Code Review with Claude Code"
date: 2026-05-28
tags: [debugging, code-review, tutorial, developer-tools]
description: "Learn how to use Claude Code as a debugging and code review partner to identify bugs, automate reviews, and build an iterative fix loop."
---

# Debugging and Code Review with Claude Code

## Introduction

You've written the code. It looks right. But when you run it, something breaks. The error message is cryptic, the stack trace points everywhere, and you've been staring at the same five lines for an hour.

Debugging is time-consuming. Claude Code makes it faster — not by magically finding bugs, but by reading your code, understanding what you intended it to do, and walking you through exactly what went wrong. This chapter covers how to use Claude Code as a debugging and code review partner: identifying bugs, automating reviews, testing fixes in real time, and building an iterative loop that improves your output.

---

## How Claude Code Approaches Debugging

Claude Code doesn't just respond to your prompt in isolation. It runs an agentic loop: gather context, take action, verify results, repeat. For debugging, that looks like this:

1. **Context gathering** — Claude reads your code, file structure, imports, and dependencies. It looks for missing definitions, misused libraries, and structural issues.
2. **Intent comparison** — Claude compares your code's actual logic against its described purpose (from your prompt, docstrings, or comments) and identifies mismatches.
3. **Fix and verification** — Claude proposes a corrected version, explains its reasoning, and generates test cases you can run immediately.

You're part of this loop. You can interrupt at any point to provide more context, redirect Claude's approach, or push back on a suggestion. Claude works autonomously but stays responsive to your input.

### Example: Fixing a Flask Discount Calculator

**Buggy code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/discount", methods=["POST"])
def discount():
    data = request.get_json()
    price = data.get("price")
    discount_rate = data.get("discount")
    discounted_price = price * discount_rate
    return jsonify({"discounted_price": discounted_price})
```

You expect `{"price": 100, "discount": 0.2}` to return `80.0`. It returns `20.0` — the discount amount, not the final price.

**Prompt:**

> "This Flask endpoint should return the final price after applying a discount, but it's returning the discount value itself. Find and fix the issue. Include the corrected code and explain what was wrong."

**Claude's fix:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route("/discount", methods=["POST"])
def discount():
    """Calculate the final price after applying a discount."""
    data = request.get_json()
    price = data.get("price")
    discount_rate = data.get("discount")

    if price is None or discount_rate is None:
        return jsonify({"error": "Missing required fields"}), 400

    if not isinstance(price, (int, float)) or not isinstance(discount_rate, (int, float)):
        return jsonify({"error": "Invalid input types"}), 400

    discounted_price = price * (1 - discount_rate)
    return jsonify({"discounted_price": round(discounted_price, 2)})
```

Claude's explanation: the original formula computed the discount amount (`price * discount_rate`) instead of the final price (`price * (1 - discount_rate)`). It also added input validation to handle missing or wrongly-typed fields — a common real-world failure mode the original code silently ignored.

---

## Automated Code Review

Claude can read, analyze, and critique code in real time — flagging logic issues, security risks, and performance problems before they hit production.

The workflow is three steps:

1. **Context ingestion** — provide the relevant file or diff. Claude builds an internal model of what the code does.
2. **Evaluation** — Claude reviews logic, structure, naming, performance, and maintainability, and explains why each issue matters.
3. **Structured feedback** — Claude returns actionable output you can use directly in pull request comments.

### Example: Reviewing a FastAPI Endpoint

**Code under review:**

```python
from fastapi import FastAPI
import sqlite3

app = FastAPI()

@app.get("/users")
def get_users():
    conn = sqlite3.connect("database.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    users = cursor.fetchall()
    conn.close()
    return {"users": users}
```

At first glance this works. Claude will catch four issues: it uses synchronous SQLite inside an async framework, blocking the event loop; there's no error handling; it selects all columns with `SELECT *` rather than explicit fields; and future parameter additions could introduce SQL injection.

**Prompt:**

> "Review this FastAPI endpoint for code quality, security, and performance. Identify issues, explain their impact, and propose corrected code following current best practices."

**Claude's suggested fix** (using the current `lifespan` pattern — `@app.on_event` has been deprecated since FastAPI 0.93):

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI, HTTPException
from databases import Database

DATABASE_URL = "sqlite:///database.db"
database = Database(DATABASE_URL)

@asynccontextmanager
async def lifespan(app: FastAPI):
    await database.connect()
    yield
    await database.disconnect()

app = FastAPI(lifespan=lifespan)

@app.get("/users")
async def get_users():
    """Retrieve all users asynchronously."""
    try:
        query = "SELECT id, username, email FROM users"
        users = await database.fetch_all(query=query)
        return {"users": [dict(user) for user in users]}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

Key changes: async I/O via the `databases` library, explicit column selection, proper error handling, and the `lifespan` context manager instead of the deprecated `@app.on_event` decorators.

### Review Focus Areas

| Area        | What Claude Checks                                 | Example Feedback                                |
| ----------- | -------------------------------------------------- | ----------------------------------------------- |
| Security    | SQL injection, unsafe evals, hardcoded secrets     | "Avoid string interpolation in SQL queries."    |
| Performance | Blocking I/O, missing caching, inefficient queries | "Replace sync DB calls with async equivalents." |
| Readability | Naming, docstrings, structural clarity             | "Add docstrings to API routes."                 |
| Compliance  | PEP8, project conventions, CLAUDE.md rules         | "Use context managers for database sessions."   |

---

## Interpreting Claude's Feedback

Claude's suggestions are starting points, not commands. Two things determine whether to accept them:

**Does it make sense in your environment?** A "best practice" in one context is wrong in another. Claude's context is limited to what you provided. If you omitted key files or constraints, its feedback will reflect those gaps.

**Does it match your intent?** Claude reasons about what the code probably needs. You reason about what it actually needs. Those aren't always the same.

### Example: Evaluating a Suggestion You Should Modify

**Original code:**

```python
class FileReader:
    def __init__(self, file_path):
        self.file_path = file_path

    def read(self):
        file = open(self.file_path, "r")
        data = file.read()
        file.close()
        return data
```

**Claude's feedback:** "Use a context manager (`with` statement) to ensure the file closes automatically even if an error occurs."

**Claude's suggested fix:**

```python
class FileReader:
    def __init__(self, file_path, chunk_size=8192):
        self.file_path = file_path
        self.chunk_size = chunk_size

    def read(self):
        """Read file in chunks for memory efficiency."""
        try:
            with open(self.file_path, "r", encoding="utf-8") as file:
                for chunk in iter(lambda: file.read(self.chunk_size), ""):
                    yield chunk
        except FileNotFoundError:
            raise FileNotFoundError(f"File not found: {self.file_path}")
        except Exception as e:
            raise RuntimeError(f"Error reading file: {e}")
```

The developer accepted the context manager suggestion but adapted the rest for their actual needs — chunked reading for large files. Claude's reasoning ("safe file handling") was the right principle; the specific implementation needed adjustment for the workload.

### Feedback Types

| Type            | What It Means                            | How to Evaluate                             |
| --------------- | ---------------------------------------- | ------------------------------------------- |
| Corrective      | Fixes an error or unsafe pattern         | Test in your environment before accepting   |
| Advisory        | Suggests a best practice                 | Check relevance to your project standards   |
| Speculative     | "You could also..." optional improvement | Evaluate the trade-off before adopting      |
| Context-limited | Based on incomplete information          | Re-prompt with the missing files or context |

---

## Testing Fixes in Real Time

The fastest path from bug to fix is an iterative loop: show Claude the problem, get a fix, test it, feed results back. Claude participates in every step.

**The loop:**

1. Show Claude the buggy code and describe what's wrong.
2. Claude suggests a fix with explanation.
3. Run the fix locally, or ask Claude to generate test cases.
4. Feed results back — if it still fails, Claude reasons about why and adjusts.
5. Repeat until resolved.

### Example: Empty List Error

**Buggy code:**

```python
def calculate_average(numbers):
    total = sum(numbers)
    return total / len(numbers)
```

Calling `calculate_average([])` raises `ZeroDivisionError`.

**Prompt:**

> "This function crashes on empty lists. Fix it and show how to test it."

**Claude's fix:**

```python
def calculate_average(numbers):
    """Calculate the average of a list of numbers. Returns 0 for empty lists."""
    if not numbers:
        return 0
    return sum(numbers) / len(numbers)
```

**Claude's tests:**

```python
print(calculate_average([10, 20, 30]))  # Expected: 20.0
print(calculate_average([]))            # Expected: 0
print(calculate_average([5]))           # Expected: 5.0
```

**Developer follow-up:** "Tests pass. But I'd prefer a `ValueError` instead of returning 0 for empty lists."

**Claude's adjustment:**

```python
def calculate_average(numbers):
    """Calculate the average of a list of numbers. Raises ValueError for empty input."""
    if not numbers:
        raise ValueError("Cannot calculate average of an empty list.")
    return sum(numbers) / len(numbers)
```

**Updated test:**

```python
try:
    print(calculate_average([10, 20, 30]))  # Expected: 20.0
    calculate_average([])                   # Expected: ValueError
except ValueError as e:
    print(f"Caught expected exception: {e}")
```

One follow-up message changed the behavior contract without restarting the conversation. That's the loop working as intended.

---

## Human-in-the-Loop Debugging

Claude is most effective when you steer the reasoning rather than hand it the wheel entirely. The productive pattern is:

| Phase                 | Your Role                                              | Claude's Role                                |
| --------------------- | ------------------------------------------------------ | -------------------------------------------- |
| Framing               | Define the problem, expected behavior, and environment | Build an understanding of the context        |
| Hypothesis generation | Evaluate theories for relevance to your setup          | Propose likely causes and affected areas     |
| Validation            | Run the code, confirm or disprove                      | Interpret results, propose next steps        |
| Refinement            | Provide feedback on each fix                           | Adjust the approach based on new information |

### Example: API Timeout Bug

**Buggy code:**

```python
import requests

def fetch_data(url):
    response = requests.get(url)
    return response.json()
```

The function occasionally hangs when an API is slow.

**Prompt:** "This function sometimes gets stuck waiting for a response. The API is valid. Can you help find out why?"

**Claude's diagnosis:** `requests.get()` waits indefinitely by default. The `timeout` parameter controls this.

**Claude's fix:**

```python
import requests

def fetch_data(url, timeout=5):
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.Timeout:
        print("Request timed out.")
        return None
    except requests.exceptions.RequestException as e:
        print(f"An error occurred: {e}")
        return None
```

**Follow-up:** "This works, but I still get a `JSONDecodeError` sometimes."

**Claude:** "That happens when the response isn't valid JSON — usually a server returning HTML for an error state. Check the `Content-Type` header before parsing."

**Refined version:**

```python
import requests
import json

def fetch_data(url, timeout=5):
    try:
        response = requests.get(url, timeout=timeout)
        response.raise_for_status()

        if "application/json" not in response.headers.get("Content-Type", ""):
            print("Response is not JSON.")
            return None

        return response.json()

    except json.JSONDecodeError:
        print("Invalid JSON in response.")
        return None
    except requests.exceptions.Timeout:
        print("Request timed out.")
        return None
    except requests.exceptions.RequestException as e:
        print(f"An error occurred: {e}")
        return None
```

Each follow-up pushed the fix one step further. Claude didn't invent the second bug — you found it through testing and brought it back.

---

## Collaborative Debugging in Practice

### Example: Email Validator

**Initial code:**

```python
import re

def is_valid_email(email):
    pattern = r"^[a-zA-Z0-9]+@[a-zA-Z]+\.[a-zA-Z]+$"
    return bool(re.match(pattern, email))
```

This rejects valid addresses like `john.doe@sub.domain.org`.

**Prompt:** "This regex rejects some valid emails like 'john.doe@sub.domain.org'. Debug and improve it — keep the function simple and secure."

**Claude's diagnosis:** The pattern disallows dots and hyphens in both the local part and domain, and doesn't handle subdomains.

**Claude's fix:**

```python
import re

def is_valid_email(email):
    """Validate an email address. Allows dots, hyphens, and subdomains."""
    pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    return bool(re.match(pattern, email))
```

**Test suite:**

```python
emails = [
    "user@example.com",
    "john.doe@sub.domain.org",
    "user-name@company.co.uk",
    "invalid@domain",
    "@missinguser.com"
]

for e in emails:
    print(f"{e}: {is_valid_email(e)}")
```

**Expected output:**

```
user@example.com: True
john.doe@sub.domain.org: True
user-name@company.co.uk: True
invalid@domain: False
@missinguser.com: False
```

---

## Best Practices

| Situation               | What to Do                                   | Why                                                       |
| ----------------------- | -------------------------------------------- | --------------------------------------------------------- |
| Reporting a bug         | Include the error trace and source snippet   | Claude's reasoning improves with the full failure context |
| Generating tests        | Describe expected behavior in plain English  | "The function should raise ValueError if input is empty"  |
| Code review             | Ask Claude to review intent, not just syntax | Catches logic mismatches, not only typos                  |
| Testing fixes           | Feed results back to Claude iteratively      | Enables targeted follow-up without restarting             |
| Trusting suggestions    | Verify in your environment                   | Claude's context is limited to what you provided          |
| Framework-specific code | Specify library versions                     | Prevents Claude from suggesting deprecated APIs           |

---

## Conclusion

Claude Code works best as a reasoning partner in a loop: you frame the problem, Claude proposes a diagnosis and fix, you test it and report back, Claude adjusts. The quality of that loop depends on the quality of your framing — clear descriptions of expected behavior, actual error output, and relevant context get better results than vague bug reports.

You now have the mechanics for each part of that loop: context-aware bug identification, structured code review, iterative fix testing, and the human-in-the-loop phases that keep the process on track.

**Next step:** Take a buggy function from your own codebase. Paste it into Claude with the error output and a clear description of what it should do. Work through the loop — hypothesis, fix, test, refine. The time from problem to working code will surprise you.

> **Acknowledgment:** Based on official Anthropic documentation and community research. Technical details reflect the state of Claude Code as of May 2026.
