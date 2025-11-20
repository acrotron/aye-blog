---
title: "The Day I Stopped Babysitting an AI: Building the Optimistic Workflow"
date: 2025-11-09
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "The story of how frustration with approval-seeking AI assistants led to Aye Chat's 'optimistic workflow' - where the AI acts first and you review after, backed by an instant undo button."
tags: ["workflow", "ux", "design-philosophy", "origin-story"]
---

You know that feeling when you're trying to get work done and someone keeps tapping your shoulder for approval? "Is this okay?" Tap. "How about this?" Tap. "Should I do this next?" Tap tap tap.

That's what using most AI coding assistants felt like to me. I'd ask it to add a feature, and then - instead of just doing it - it would present me with a diff and wait. Like a puppy showing me its toy, waiting for validation. Every. Single. Time.

I was babysitting an AI like it was a toddler with scissors.

And I kept thinking: "These models are getting scary good. Claude 4.5 Sonnet gets things right maybe 70-80% of the time on first try. Why am I still approving every comma it wants to add?"

So I built something different. Something that would just *do the thing* and let me fix it if it screwed up. I called it the **optimistic workflow**, and it became the heart of Aye Chat.

This is the story of how that came together - and why it works.

## The Undo Button on Steroids

Letting an AI write directly to your files sounds insane at first. What if it misunderstands and deletes your entire database layer? What if it gets creative and rewrites your authentication in a way that locks everyone out?

The only way the optimistic approach works - the *only* way - is if you have a perfect undo button. Not Git (too slow, too manual). Not Ctrl+Z (only works in editors). Something instant. Something automatic. Something that happens *before* the AI even thinks about touching your files.

That's what I built first: a snapshot engine that acts like a paranoid librarian.

You know those rare book libraries where they photograph every page before letting you read it? That's the idea. Before the AI writes *anything*, we take a perfect snapshot of what's there. The whole file, exactly as it was, tucked away in `.aye/snapshots/` with a timestamp and the prompt that triggered the change.

Only *then* does the AI get to write.

```python
# aye/model/snapshot.py

def apply_updates(updated_files: List[Dict[str, str]], prompt: Optional[str] = None) -> str:
    """
    1. Take a snapshot of the *current* files.
    2. Write the new contents supplied by the LLM.
    """
    file_paths: List[Path] = [Path(item["file_name"]) for item in updated_files]
    batch_ts = create_snapshot(file_paths, prompt)  # Safety net FIRST
    for item in updated_files:
        fp = Path(item["file_name"])
        fp.parent.mkdir(parents=True, exist_ok=True)
        fp.write_text(item["file_content"], encoding="utf-8")  # Then write
    return batch_ts
```

This backup-first approach is non-negotiable. It's the foundation that makes everything else possible. No snapshot, no write. Period.

And here's the beautiful part: the snapshot isn't just a file copy. It's a time capsule. It remembers *why* the change happened (your prompt is in the metadata), *when* it happened (timestamp), and *exactly* what was there before. It's version control purpose-built for the "try -> fail -> undo" loop of AI coding.

## Two Commands That Changed Everything

A safety net is useless if it's tangled up in a closet somewhere. The whole point of moving fast is... well, moving fast. So reviewing and reverting changes had to be just as instant as making them.

Enter two stupidly simple commands: `diff` and `restore`.

### Seeing What Changed: `diff`

After the AI makes a change, you naturally wonder: "Okay, what exactly did you do?" Instead of opening another tool or switching windows, you just type:

```bash
(ツ» diff calculator.py
```

Boom. Instant, colorized diff right in your terminal. It's just comparing the live file against the snapshot we took two seconds ago. You see the change, you understand it, you move on. No context switch. No friction.

```python
# aye/presenter/diff_presenter.py

def show_diff(file1: Path, file2: Path) -> None:
    """Show diff between two files using system diff command."""
    try:
        result = subprocess.run(
            ["diff", "--color=always", "-u", str(file2), str(file1)],
            # ...
        )
        # ...
    except FileNotFoundError:
        # Fallback to Python's difflib if system diff is not available
        _python_diff_files(file1, file2)
```

It's so simple it's almost boring. But that's the point. Boring infrastructure that just works is what lets you focus on the interesting stuff.

### The Magic Undo: `restore`

And if you don't like what you see? If the AI got creative in the wrong way?

```bash
(ツ» restore calculator.py
```

That's it. The file is back to exactly how it was. The AI's change is gone, like it never happened. No Git commands to remember, no stash juggling, no "wait which commit was that again?"

```python
# aye/model/snapshot.py

def restore_snapshot(ordinal: Optional[str] = None, file_name: Optional[str] = None) -> None:
    # ...
    if ordinal is None and file_name is not None:
        snapshots = list_snapshots(Path(file_name))
        if not snapshots:
            raise ValueError(f"No snapshots found for file '{file_name}'")
        _, snapshot_path_str = snapshots[0]
        # ...
        shutil.copy2(snapshot_path, original_path)
        return
    # ...
```

This one-command rollback is what makes the optimistic workflow *work*. It removes the fear. The cost of a bad AI suggestion drops to near zero - just the three seconds it takes to type `restore`. That's when you realize: you're not babysitting anymore. You're collaborating.

## How It Feels in Practice

Let me show you where this gets real. You're working on a simple calculator module with an `add()` function:

```python
# calculator.py
def add(a, b):
    return a + b
```

You've got tests. They're green. Life is good.

Then you think: "You know what? This should handle any number of arguments, not just two." So you ask:

