---
title: "The Optimistic Workflow: How Aye Chat Reimagines AI-Assisted Development"
date: 2025-11-09
draft: false
author: "Vyacheslav Mayorskiy, dev team"
summary: "A deep dive into Aye Chat's 'optimistic workflow,' where AI changes are applied instantly and made safe by a robust, automatic snapshot and restore system."
tags: ["workflow", "ux", "design-philosophy"]
---

Traditional AI coding assistants follow a cautious, multi-step process: you prompt, the AI suggests, you copy, you paste, and you review. This workflow, while safe, is riddled with friction. It treats the AI as an untrusted outsider whose work must be manually chaperoned into the codebase.

Aye Chat is built on a different philosophy: **confident collaboration**. We believe that to achieve true flow, the developer and the AI must operate as a single, integrated unit. This led us to design an **optimistic workflow**, where the AI's suggestions are applied to your files *immediately*. The review step happens *after* the change is made, not before. This might sound risky, but it's made completely safe by an automatic snapshotting system that underpins the entire experience.

This post is a deep dive into the mechanics of our optimistic workflow, exploring the snapshot engine and the simple commands that give you the confidence to let the AI take the wheel.

## Part 1: The Undo Button on Steroids: The Snapshot Engine

Letting an AI write directly to your files sounds scary. What if it misunderstands and wipes out a critical function? The only way this optimistic approach works is if you have a perfect, instant undo button. That's the job of our snapshot engine, implemented in [aye/model/snapshot.py](https://github.com/acrotron/aye-chat/blob/main/src/aye/model/snapshot.py).

### How It Works

Think of it like a meticulous librarian. Before letting anyone write in a rare book, the librarian first takes a perfect photograph of the page. That's what our snapshot engine does for your code.

Whenever the LLM suggests a file change, we perform a quick, two-step dance before writing anything to disk:

1.  **Take a Snapshot**: First, we copy the *current* state of the affected files into a timestamped folder inside `.aye/snapshots/`. We also tuck a `metadata.json` file in there, which includes the prompt you used, so you always know *why* a change was made.
2.  **Write the New Content**: Only after the backup is safely stored do we write the new file content from the AI into your working directory.

```python
# aye/controller/llm_handler.py

def process_llm_response(...):
    # ...
    if not updated_files:
        # ...
    else:
        try:
            # Apply updates to the model (Model update)
            apply_updates(updated_files, prompt)
            # ...
        except Exception as e:
            rprint(f"[red]Error applying updates:[/] {e}")
```

```python
# aye/model/snapshot.py

def apply_updates(updated_files: List[Dict[str, str]], prompt: Optional[str] = None) -> str:
    """
    1. Take a snapshot of the *current* files.
    2. Write the new contents supplied by the LLM.
    """
    file_paths: List[Path] = [Path(item["file_name"]) for item in updated_files]
    batch_ts = create_snapshot(file_paths, prompt)
    for item in updated_files:
        fp = Path(item["file_name"])
        fp.parent.mkdir(parents=True, exist_ok=True)
        fp.write_text(item["file_content"], encoding="utf-8")
    return batch_ts
```

This backup-first approach is critical. It ensures that no change is ever made without a safety net. The snapshot isn't just a file copy; it's a time capsule of your code's state *at the moment before the AI acted*, linked directly to the prompt that triggered the change.

## Part 2: The Frictionless Safety Net: `diff` and `restore`

A safety net is useless if it's tangled and hard to deploy. The whole point of the optimistic workflow is to move fast, so reviewing and reverting changes had to be just as fast. This is where two simple commands, `diff` and `restore`, become your best friends.

### Viewing Changes with `diff`

After the AI makes a change, you'll naturally wonder, "What exactly did it do?" Instead of switching to a Git client or another tool, you just type `diff <file_name>` right in the chat.

```bash
(ツ» diff tutorial_example.py
```

Behind the scenes, we're just running a standard `diff` command between the live file and the snapshot we just took. You get a familiar, colorized output right in your terminal. This instant feedback loop keeps you in the flow, without ever leaving the conversation.

