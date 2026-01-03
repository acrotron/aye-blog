---
title: "Designing Terminal UX for AI is really about DX (and a few classic UX principles)"
date: 2025-12-16
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "Terminal AI tools live or die by developer experience: hierarchy, progressive disclosure, reversible actions, focus primitives, honest latency, and autocomplete that respects muscle memory."
tags: ["ux", "dx", "terminal", "design-philosophy"]
---

A lot of AI devtools talk about models. The harder problem is DX ("Developer Experience"): how do you make an AI assistant feel *native* inside the terminal, without breaking the muscle memory developers already have?

When we built Aye Chat, the UX playbook looked surprisingly “traditional”:

## 1) Hierarchy first: route user intent the way a shell does

In a terminal, ambiguity is the enemy. So Aye Chat treats input with a clear priority:

- Built-in commands (`help` / `restore` / `model` / etc.)
- Shell commands (run the real command directly)
- Everything else becomes an AI prompt

This is a classic hierarchical decision tree: predictable, fast, and easy to learn. The key DX benefit is you don’t “leave the terminal mindset” to use AI.

## 2) Progressive disclosure: power features only appear when you need them

The terminal is already dense; AI shouldn’t add UI clutter.

Aye Chat keeps the default experience minimal (type a prompt, get an answer), then progressively reveals capabilities:

- `help` exists, but it’s not forced into every interaction.
- `verbose on|off` toggles how much operational detail you see.
- RAG context selection is usually invisible; in verbose mode you can see which files were included.
- For large projects, indexing runs in the background; you only see progress if you opt into verbosity (and it shows as a small inline hint in the prompt).

This keeps the “happy path” clean while still supporting the power-user path.

## 3) “Optimistic UX” with an explicit escape hatch

AI edits are only useful if they’re fast, but speed without safety kills trust.

Aye Chat leans into an optimistic workflow: apply changes directly, but make rollback instantaneous.

- Every update creates snapshots automatically.
- `diff` is the review step.
- `restore` / `undo` is the safety net.

DX-wise, this shifts the mental model from “AI is risky” to “AI is reversible.” Developers move faster because the cost of being wrong is low.

## 4) Reduce cognitive load with “focus tools” instead of more UI

Two small primitives do a lot of work:

- `@file` references: include a file inline, on demand.
- `with <files>: <prompt>`: constrain the problem to specific files (supports wildcards).

This is progressive disclosure again: you don’t need to learn these on day one, but when prompts get vague, you have a precise way to tell the system what matters.

## 5) Interaction design for the terminal: make latency feel manageable

AI latency is real; good terminal UX acknowledges it instead of hiding it.

Aye Chat uses “progressive waiting” messages (spinner text that changes over time):

- “Building prompt…”
- “Sending to LLM…”
- “Still waiting…”

It’s a small thing, but it makes the experience feel responsive and honest.

## 6) Autocomplete as a DX feature (not a gimmick)

Terminal tools win when they help you stay in flow. Aye Chat’s completion behavior is designed for that:

- Multi-column completions for readability.
- Special-cased auto-complete for `@file` contexts.
- A user setting to choose “readline-like” vs “complete while typing”.

The principle here is consistency: the UI adapts to the terminal, not the other way around.

## Takeaway

If you’re building AI for developers, UX principles like hierarchy, progressive disclosure, and reversible actions aren’t “nice to have.” They’re the difference between a demo and a daily driver.

---
## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings the power of AI directly into your command-line workflow. Edit files, run commands, and chat with your codebase without ever leaving the terminal.

### Support Us

- Star our GitHub repository: https://github.com/acrotron/aye-chat#aye-chat-ai-powered-terminal-workspace
- Spread the word. Share Aye Chat with your team and friends who live in the terminal.
---
