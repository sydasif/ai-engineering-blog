---
title: "Refactoring and Optimization Workflows: Turning Messy Code into Clean, Fast Systems"
date: 2026-05-30
tags: [refactoring, optimization, python, claude-code, clean-code]
description: "Learn how to use Claude Code to clean up technical debt, implement design patterns, and optimize performance without breaking functionality."
---

# Refactoring and Optimization Workflows: Turning Messy Code into Clean, Fast Systems

## Introduction

You have working code that you dread touching. Variable names make no sense. The same logic appears in three places. It's slow on large inputs, and adding a new feature feels fragile.

Refactoring fixes that — and Claude Code makes it faster. This guide covers how to clean up, restructure, and optimize code without breaking what already works: spotting design patterns in spaghetti logic, improving performance, balancing simplicity against efficiency, and applying all of it to a real-world web scraper.

---

## Principles of Clean Refactoring

Refactoring improves existing code without changing its external behavior. Not rewriting from scratch — making the code cleaner, more readable, and easier to maintain while keeping exactly what it does.

Three rules to hold throughout any refactor:

1. **Maintain functional parity.** Output before and after must be identical for the same input.
2. **Simplify structure.** Replace complex or duplicated logic with simpler, reusable functions.
3. **Increase clarity.** The new version should be more readable and aligned with your coding standards.

### Example: Refactoring an Order Calculator

**Original (works, but messy):**

```python
def calculate_order_total(items, discount_rate, tax_rate):
    total = 0
    for item in items:
        price = item['price']
        quantity = item['quantity']
        total += price * quantity
    discount = total * discount_rate
    total = total - discount
    tax = total * tax_rate
    total = total + tax
    return round(total, 2)
```

Problems: repeated variable reassignment, all logic in one function, no docstrings, hard to unit-test.

**Prompt:**

> "Refactor this order total function for clarity and maintainability. Break it into smaller, reusable functions. Preserve the exact same behavior."

**Claude's refactored version:**

```python
def calculate_order_total(items, discount_rate=0.0, tax_rate=0.0):
    """Calculate the total order price with discount and tax applied."""
    subtotal = _calculate_subtotal(items)
    discounted = _apply_discount(subtotal, discount_rate)
    total = _apply_tax(discounted, tax_rate)
    return round(total, 2)

def _calculate_subtotal(items):
    """Compute subtotal based on item prices and quantities."""
    return sum(item['price'] * item['quantity'] for item in items)

def _apply_discount(subtotal, discount_rate):
    """Apply discount to subtotal."""
    return subtotal * (1 - discount_rate)

def _apply_tax(amount, tax_rate):
    """Apply tax to the discounted amount."""
    return amount * (1 + tax_rate)
```

**Validation test:**

```python
items = [{"price": 10.0, "quantity": 2}, {"price": 5.0, "quantity": 3}]

# Original inline calculation
original = 0
for item in items:
    original += item["price"] * item["quantity"]
original -= original * 0.1
original += original * 0.07
original = round(original, 2)

# Refactored
new_total = calculate_order_total(items, discount_rate=0.1, tax_rate=0.07)
print(original == new_total)  # Expected: True
```

`True` confirms behavior is preserved, structure improved.

| Aspect         | Before               | After                                |
| -------------- | -------------------- | ------------------------------------ |
| Length         | Single long function | Modular functions                    |
| Responsibility | Mixed                | Each function handles one task       |
| Extensibility  | Hard to modify       | Easy to adjust discount or tax rules |
| Testability    | Hard to isolate      | Unit-test friendly                   |

---

## Design Pattern Discovery

Patterns often emerge in code before they're named. Claude can recognize them — Singleton, Factory, Observer — even when they developed unintentionally, and refactor them into explicit, robust implementations.

### Example: Discovering a Singleton Pattern

**Original logger:**

```python
class Logger:
    def __init__(self):
        self.logs = []

    def log(self, message):
        self.logs.append(message)
        print(f"[LOG]: {message}")

logger = Logger()
logger.log("Application started.")
```

This works in a single module. Import it into multiple modules and each gets its own instance, breaking log consistency.

**Prompt:**

> "This logger creates new instances when imported in different modules. Identify the design pattern it resembles and refactor it properly."

