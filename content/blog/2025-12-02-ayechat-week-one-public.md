---
title: "One Week in Public: What Our First 129 Users Taught Us"
date: 2025-12-02
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "A look at Aye Chat's first week after going public: usage numbers, retention, and why the optimistic workflow seems to 'click' after a few hours."
tags: ["metrics", "retention", "optimistic-workflow", "product"]
---

Shipping a tool you built for yourself into the public is a strangely vulnerable feeling.

For months, Aye Chat lived on my machine and a handful of teammates' laptops. We used it every day, wired it into our habits, and fine-tuned the "optimistic workflow" until it felt natural.

Then we opened the doors.

This post is a look back at the **first week after going public**: who showed up, how long they stayed, what surprised us, and what the numbers seem to say about a workflow that lets an AI touch your files first and asks questions later.

---
## The Numbers: 129 People Walked In, Some Never Left

In the first week:

- **129 people** tried Aye Chat.
- **42 users** stayed for **more than 1 hour**.
- **20 users** stayed for **more than 1 full day of usage**.

Inside those 20 long-term users, we saw this breakdown:

- **1 user** with **6 days+** of usage in the first week
- **3 users** with **5 days+**
- **4 users** with **4 days+**
- **3 users** with **3 days+**
- **3 users** with **2 days+**

On the open-source side:

- **31 GitHub stars**
- **5 forks**

For a brand-new terminal tool with essentially zero marketing, this is a small but very dense signal: people are not just installing it, they are **coming back**.

A different way to see it:

- Most experiments die in the first session.
- Here, **42/129** people crossed the one-hour mark.
- **20/129** kept Aye Chat open and in use across **multiple days**.

For an early-stage developer tool, that long tail of multi-day usage is exactly what you hope to see.

---
## The 2–3 Hour Hump: Learning to Trust the Optimistic Workflow

The most interesting pattern in the logs is not just how many people stayed, but **when** they started behaving like power users.

Aye Chat is built on an idea that still feels a bit alien if you come from traditional AI assistants:

> Let the AI act first. Review and undo after.

Instead of:

- Prompt -> Model shows diff -> You approve -> Tool applies

we do:

- Prompt -> Tool snapshots files -> Model applies changes directly -> You `diff` / `restore` if needed

This **optimistic workflow** is backed by a paranoid safety net (snapshots in `.aye/`, instant `restore`, easy `diff`). On paper, it is safer than a typical manual workflow. In practice, it demands something from the user:

- You have to be willing to **let go of the approval step**.
- You have to **internalize that undo is perfect and cheap**.

From conversations and from usage traces, a pattern is emerging:

- The first **1–2 hours** often look cautious.
  - People run small prompts.
  - They check `diff` a lot.
  - They sometimes mirror changes manually in another editor.
- Somewhere around **hour 2**, behavior flips.
  - Users start issuing bigger, bolder prompts.
  - `restore` becomes muscle memory.
  - The tool turns from "AI demo" into "primary coding surface".

That flip is reflected in the retention curve:

- Many users who drop off **before** the ~2-hour mark never come back.
- Users who **cross** that mark are disproportionately represented in the 2–6 days+ cohort.

In other words: it seems to take **2–3 hours** to really get the hang of the optimistic workflow. But once it lands, people tend to **stay**.

---
## How This Compares to Typical Early-Stage Retention

Early-stage developer tools usually follow a familiar pattern:

1. A spike of curiosity-driven installs.
2. A steep drop after the first session.
3. A small core who adopt it into their real work.

You rarely win or lose on raw install count; you win or lose on **depth**.

By that standard, our first week looks encouraging:

- Out of **129** people who tried Aye Chat, a meaningful slice used it for **multiple hours** and then **multiple days**.
- We are not just seeing one or two outliers living in the terminal; we see a small **cluster** of users who integrated Aye Chat into their daily flow almost immediately.

The sample is tiny and it is far too early to declare victory. But seeing **11 users** with **3+ days of activity** in week one is a strong early sign that:

- The problem is real.
- The workflow resonates with a subset of developers.
- Once that subset crosses the learning curve, churn drops sharply.

---
## Why the Workflow Is Hard to Explain but Easy to Feel

A lot of this comes back to something I wrote in the [optimistic workflow post](/posts/2025-11-09-ayechat-optimistic-workflow/):

> The first time I prompted the AI, watched it write code directly to my file, checked the diff, and moved on – all in under 10 seconds – I knew this was different.

The challenge is that this feeling is hard to convey in a README.

On a landing page, "AI edits your code directly" sounds risky.
From inside Aye Chat, with `diff` and `restore` bound to your fingers, it quickly becomes **liberating**:

- You stop copy/pasting between a browser and the terminal.
- You stop babysitting an AI that keeps asking for permission.
- You start iterating: prompt -> change -> tests -> fix -> refactor, in one continuous loop.

The data from week one suggests that:

- If someone only has 15–30 minutes, they mostly see a novel AI CLI.
- If someone gives it **a couple of hours**, they start to feel that shift from **supervised tool** to **trusted collaborator**.

Our job now is to shorten that path.

---
## What We Are Taking Away from Week One

A few concrete lessons we are acting on:

1. **Optimize the first two hours.**
   - Better onboarding around `diff`, `restore`, and snapshots.
   - Clearer examples that encourage users to let the AI touch real files sooner, but safely.

2. **Lean into depth, not just reach.**
   - We care more about the next 20 people who will use Aye Chat for 5 days than the next 2,000 who will try it for 5 minutes.

3. **Respect that trust is earned locally.**
   - Our best advocates so far are the users who have already lived inside Aye Chat for days. The product has to keep justifying that trust with reliability, privacy, and a UX that stays out of the way.

This was only week one. The numbers will change, and there is no guarantee the curve will stay this friendly. But seeing real developers give a brand-new tool **hours and days of their working time** is the strongest validation we could have hoped for.

We will keep shipping, keep listening, and keep pushing on this strange, exciting idea that the fastest way to write software might be to **stop babysitting the AI**.

---
## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings the power of AI directly into your command-line workflow. Edit files, run commands, and chat with your codebase without ever leaving the terminal.

### Support Us

* Star our GitHub repository: https://github.com/acrotron/aye-chat#aye-chat-ai-powered-terminal-workspace – it helps new users discover Aye Chat.
* Spread the word. Share Aye Chat with your team and friends who live in the terminal.
---
