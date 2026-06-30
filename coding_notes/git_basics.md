# Git Basics

A beginner's guide to the four git commands you'll use almost every day:
`git pull`, `git add`, `git commit`, and `git push`.

---

## What is git, and why do we use it?

**Git is a "save system" for code.** Think of it like the save points in a video game,
but much more powerful:

- It remembers **every version** of your project, so you can always go back to an earlier one.
- It records **who changed what, when, and why** — so you can understand how the code got
  to where it is.
- It lets **multiple people work on the same project** without overwriting each other's work.

Your project lives in two places:

- **Local** — the copy of the project on *your* computer, where you actually edit files.
- **Remote** — a shared copy stored online (usually on GitHub). This is the "official" copy
  everyone shares.

The four commands below are how you move work between your local copy and the remote copy.

A handy way to picture it:

```
   REMOTE (GitHub)
        |  ^
  pull  |  |  push
        v  |
   YOUR COMPUTER  --add-->  staging  --commit-->  saved locally
```

---

## The everyday workflow

A normal session looks like this:

1. **`git pull`** — get the latest changes before you start.
2. Edit your files (write code).
3. **`git add`** — pick which changes you want to save.
4. **`git commit`** — save those changes (with a message).
5. **`git push`** — share your saved changes with everyone.

Let's go through each one.

---

## 1. `git pull` — download the latest changes

```bash
git pull
```

**What it does:** Brings the newest version of the project *from the remote* down to your
computer.

**Why we do it:** Other people (or you, on another computer) may have made changes since you
last worked. Pulling first means you start from the most up-to-date version, which avoids
conflicts later.

> **Rule of thumb:** Always `git pull` *before* you start working for the day.

---

## 2. `git add` — choose what to save

```bash
git add filename.py      # add one specific file
git add .                # add ALL changed files in this folder
```

**What it does:** Marks which changed files you want included in your next save. This waiting
area is called the **staging area**.

**Why we do it:** Sometimes you change several files but only want to save *some* of them
together. `git add` lets you pick. (When you're starting out, `git add .` to grab everything
is perfectly fine.)

> **Tip:** Run `git status` anytime to see what's changed and what's been added. It's the
> most useful command for figuring out "where am I?"

---

## 3. `git commit` — save your changes

```bash
git commit -m "Add the kick drum sound"
```

**What it does:** Permanently records the staged changes into git's history as one "save
point" (called a **commit**). The `-m` stands for **message**.

**Why we do it:** Each commit is a snapshot you can return to later. The message explains
*what you changed and why*, so future-you (and teammates) can understand the history.

**Writing good messages:**
- Say *what* the change does, briefly: `"Fix tempo calculation bug"`
- Not just `"stuff"` or `"asdf"` — those won't help you later.

> A commit only saves changes you've `git add`ed. Anything not added is left out.

---

## 4. `git push` — share your changes

```bash
git push
```

**What it does:** Uploads your local commits *to the remote* (GitHub), so everyone else can
see and use them.

**Why we do it:** Commits live only on your computer until you push. Pushing publishes your
work to the shared copy. If your computer broke right now, anything you pushed would be safe.

> **Rule of thumb:** Push when you've finished a meaningful chunk of work, so it's backed up
> and shared.

---

## Putting it all together

A complete, typical session:

```bash
git pull                              # 1. get the latest
# ... edit your code ...
git status                            #    (optional) see what changed
git add .                             # 2. stage your changes
git commit -m "Add hi-hat pattern"    # 3. save them with a message
git push                              # 4. share them
```

---

## Quick reference

| Command               | What it does                            | When to use it           |
|-----------------------|-----------------------------------------|--------------------------|
| `git pull`            | Download latest changes from the remote | Before you start working |
| `git status`          | Show what's changed                     | Anytime you're unsure    |
| `git add .`           | Stage your changes to be saved          | After editing            |
| `git commit -m "..."` | Save staged changes with a message      | After staging            |
| `git push`            | Upload your commits to the remote       | After committing         |

---