**Claude's analysis:** "This should be a singleton — one shared instance across the application. The basic `__new__` override is the common introduction, but it's not thread-safe. In multi-threaded environments, two threads can check `_instance is None` simultaneously and both create an instance."

**Thread-safe Singleton:**

```python
import threading

class Logger:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking: re-check after acquiring the lock
                if cls._instance is None:
                    cls._instance = super(Logger, cls).__new__(cls)
                    cls._instance.logs = []
        return cls._instance

    def log(self, message):
        self.logs.append(message)
        print(f"[LOG]: {message}")
```

**Verification:**

```python
logger1 = Logger()
logger2 = Logger()
logger1.log("Application started.")
logger2.log("Processing data...")
print(f"Same instance? {logger1 is logger2}")  # True
```

**One important caveat Claude should flag:** for async frameworks (FastAPI, aiohttp, asyncio-based servers), `threading.Lock` doesn't work correctly. Use `asyncio.Lock()` with a factory method instead — `__new__` can't be awaited. For most logging scenarios, a module-level singleton (a plain module-level instance) is simpler and avoids the problem entirely.

**Optional: Factory pattern**

```python
class LoggerFactory:
    _logger = None
    _lock = threading.Lock()

    @staticmethod
    def get_logger():
        if LoggerFactory._logger is None:
            with LoggerFactory._lock:
                if LoggerFactory._logger is None:
                    LoggerFactory._logger = Logger()
        return LoggerFactory._logger
```

Use this when you want to separate creation from usage — useful when the logger initialization takes arguments or depends on configuration.

| Pattern   | Symptom in Code                             | Benefit                                         |
| --------- | ------------------------------------------- | ----------------------------------------------- |
| Singleton | Shared-state class instantiated repeatedly  | Centralized control, consistent resource access |
| Factory   | Objects created manually in multiple places | Clean separation of creation and usage          |
| Observer  | Event-driven callbacks without structure    | Decoupled, maintainable event handling          |

---

## 5.3 Performance Optimization

Optimize where you have a measured bottleneck. Claude helps identify inefficiencies early, explains root causes, and proposes targeted improvements.

### Example: Optimizing a Word Counter

**Original (O(n²)):**

```python
def count_unique_words(file_path):
    unique_words = []
    with open(file_path, "r", encoding="utf-8") as f:
        for line in f:
            words = line.strip().split()
            for word in words:
                if word not in unique_words:  # O(n) scan per word
                    unique_words.append(word)
    return len(unique_words)
```

Each membership check scans the entire list. On a large file this compounds quickly.

**Prompt:**

> "This function becomes very slow with large text files. Analyze and optimize it without changing its behavior."

**Claude's reasoning:** "List membership checks are O(n). Replace the list with a set for O(1) average lookups. Also normalize to lowercase for consistent deduplication."

**Optimized (O(n)):**

```python
def count_unique_words(file_path):
    """Count unique words in a text file using a set for O(1) lookups."""
    unique_words = set()
    with open(file_path, "r", encoding="utf-8") as f:
        for line in f:
            for word in line.strip().split():
                unique_words.add(word.lower())
    return len(unique_words)
```

On a 10MB file the difference is minutes vs. milliseconds.

| Focus          | Problem                                 | Recommendation              | Impact              |
| -------------- | --------------------------------------- | --------------------------- | ------------------- |
| Data structure | List for membership checks              | Replace with set            | O(n) → O(1) average |
| Algorithm      | Nested iteration                        | Simplify loop logic         | Better scalability  |
| Memory         | Unbounded list growth                   | Hash-based collection       | Reduced storage     |
| I/O            | Synchronous DB calls in async framework | Async or connection pooling | Faster throughput   |

---

## 5.4 Simplification vs. Efficiency

Not every optimization is worth its complexity. Sometimes the simpler code is the better choice.

### Example: Filtering Even Numbers

**Simple, readable version:**

```python
def filter_even_numbers(numbers):
    """Return all even numbers from the list."""
    return [n for n in numbers if n % 2 == 0]
```

**Prompt:** "Can this be optimized for performance if it needs to handle millions of integers, or should I keep it simple?"

