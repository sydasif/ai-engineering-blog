# 🖋️ AI Engineering Blog

Welcome to my technical blog focused on AI engineering, agentic workflows, and the evolving landscape of LLM-assisted development.

## 📚 Table of Contents

### 🚀 Getting Started

- [Getting Started with Claude Code: Your First AI Coding Partner](content/posts/2026-05-28-getting-started-with-claude-code.md) — _A comprehensive guide to installing and collaborating with the Claude Code CLI._

### 🧠 Deep Dives

- [How Claude Code Thinks: Inside Your AI Coding Assistant](content/posts/2026-05-28-how-claude-code-thinks.md) — _An exploration of context windows, tokens, and the reasoning engine._

### ✍️ Mastering Prompting

- [Prompting Foundations for Developers](content/posts/2026-05-28-prompting-foundations-for-developers.md) — _How to move from generic outputs to production-ready code through structured prompting._

---

## 🛠️ Repository Structure

This repository is organized for scalability and compatibility with Static Site Generators (SSGs) like Hugo, Jekyll, or Astro.

```text
.
├── assets/
│   ├── css/        # Custom styles
│   └── images/     # Blog post imagery and diagrams
├── content/
│   └── posts/      # Markdown blog posts with YAML frontmatter
├── README.md       # Repository index and guide
├── .gitignore      # Version control exclusions
└── LICENSE         # Open source licensing
```

## ✍️ Contributing & Adding New Posts

To add a new post to this blog:

1. Create a new `.md` file in `content/posts/` using the format `YYYY-MM-DD-slug.md`.
2. Include the required YAML frontmatter at the top:
   ```yaml
   ---
   title: "Your Post Title"
   date: YYYY-MM-DD
   tags: [tag1, tag2]
   description: "A brief summary for SEO and previews."
   ---
   ```
3. Update the **Table of Contents** in the `README.md` to include your new post.
4. Add any necessary images to `assets/images/`.

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.
