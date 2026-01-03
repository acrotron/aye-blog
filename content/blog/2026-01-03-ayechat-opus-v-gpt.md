---
title: "When a Model Gets Stuck: How GPT‑5.2 Finished a 'Simple' Spinner That Opus 4.5 Couldn't"
date: 2026-01-03
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "A tiny terminal UX request turned into a concurrency bug hunt. Claude Opus 4.5 got close but kept looping on edge cases; GPT‑5.2 finally nailed the real fix by treating it as a UI state machine and serializing rendering."
tags: ["terminal", "ux", "dx", "streaming", "concurrency", "debugging", "models"]
---

I have a weakness for feature requests that start with:

> "This should be simple."

Because they *are* simple.

Right up until you try to make them correct.

This one came from a very real pain point in Aye Chat:

> "Sometimes the LLM response stalls mid‑sentence. Show a basic spinner when that happens, and remove it when more text arrives."

In a web UI, a stall is annoying.
In a terminal, a stall looks exactly like a crash.

So yes - I needed a subtle `⋯ waiting for more` indicator.
Not a blinking disco. Not a permanent footer. Just an honest signal: *we're alive, we're waiting*.

And this is where the story gets interesting, because it wasn't just a concurrency story.

It was also a **model story**.

Claude Opus 4.5 got us most of the way there, then got stuck in a loop of "reasonable fixes" that didn't quite land.
GPT‑5.2 came in and finished the job - not by being more verbose or more confident, but by being more precise about what the system actually was: a little real‑time UI state machine with multiple writers.

This is the write-up I wish I had before I lost an afternoon to a spinner.

---

## The scene: simulated streaming meets real-world stalls

Aye Chat streams responses into a terminal UI using Rich (`Live`).
We also have simulated streaming: even if a provider returns larger chunks, we animate output word-by-word so it feels like "typing."

Real providers behave like this:

- you receive **some** tokens,
- then there's a gap (LLM is gathering some thoughts, server-side pause, backpressure),
- then streaming resumes.

If that gap happens mid-sentence, users freeze with it.

So the request had four deceptively clean requirements:

1. Detect "stall" while streaming.
2. Show `⋯ waiting for more` only during stalls.
3. Remove it immediately when new text arrives.
4. Don't break Markdown or final formatting.

---

## The architecture (and why it can lie to you)

At a high level the streaming UI has three moving parts:

- `update(content: str)` - called when *new streamed content arrives* (full accumulated content, not a delta).
- `_animate_words(new_text: str)` - prints newly received text word-by-word with a small delay.
- a background monitor thread - periodically decides whether we are "stalled."

Rendering is via a helper like:

```py
_create_response_panel(content, use_markdown=True, show_stall_indicator=False) -> Panel
```

When `show_stall_indicator=True`, it appends:

```
⋯ waiting for more
```

So far, boring.

But then you hit the question that decides everything:

> What does "stalled" *mean*?

And it turns out there are two kinds of "stalled":

- **Network stall:** no new content is arriving.
- **User-visible stall:** nothing is changing on screen.

Those are not the same in a system that intentionally delays rendering.

---

## Where Opus 4.5 got stuck: fixing symptoms instead of the machine

Claude Opus 4.5 did the first part quickly:

- add a timestamp,
- monitor elapsed time,
- show the indicator if we exceed a threshold.

It "worked"… until it didn't.

The bug appeared very specific:

> The indicator blinks briefly even when words are still printing.

That symptom is a big clue. It means the stall detector is looking at **time since last network update**, while the UI is still busy animating buffered words.

Opus tried the next obvious move: add an `_is_animating` flag and suppress the indicator while animating.

Still not fixed.

At that point, you can feel a model fall into a common trap: it keeps proposing plausible tweaks (threshold changes, different checks, "maybe we should…"), but it doesn't fully re-frame the problem.

Because the real problem wasn't just "the flag is wrong."

The real problem was that we had **two concurrent writers** to the same UI.

- the animation path calls `Live.update()` as it prints words
- the monitor thread calls `Live.update()` as it toggles the indicator

Without serialization, you *will* eventually render an inconsistent intermediate frame - which, to the user, looks like blinking.

So the "didn't fix it" moment wasn't Opus being incompetent.

It was Opus being stuck in a local optimum:

- treat it as timing
- treat it as one boolean
- treat it as "add one more guard"

When what we needed was: **state + synchronization**.

That's the moment I switched models.

---

## What GPT‑5.2 did differently: treat it like a state machine with a single renderer

GPT‑5.2 didn't win by being clever.
It won by being strict.

It made three changes that turned the spinner from "mostly works" into "boring and correct."

### 1) Serialize shared state and all UI updates

First: a lock.

```py
self._lock = threading.RLock()
```

And then a rule:

> If it touches shared state or calls `Live.update()`, it must hold the lock.

