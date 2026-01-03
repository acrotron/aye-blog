---
title: "The Day I Realized Terminal UX for AI Is Really About DX"
date: 2025-12-16
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "Models get the headlines. DX is the hard part: making an AI assistant feel native in the terminal without breaking muscle memory. Here are the surprisingly classic UX principles behind Aye Chat."
tags: ["ux", "dx", "terminal", "design-philosophy", "origin-story"]
---

Most AI devtools love to talk about models.

And sure, models matter.

But the thing that actually decides whether a developer keeps using your tool is way less glamorous:

> Does it feel *native* in the terminal?

Because the terminal is not just a UI. It’s a mindset.

When a tool breaks that mindset, you feel it immediately. You lose flow. You hesitate. You start thinking about the tool instead of the work.

When we built Aye Chat, I expected the hardest problems to be “AI problems.”

Turns out a lot of the hardest problems were “1990s UX problems,” just wearing modern clothes.

Here’s the playbook we ended up following.

---
## 1) Hierarchy first: route intent the way a shell does

The terminal has one sacred rule:

- What you type should be interpreted consistently.

Ambiguity is the enemy.

So we made Aye Chat do something simple and strict: **treat input with a clear priority order**.

1. Built-in commands first (like `help`, `restore`, `model`, etc.)
2. If it looks like a shell command, run the real command directly
3. Everything else becomes an AI prompt

That’s it.

It’s basically a hierarchical decision tree (a fancy way of saying: predictable routing).

The DX win is bigger than it sounds:

- You don’t have to “enter AI mode.”
- You don’t leave the terminal mindset.
- You type like you always type.

---
## 2) Progressive disclosure: keep the happy path clean

Terminals are dense by default.

That’s not a bug; that’s why we like them.

So the last thing we wanted was an AI assistant that constantly waved its arms:

- “Here are 17 modes!”
- “Here is a dashboard!”
- “Here is a menu!”

No.

Aye Chat tries to be boring on the happy path:

- Type a prompt.
- Get an answer.

Then, only when you need power, it’s there:

- `help` exists, but it’s not shoved into every interaction.
- `verbose on|off` lets you choose how much operational detail you want.
- RAG context selection is normally invisible; in verbose mode you can see which files were included.
- On big projects, indexing runs in the background; you only see progress if you opt into verbosity (and even then it’s a small inline hint).

This is classic progressive disclosure: minimal surface area up front, depth when you ask for it.

---
## 3) The “optimistic” part: speed with an explicit escape hatch

Here’s the uncomfortable truth about AI coding tools:

- If they’re slow, people stop using them.
- If they’re fast but unsafe, people stop trusting them.

You can’t ship “fast but scary” and hope users will just be brave.

So we leaned hard into an optimistic workflow:

- Apply changes directly.
- Make rollback instantaneous.

That only works if undo is not a suggestion. It has to be real.

In Aye Chat that becomes:

- Every update creates snapshots automatically.
- `diff` is the review step.
- `restore` / `undo` is the safety net.

The DX shift is subtle but huge:

- The mental model changes from “AI is risky” to “AI is reversible.”
- Developers move faster because the cost of being wrong drops.

This is the same reason Ctrl+Z makes creative software usable. If undo is cheap, you explore.

---
## 4) Reduce cognitive load with focus tools (not more UI)

When prompts get vague, AI gets vague.

The typical response is to add more UI:

- more knobs
- more toggles
- more settings

We went the other way.

We added two small primitives that let you focus the model without leaving the terminal:

- `@file` references: include a file inline, on demand
- `with <files>: <prompt>`: constrain the prompt to specific files (supports wildcards)

This is progressive disclosure again:

- You don’t need these on day one.
- But when you hit the “why is it misunderstanding me?” moment, you get a precise tool to say: *this is what matters*.

It’s not UI. It’s a focusing mechanism.

---
## 5) Terminal interaction design: be honest about latency

AI latency is real.

Pretending it isn’t real doesn’t make the experience smoother. It makes it feel broken.

So we decided to acknowledge waiting explicitly, with “progressive waiting” messages that evolve over time:

- “Building prompt…”
- “Sending to LLM…”
- “Still waiting…”

It’s a small thing, but it changes the vibe.

Instead of:

- “Did it freeze?”

you get:

- “It’s working, and it’s telling me what stage it’s in.”

That’s not just UX polish. That’s trust.

---
## 6) Autocomplete as a DX feature (not a gimmick)

Terminal tools win when they keep you in flow.

Autocomplete is one of those quiet features that determines whether something feels native or alien.

So Aye Chat’s completion behavior is designed like a terminal tool, not like a web app bolted onto a CLI:

- Multi-column completions for readability
- Special-cased auto-complete for `@file` contexts
- A setting to choose “readline-like” vs “complete while typing”

The principle is simple:

> The UI adapts to the terminal, not the other way around.

---
## Takeaway

If you’re building AI for developers, UX principles like:

- hierarchy
- progressive disclosure
- reversible actions

aren’t “nice to have.” They’re the difference between:

- a demo you try once
- and a daily driver you don’t want to leave

Models get better every month.

DX is the part you have to earn.

---
## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings the power of AI directly into your command-line workflow. Edit files, run commands, and chat with your codebase without ever leaving the terminal.

### Support Us

- Star our GitHub repository: https://github.com/acrotron/aye-chat#aye-chat-ai-powered-terminal-workspace
- Spread the word. Share Aye Chat with your team and friends who live in the terminal.
---