**Claude's reasoning:** "List comprehensions run in C and are already fast. For most cases this is optimal. If your dataset exceeds available memory, a generator reduces memory overhead by yielding one element at a time."

**Memory-efficient generator:**

```python
def filter_even_numbers_stream(numbers):
    """Yield even numbers lazily for large datasets."""
    for n in numbers:
        if n % 2 == 0:
            yield n
```

**Trade-off:**

```python
import time
nums = list(range(10_000_000))

start = time.time()
filter_even_numbers(nums)
print("List comprehension:", round(time.time() - start, 3), "s")

start = time.time()
list(filter_even_numbers_stream(nums))
print("Generator:", round(time.time() - start, 3), "s")
```

| Approach           | Performance               | Memory                | Best Use                  |
| ------------------ | ------------------------- | --------------------- | ------------------------- |
| List comprehension | Fast for small-medium     | Stores all results    | General purpose           |
| Generator          | Slightly slower iteration | Streams one at a time | Large datasets, pipelines |
| NumPy vectorized   | Very fast for numerics    | Moderate              | Numerical heavy lifting   |

Simplify where clarity yields lasting value. Optimize only where you have a measured bottleneck.

---

## 5.5 Example Project: Optimizing a Web Scraper

A complete example: take a slow synchronous scraper and turn it into a concurrent, fault-tolerant pipeline.

### Baseline (Slow but Correct)

```python
import time
import requests
from bs4 import BeautifulSoup

HEADERS = {"User-Agent": "MasteringClaudeCode/1.0"}

def fetch(url, timeout=10):
    resp = requests.get(url, headers=HEADERS, timeout=timeout)
    resp.raise_for_status()
    return resp.text

def parse(html):
    soup = BeautifulSoup(html, "html.parser")
    title = soup.title.string.strip() if soup.title else ""
    h1 = soup.find("h1").get_text(strip=True) if soup.find("h1") else ""
    return {"title": title, "h1": h1}

def scrape(urls):
    results = []
    for url in urls:
        html = fetch(url)
        data = parse(html)
        data["url"] = url
        results.append(data)
        time.sleep(0.2)  # polite delay
    return results
```

Problems: synchronous (waits for each request), no retries, no concurrency, fragile error handling.

### Optimized (Concurrent, Robust)

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import random

HEADERS = {"User-Agent": "MasteringClaudeCode/1.0"}
MAX_CONCURRENCY = 10
REQUEST_TIMEOUT = aiohttp.ClientTimeout(total=12, connect=5)

async def fetch(session, url, attempt=1, max_attempts=3):
    try:
        async with session.get(url) as resp:
            resp.raise_for_status()  # raises ClientResponseError for 4xx/5xx
            return await resp.text()
    except (aiohttp.ClientError, asyncio.TimeoutError):
        if attempt >= max_attempts:
            raise
        backoff = (2 ** (attempt - 1)) + random.uniform(0, 0.25)
        await asyncio.sleep(backoff)
        return await fetch(session, url, attempt + 1, max_attempts)

def parse_fast(html):
    soup = BeautifulSoup(html, "lxml")  # faster C-based parser
    title = soup.title.string.strip() if soup.title else ""
    h1 = soup.find("h1").get_text(strip=True) if soup.find("h1") else ""
    return {"title": title, "h1": h1}

async def scrape_one(semaphore, session, url):
    async with semaphore:
        try:
            html = await fetch(session, url)
            data = parse_fast(html)
            data["url"] = url
            return url, data, None
        except Exception as e:
            return url, None, str(e)

async def scrape(urls):
    semaphore = asyncio.Semaphore(MAX_CONCURRENCY)
    connector = aiohttp.TCPConnector(limit=MAX_CONCURRENCY)
    async with aiohttp.ClientSession(
        headers=HEADERS,
        timeout=REQUEST_TIMEOUT,
        connector=connector
    ) as session:
        tasks = [scrape_one(semaphore, session, url) for url in urls]
        results = []
        for coro in asyncio.as_completed(tasks):
            url, data, err = await coro
            results.append({"url": url, "error": err} if err else data)
        return results