```bash
(ツ» modify the add function to take a list of numbers instead of two arguments
```

The AI doesn't ask for permission. It just does it:

```
-{•!•}- » I have modified the `add` function to accept a list of numbers.
```

Behind the scenes, the snapshot happened first (safety net deployed), then the file got rewritten. You're curious what changed:

```bash
(ツ» diff calculator.py
--- .aye/snapshots/001_.../calculator.py
+++ calculator.py
@@ -1,2 +1,2 @@
-def add(a, b):
-    return a + b
+def add(numbers):
+    return sum(numbers)
```

Clean. Elegant. The AI understood the assignment. You feel that little dopamine hit - this is going to work.

So you run the tests (without exiting the session mind you):

```bash
(ツ» pytest
```

And then:

```
================================ FAILURES =================================
_______________________________ test_add __________________________________

    def test_add():
>       assert add(2, 3) == 5
E       TypeError: add() takes 1 positional argument but 2 were given

test_calculator.py:4: TypeError
========================= 1 failed in 0.12s ==========================
```

Oh God. OH GOD.

The tests are broken. The function signature changed but the tests still call it the old way. Your brain immediately starts racing: "Okay I need to update the tests, or wait maybe I should revert this, or maybe I should just fix the function to handle both cases, or—"

And then you remember: you have an undo button.

```bash
(ツ» restore calculator.py
```

One second later:

```bash
(ツ» pytest
========================= 1 passed in 0.08s ===========================
```

Green again. Crisis averted. Your heart rate returns to normal.

Total time from "oh no" to "all good": **4 seconds**.

That's the moment it clicks. You're not afraid anymore. You can try things. Wild things. Aggressive refactors. Experimental rewrites. Because the cost of failure isn't hours of Git archaeology or careful manual rollbacks - it's typing seven letters: `restore`.

You can iterate fearlessly. And when you're not afraid to fail, you move *fast*.

That's the workflow. Fast, confident, reversible. No approval dialogs. No copy/paste. No babysitting.

## The Git Upgrade: From Sedan to Sports Car

Our file-copy snapshot system works great. It's simple, it's reliable, it works on any project with zero setup. It's like a dependable sedan - gets you where you need to go without fuss.

But I knew developers had a sports car sitting in the garage: **Git**. And I kept thinking: what if we could give them the option to use it?

So that's what we're building next: a Git-powered snapshot engine that's faster, more powerful, and integrates seamlessly with tools developers already use.

### The Plan: Two Engines, One Interface

We're using the **Strategy pattern** - fancy name for a simple idea. Think of it as building a car with interchangeable engines. Same car, same controls, but you can swap in a different engine depending on what you need.

Two "engines":
1. **`FileCopyStrategy`**: The sedan. Our current file-copy logic. Default for non-Git projects.
2. **`GitStrategy`**: The sports car. Uses native Git commands. Automatically kicks in for Git repos.

This way, simple projects stay simple, but Git users get superpowers.

### How the Git Version Will Work

Instead of copying files, we just use Git operations developers already know:

1. **Snapshot (`apply_updates`)**: `git stash push -m "aye-chat: <your_prompt>"`
   - Instant, space-efficient, automatically documented

2. **View changes (`diff`)**: `git diff stash@{0} -- <file_name>`
   - Compare against the stashed version

3. **Undo (`restore`)**: `git stash pop`
   - One command, back to where you were

But here's where it gets interesting.

### The Real Power: Partial Acceptance

Ever wanted to accept *part* of an AI's suggestion but not all of it? With Git, we can build a `review` command that uses `git checkout -p stash@{0}`. It lets you approve changes hunk-by-hunk, like a code review.

You get to say: "Yes to this function, no to that refactor, yes to the docstring, no to the renaming."

That's not just faster. That's a completely different level of control.

### Using Your Existing Tools

And because the AI's changes are just Git stashes, you can use *any* Git tool to inspect them. VS Code's source control panel. `lazygit`. Whatever you already use. Aye Chat becomes part of your existing workflow instead of replacing it.

This isn't about using a fancier tool for the same job. It's about unlocking a workflow that wasn't possible before - one where you can collaborate with an AI at the speed of thought, with surgical precision when you need it.

## What This All Means

The optimistic workflow isn't just a feature. It's a different way of thinking about AI collaboration.

Instead of:
- Prompt -> Review -> Approve -> Apply -> Test

You get:
- Prompt -> Apply -> (Test/Review if needed) -> (Undo if wrong)

The approval step vanishes. The AI becomes a collaborator you can trust to act, knowing you have a perfect undo button if it goes sideways.

It took me about a week to build the first version - nights and a weekend, obsessively iterating. The snapshot engine took a day. The `diff` and `restore` commands took a few hours. The Git integration is still in progress.

But the feeling of it? That happened immediately. The first time I prompted the AI, watched it write code directly to my file, checked the diff, and moved on - all in under 10 seconds - I knew this was different.

No more babysitting. No more approval fatigue. Just flow.

That's what we're building. And honestly? It's exhilarating.

---
## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings the power of AI directly into your command-line workflow. Edit files, run commands, and chat with your codebase without ever leaving the terminal.

### Support Us 🫶

*   **Star 🌟 our [GitHub repository](https://github.com/acrotron/aye-chat#aye-chat-ai-powered-terminal-workspace).** It helps new users discover Aye Chat.
*   **Spread the word 🗣️.** Share Aye Chat on social media and recommend it to your friends.
---
