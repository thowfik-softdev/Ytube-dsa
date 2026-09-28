---
type: foundation
title: Git & GitHub — The Mental Model, the Core Commands, and Fixing Mistakes
tags: [foundations, git, github, version-control, branching, merge, rebase, workflow]
created: 2026-09-29
status: seen
source: "Kunal Kushwaha · Community Classroom — lecture 01 Git & GitHub"
related: ["[[Introduction to Programming]]"]
---

# 28 · Git & GitHub

> 📁 Part 28 of 28 in [Ytube dsa/](README.md) · **Prev:** [27 — Large Numbers & File Handling](27-LargeNumbers_and_FileIO_Notes.md)

## Introduction
The bootcamp opens with Git, and it belongs at the end here as reference — you've been using it throughout.

**Git** is a distributed version-control system: it records snapshots of your project so you can inspect history, work on parallel versions, and undo almost anything. **GitHub** is a hosting service for Git repositories, plus collaboration tools on top.

They are not the same thing, and the distinction is a standard interview question.

## The Mental Model

Everything makes sense once you see the **four areas**:

```
Working Directory  --git add-->  Staging Area  --git commit-->  Local Repo  --git push-->  Remote
      (edits)                      (chosen)                     (history)                  (GitHub)
```

| Area | What it holds |
|---|---|
| **Working directory** | your files as they are right now |
| **Staging area (index)** | changes you've chosen for the next commit |
| **Local repository** | committed history, on your machine |
| **Remote** | the shared copy (GitHub) |

The **staging area** is Git's distinctive idea: you choose *which* changes go into a commit, rather than committing everything at once. It's what lets you make one logical commit out of a messy working directory.

---

## 1. Setup and Starting

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

git init                       # start tracking an existing folder
git clone <url>                # copy an existing repo (and its full history)
```

## 2. The Everyday Loop

```bash
git status                     # what's changed, what's staged  <- use constantly
git add file.java              # stage one file
git add .                      # stage everything
git commit -m "message"        # record the staged changes
git log --oneline              # see history compactly
git diff                       # unstaged changes
git diff --staged              # staged changes
```

> 💡 **`git status` is the single most useful command.** It tells you where you are, what's staged, and usually suggests the command you want next. Run it before and after everything while learning.

**Commit messages matter.** Use the imperative mood ("Add binary search notes", not "Added" or "Adding"), a short summary line, then a blank line and detail if needed. Commits are read far more often than they're written.

## 3. Branching

A **branch** is a movable pointer to a commit. Creating one is instant and free — it copies nothing.

```bash
git branch                     # list branches
git switch -c feature-x        # create and switch (modern)
git checkout -b feature-x      # the older equivalent
git switch main                # move back
git merge feature-x            # bring feature-x's work into the current branch
git branch -d feature-x        # delete once merged
```

**Why branch:** keep unfinished work off `main`, work on several things at once, and let a team work in parallel without collisions.

**Merge conflicts** happen when two branches change the same lines. Git marks them:
```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> feature-x
```
Edit the file to the version you want, delete the markers, then `git add` and `git commit`. Conflicts are normal, not a failure.

## 4. Working With a Remote

```bash
git remote -v                  # which remotes exist
git remote add origin <url>    # connect a local repo to GitHub
git push -u origin main        # first push, and remember the tracking
git push                       # subsequent pushes
git pull                       # fetch + merge others' work
git fetch                      # download without merging — inspect first
```

`origin` is just the conventional name for the default remote — nothing special about it.

> 💡 **`fetch` then `log` before `pull`** when you're unsure what's changed upstream. `pull` is `fetch` + `merge` in one step, which is convenient right up until it isn't.

## 5. Undoing Things

This is the part worth knowing *before* you need it.

| Situation | Command |
|---|---|
| Unstage a file (keep the edits) | `git restore --staged file` |
| Discard working-directory edits | `git restore file` ⚠️ destroys them |
| Fix the last commit's message | `git commit --amend` |
| Undo the last commit, **keep** changes staged | `git reset --soft HEAD~1` |
| Undo the last commit, keep changes unstaged | `git reset HEAD~1` |
| Undo the last commit, **discard** changes | `git reset --hard HEAD~1` ⚠️ |
| Undo a *pushed* commit safely | `git revert <hash>` — makes a new, opposite commit |
| Find a "lost" commit | `git reflog` — your safety net |

> ⚠️ **`reset` vs `revert`:** `reset` rewrites history — fine for local commits, dangerous once pushed, because everyone else's history diverges. `revert` creates a *new* commit that undoes an old one, leaving history intact. **Use `revert` for anything already shared.**

> 💡 **`git reflog` is the undo of last resort.** It records everywhere `HEAD` has been, including commits you "lost" to a bad reset — those commits usually still exist and can be recovered with `git reset --hard <hash>` from the reflog. Knowing this turns most Git disasters into inconveniences.

## 6. `.gitignore`

Files Git should never track:

```gitignore
# Java build artifacts
*.class
out/
target/

# IDE
.idea/
*.iml