```

Key changes Claude explains:

- **Concurrency.** Async I/O overlaps requests, hiding network latency. Wall time drops from `N * latency` to roughly `latency`.
- **Connection pooling.** `TCPConnector` reuses connections, reducing handshake overhead.
- **Timeouts and exponential backoff.** Fails fast on slow endpoints and recovers from transient errors.
- **Faster parser.** `lxml` is a C-based HTML parser — significantly faster than `html.parser` for large pages.
- **Structured error handling.** Partial failures are captured per-URL rather than crashing the pipeline.

> **Note on `raise_for_status()`:** The optimized version calls `resp.raise_for_status()` to handle 4xx/5xx responses. This is the correct pattern — `aiohttp.ClientResponseError` requires internal `request_info` and `history` arguments and should not be constructed manually.

**Run it:**

```python
URLS = ["https://example.com", "https://httpbin.org/html"]
items = asyncio.run(scrape(URLS))
```

| Optimization       | What Changed                                 | Why It Matters                                |
| ------------------ | -------------------------------------------- | --------------------------------------------- |
| Concurrency        | Single-threaded → asyncio + semaphore        | Overlaps I/O, wall time ≈ one request latency |
| Connection reuse   | New TCP per request → pooled                 | Reduces handshake overhead                    |
| Timeouts + backoff | None → explicit + exponential                | Faster failure, automatic recovery            |
| Parser             | `html.parser` → `lxml`                       | C-based, measurably faster on large HTML      |
| Error handling     | Exceptions crash pipeline → captured per-URL | Pipeline survives partial failures            |

---

## 5.6 Cost Considerations

Token costs are real for teams running Claude at scale. A few practices keep them under control.

**Current API pricing (May 2026, per million tokens):**

| Model      | Input | Output | Best For                                 |
| ---------- | ----- | ------ | ---------------------------------------- |
| Haiku 4.5  | $1.00 | $5.00  | Simple refactors, repetitive tasks       |
| Sonnet 4.6 | $3.00 | $15.00 | General coding — best cost/quality ratio |
| Opus 4.7   | $5.00 | $25.00 | Complex architecture, deep analysis      |

Prompt caching cuts input costs by up to 90%. The Batch API cuts everything 50%. Combined, costs can drop by up to 95% for high-volume workloads.

**A rough cost estimator:**

```python
def estimate_cost(prompt: str, response_tokens: int, model: str = "sonnet") -> float:
    """Estimate API cost for a single request (USD)."""
    pricing = {
        "haiku":  {"in": 1.00, "out": 5.00},
        "sonnet": {"in": 3.00, "out": 15.00},
        "opus":   {"in": 5.00, "out": 25.00},
    }
    # ~4 chars per token
    in_tokens = max(1, len(prompt) // 4)
    p = pricing[model]
    return round(
        (in_tokens / 1_000_000) * p["in"] + (response_tokens / 1_000_000) * p["out"],
        6
    )

prompt = "Refactor this 200-line Python class for readability."
print(f"Estimated cost: ${estimate_cost(prompt, response_tokens=500, model='sonnet')}")
```

**Cost-control habits:**

| Practice                           | Why It Helps                                                     |
| ---------------------------------- | ---------------------------------------------------------------- |
| Match model to task complexity     | Haiku for simple refactors; Opus only for deep architecture work |
| Keep a 2-4 sentence project anchor | Reduces repeated context tokens across turns                     |
| Ask for only what you need         | "Return only the refactored function" cuts unnecessary output    |
| Batch similar requests             | Amortizes system tokens; 50% off with the Batch API              |
| Trim context to relevant files     | Large irrelevant context dilutes focus and burns tokens          |

---

## Conclusion

Refactoring and optimization are small, intentional improvements that compound over time. With Claude Code you can decompose tangled functions into clean modules, name the patterns that already exist in your code, replace O(n²) lookups with O(1) data structures, and scale a synchronous scraper into a concurrent pipeline — without breaking what already works.

**Try this:** Pick a messy function from your own codebase. Prompt Claude: "Refactor this for clarity and maintainability. Preserve all behavior. Break it into smaller, reusable pieces." Run the validation test. Then profile before and after on real data if performance matters.

> **Acknowledgment:** Based on official Anthropic documentation and community research. Technical details reflect the state of Claude Code as of May 2026.