We centralized rendering into a single helper so we stopped sprinkling `Live.update()` in random code paths:

```py
def _refresh_display(self, use_markdown: bool = False, show_stall: bool = False) -> None:
    with self._lock:
        if not self._live:
            return

        self._live.update(
            _create_response_panel(
                self._animated_content,
                use_markdown=use_markdown,
                show_stall_indicator=show_stall,
            )
        )
        self._showing_stall_indicator = show_stall
```

This one change kills a whole class of "blinking because two threads fought for the frame buffer."

### 2) Redefine "stall" as "caught up and no new input"

This was the conceptual pivot.

A stall should only be possible when:

- we are **not currently animating**, and
- the animated output has **caught up** to what we have received.

In code:

```py
caught_up = (not self._is_animating) and (self._animated_content == self._current_content)
```

That single definition fixes the original "indicator shows while words are still printing" bug.

If the UI is still draining buffered words, you're not stalled.
You're busy.

### 3) Use "last receive time," not "last render time"

After the above, we hit a second, subtler bug:

> When streaming is actually paused, the indicator blinks instead of staying lit.

This is a classic mistake in real-time UI code: if you update the "progress timestamp" when you redraw the indicator, the indicator becomes self-canceling.

So GPT‑5.2 separated the concepts:

- `_last_receive_time` changes **only when new stream content arrives**
- redraws do not touch it

```py
self._last_receive_time: float = 0.0
```

Updated only in `update()` when content truly changes:

```py
with self._lock:
    if content == self._current_content:
        return
    self._last_receive_time = time.time()
```

And the monitor checks:

```py
time_since_receive = time.time() - self._last_receive_time
should_show_stall = time_since_receive >= self._stall_threshold
```

That makes the indicator "sticky" in the correct way:

- it turns on after the threshold,
- it stays on continuously,
- and it turns off immediately when new text arrives.

---

## The final monitor loop (the boring version that works)

Here's what the working logic boils down to:

```py
def _monitor_stall(self) -> None:
    while not self._stop_monitoring.is_set():
        if self._stop_monitoring.wait(0.5):
            break

        with self._lock:
            if not self._started or not self._animated_content:
                continue

            caught_up = (not self._is_animating) and (self._animated_content == self._current_content)
            if not caught_up:
                continue

            time_since_receive = time.time() - self._last_receive_time
            should_show_stall = time_since_receive >= self._stall_threshold

            if should_show_stall != self._showing_stall_indicator:
                self._live.update(
                    _create_response_panel(
                        self._animated_content,
                        use_markdown=False,
                        show_stall_indicator=should_show_stall,
                    )
                )
                self._showing_stall_indicator = should_show_stall
```

Key properties:

- no indicator while buffered words are still animating
- indicator appears only after **no new content** arrives for `stall_threshold`
- indicator stays on continuously once shown
- indicator disappears immediately when new text arrives

The spinner stops being a feature.
It becomes infrastructure.

Which is exactly what terminal UX should be.

---

## The real theme: why swapping models is a debugging tool

I'm not interested in "model wars."

But I *am* interested in the practical reality of building with them:

- Some models are great at first-pass implementation.
- Some models are great at refactoring.
- Some models are great at pushing through annoying edge cases.

In this case:

- **Opus 4.5** got to a plausible implementation quickly, and even cleaned up structure when asked.
  But it kept circling around incremental fixes.
- **GPT‑5.2** zoomed out, saw "two writers + ambiguous definition of stall," and forced the solution into a small state machine with serialized rendering.

That doesn't mean GPT is "better" in the abstract.

It means something more useful:

> When you feel a model looping, change the shape of the conversation - or change the model.

In Aye Chat, switching models is cheap.
And when you're stuck on a UI race condition that only reproduces one out of ten times, "cheap" matters.

---

## Takeaways (and the part that matches Aye Chat's philosophy)

1. **A spinner has a bigger correctness surface area than it deserves.**
   Animation + monitoring + concurrent rendering is a real system.

2. **"Stall" is a state, not a timeout.**
   It must mean "caught up and no new input," not "some time passed."

3. **Don't let rendering update the clock that decides whether to render.**
   That's how you invent blinking.

4. **Locking isn't optional when multiple threads can render.**
   Even if nothing crashes, the UX will.

5. **Model choice is part of the toolchain.**
   When one model gets stuck in local fixes, another might see the global shape.

In a weird way, this tiny `⋯ waiting for more` indicator is the same lesson as the optimistic workflow:

- let the system move fast,
- but build it so you can recover instantly,
- and be pragmatic about the tools (including the model) that get you unstuck.

---

## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings AI directly into command-line workflows. Edit files, run commands, and chat with your codebase without leaving the terminal.

### Support Us

- Star our GitHub repository: https://github.com/acrotron/aye-chat
- Spread the word. Share Aye Chat with your team and friends who live in the terminal.
---