```python
# aye/presenter/diff_presenter.py

def show_diff(file1: Path, file2: Path) -> None:
    """Show diff between two files using system diff command or Python fallback."""
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

### Reverting Changes with `restore`

What if you don't like the change? No problem. This is where the real magic happens. Just type `restore <file_name>`.

This command reaches into the time capsule, pulls out the original version of the file, and puts it right back where it was. The AI's change is gone, as if it never happened.

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

This one-command rollback is the cornerstone of the optimistic workflow. It removes the fear of letting an AI write directly to your files. The cost of a bad suggestion is reduced to near zero—just the few seconds it takes to type `restore`.

## Part 3: The Workflow in Action

Let's walk through how these pieces come together in a typical session. Imagine you're working on a file and you want to add a docstring.

1.  **The Prompt**: You ask the AI to modify the file.
    ```bash
    (ツ» add a docstring to the hello_world function
    ```

2.  **The Action**: Aye Chat sends the prompt and file context to the LLM. The LLM responds with new file content. The `apply_updates` function is called.
    -   `create_snapshot` saves the original `tutorial_example.py` to `.aye/snapshots/001_.../`.
    -   The new content with the docstring is written to `tutorial_example.py`.

3.  **The Feedback**: The assistant confirms the change.
    ```
    -{•!•}- » I have added a docstring to the `hello_world` function as you requested.
    ```

4.  **The Review**: You inspect the change directly in the terminal.
    ```bash
    (ツ» diff tutorial_example.py
    --- .aye/snapshots/001_.../tutorial_example.py
    +++ tutorial_example.py
    @@ -1,2 +1,3 @@
     def hello_world():
    +    """Prints 'Hello, World!' to the console."""
         print("Hello, World!")
    ```

5.  **The Decision**: You decide you don't want the change.
    ```bash
    (ツ» restore tutorial_example.py
    ```

6.  **The Reversal**: The snapshot is instantly restored, and the file is back to its original state. You're ready for your next prompt, having lost no time or momentum.

## Part 4: Future Work: A Git-Powered Safety Net

Our file-copy system is like a reliable sedan: it gets the job done simply and without fuss. It works on any project, right out of the box, with no dependencies. But we know that for professional developers, the garage often has a high-performance sports car: **Git**. Our next step is to give our snapshot engine a Git-powered upgrade.

### Refactoring with the Strategy Pattern

To do this without breaking the simple "sedan" version, we're using a classic software design trick: the **Strategy pattern**. Think of it as building a car with an interchangeable engine. We'll define a common interface for snapshotting, then create two "engines" that fit it:

1.  **`FileCopyStrategy`**: This is the sedan engine. It will contain our current logic of copying files and will be the default for any project that isn't a Git repository.
2.  **`GitStrategy`**: This is the sports car engine. It will be automatically used for Git projects and will translate our snapshot commands into native Git operations.

This design lets us offer a more advanced workflow where it's available, without sacrificing the plug-and-play simplicity of the original.

### How the `GitStrategy` Will Work

By mapping our commands to Git operations, we gain enormous power and efficiency.

1.  **Creating a Snapshot (`apply_updates`)**: Instead of copying files, we'll just run `git stash push -m "aye-chat: <user_prompt>"`. It's what developers already do to save work-in-progress. It's incredibly fast, space-efficient, and the stash message automatically documents the change.

2.  **Viewing Changes (`diff`)**: The `diff` command will simply map to `git diff stash@{0} -- <file_name>`, comparing your file directly against the stashed version.

3.  **Reverting Changes (`restore`)**: And `restore`? That's just `git stash pop`. One command, and you're back to exactly where you were.

### Unlocking a More Powerful Workflow

This isn't just about using a fancier tool for the same job. Switching to Git unlocks a whole new level of power and control, turning our simple safety net into a full-blown trapeze act.

*   **Efficiency**: Git's delta compression is built for this, making snapshots of even huge files trivial.
*   **Robustness**: A `git stash pop` can often be recovered, making `restore` far less destructive than a simple file overwrite.
*   **Fine-Grained Control**: Ever wanted to accept only *part* of an AI's suggestion? With Git, we can. We can build a `review` command that uses `git checkout -p stash@{0}` to let you approve changes hunk-by-hunk, just like a code review.
*   **Interoperability**: Since the AI's changes are just Git stashes, you can use any of your favorite Git tools (VS Code's source control panel, `lazygit`, etc.) to inspect them. This provides a seamless bridge between Aye Chat and your existing tools.

By integrating deeply with Git, we transform the snapshot engine from a simple backup tool into a professional-grade versioning system tailored for AI collaboration. It's the next logical step in making the optimistic workflow not just fast, but also powerful and flexible.

## Conclusion

The optimistic workflow is more than just a feature; it's a fundamental shift in the human-AI interaction model. By reversing the traditional "review then apply" sequence and backing it with a robust, automatic snapshot system, Aye Chat removes friction and fosters a sense of trust and partnership with the AI.

This allows developers to stay in a state of flow, iterating on ideas at the speed of thought. It transforms the AI from a mere suggestion box into a true collaborator that can be trusted to act on its own, knowing that any step can be instantly undone. This is the future of AI-assisted development—confident, collaborative, and incredibly fast.

---
## About Aye Chat

**Aye Chat** is an open-source, AI-powered terminal workspace that brings the power of AI directly into your command-line workflow. Edit files, run commands, and chat with your codebase without ever leaving the terminal.

### Support Us 🫶

*   **Star 🌟 our [GitHub repository](https://github.com/acrotron/aye-chat#aye-chat-ai-powered-terminal-workspace).** It helps new users discover Aye Chat.
*   **Spread the word 🗣️.** Share Aye Chat on social media and recommend it to your friends.
---