# OS
.DS_Store
```

Rules of thumb: **never commit** build output, dependencies (`node_modules`), IDE settings, or — critically — **secrets**. A committed API key stays in history even after you delete the file, and must be treated as compromised.

> ⚠️ `.gitignore` only affects **untracked** files. If something is already tracked, adding it to `.gitignore` changes nothing — you must `git rm --cached <file>` first. This catches everyone once.

## 7. GitHub Collaboration

**Pull request (PR) workflow:**

1. **Fork** (someone else's project) or branch (your own)
2. Make your changes on a branch
3. Push the branch to GitHub
4. Open a **pull request** — proposing your changes for review
5. Discuss, revise, then **merge**

**Issues** track bugs and features. **Actions** run CI on push (tests, builds). **README.md** is what visitors see first — it's the front page of the project.

## 8. Why It Matters Here

For this vault specifically:

- **Every note and solution is committed**, so you can see what you understood and when
- **History is the record of consistency** — more honest than a streak counter
- **Branches** let you experiment on a problem without wrecking a working solution
- **GitHub is a portfolio** — a recruiter can see real work, not just claims

> 💡 The habit worth building: **commit small and often, with real messages.** "Day 14: binary search ceiling/floor + 3 problems" is worth infinitely more in six months than "update".

---

## ⚠️ Common Misunderstandings
**1. Git and GitHub are the same thing.**
❌ same · ✅ Git is the version-control tool (local, works offline); GitHub is a hosting service for Git repos. Interviewers ask this.

**2. `git add` saves your work.**
❌ saves · ✅ it **stages** it. Nothing is recorded until `git commit`.

**3. `git commit` publishes it.**
❌ publishes · ✅ commits are **local**. `git push` shares them.

**4. Branches copy the project.**
❌ copy · ✅ a branch is a **pointer to a commit**. Creating one is instant and costs nothing.

**5. Merge conflicts mean something went wrong.**
❌ wrong · ✅ they're normal when two branches touch the same lines. Git can't guess; you decide.

**6. `reset --hard` and `revert` are interchangeable.**
❌ interchangeable · ✅ `reset` **rewrites** history (dangerous once pushed); `revert` adds a new opposite commit (safe for shared branches).

**7. A bad reset loses work forever.**
❌ forever · ✅ `git reflog` usually still has the commit. It's the safety net almost nobody knows about.

**8. Adding a file to `.gitignore` untracks it.**
❌ untracks · ✅ `.gitignore` only applies to **untracked** files. Already-tracked files need `git rm --cached`.

**9. Deleting a committed secret removes it.**
❌ removes · ✅ it stays in history. Rotate the credential — treat it as compromised.

## Interview Angles
- **"Git vs GitHub?"** — Tool vs hosting service. A warm-up question with a surprisingly high failure rate.
- **"What's the staging area for?"** — Choosing *which* changes form the next commit, so one messy working directory can produce clean, logical commits.
- **"merge vs rebase?"** — Merge preserves history and adds a merge commit; rebase replays your commits on top for a linear history but **rewrites** them, so never rebase shared branches.
- **"How do you undo a pushed commit?"** — `git revert`, never `reset`, because others have the old history.
- **"Describe your branching workflow."** — Feature branches off `main`, PR for review, merge, delete. Mention CI if you've used it.
- **"You committed a secret. Now what?"** — Rotate the credential immediately; removing the file doesn't remove it from history.

## Related · Next
- **Related:** [[Introduction to Programming]] (01) · `RULES.md` and `CONTEXT.md` in this vault, which describe the daily commit habit
- **Practice:** create a branch, make two commits, merge it back, then deliberately create a conflict and resolve it. Then `reset --hard` a commit and recover it with `reflog` — doing that once removes the fear permanently.
- **Next:** You've reached the end of the package. Back to [README](README.md), or straight into the roadmap proper.

---

## 🔁 Rapid Revision (self-test)
<details><summary>1. Git vs GitHub?</summary>

Git is the distributed version-control tool running locally; GitHub is a hosting service for Git repositories with collaboration features layered on.
</details>

<details><summary>2. The four areas, and what moves between them?</summary>

Working directory → (`add`) → staging area → (`commit`) → local repo → (`push`) → remote.
</details>

<details><summary>3. What is the staging area for?</summary>

Choosing which changes go into the next commit, so a messy working directory can still produce clean, logical commits.
</details>

<details><summary>4. What is a branch, really?</summary>

A movable **pointer to a commit**. Creating one copies nothing and is instant.
</details>

<details><summary>5. `reset` vs `revert` — and which for a pushed commit?</summary>

`reset` rewrites history (safe only locally); `revert` adds a new commit that undoes an old one. For anything already pushed, use **`revert`**.
</details>

<details><summary>6. You did a bad `reset --hard`. What now?</summary>

`git reflog` — it records everywhere HEAD has been, so the "lost" commit is usually still recoverable.
</details>

<details><summary>7. You added a tracked file to `.gitignore` and it's still tracked. Why?</summary>

`.gitignore` only affects **untracked** files. Run `git rm --cached <file>` to stop tracking it.
</details>

<details><summary>8. You committed and pushed an API key. What's the correct response?</summary>

**Rotate the credential** — it remains in history even if you delete the file, so treat it as compromised.
</details>

<details><summary>9. merge vs rebase?</summary>

Merge preserves history with a merge commit. Rebase replays commits for a linear history but **rewrites** them — never on shared branches.
</details>
