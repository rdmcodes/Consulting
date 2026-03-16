# Git Collaboration Guide
**For: rdmcodes/Consulting — Two-Person Team**

---

## The Golden Rule

> **Always pull before you push. Never work on the same file at the same time.**

---

## Daily Workflow (Do This Every Time)

### 1. Start your session — pull first
```bash
git pull origin main
```
This syncs your local copy with whatever your partner pushed. Do this before touching any files.

### 2. Do your work
Edit files, add content, build things.

### 3. Stage and commit your changes
```bash
git add .
git commit -m "short description of what you did"
```
**Good commit messages:**
- `add Block and Drum audit report`
- `update lead-scout skill template`
- `fix typo in CONSULTING-PLAYBOOK`

**Bad commit messages:**
- `stuff`
- `changes`
- `asdfgh`

### 4. Pull again before pushing (in case partner pushed while you worked)
```bash
git pull origin main
```

### 5. Push your work
```bash
git push origin main
```

---

## Dividing Work to Avoid Conflicts

The easiest way to avoid problems is to **not edit the same file at the same time**.

### Split by folder or file ownership:
| Person | Owns |
|--------|------|
| Dorien | `businesses/` folder, audit reports, site designs |
| Partner | `skills/` folder, playbooks, templates |

Communicate in Slack/text before editing shared files like `CONSULTING-PLAYBOOK.md` or `README.md`.

---

## What To Do When You Get a Merge Conflict

A conflict happens when you both edited the same part of the same file. Git will mark it like this inside the file:

```
<<<<<<< HEAD
Your version of the line
=======
Your partner's version of the line
>>>>>>> origin/main
```

**Fix it:**
1. Open the file
2. Decide which version to keep (or combine both)
3. Delete the `<<<<<<<`, `=======`, and `>>>>>>>` markers
4. Save the file
5. Run:
```bash
git add .
git commit -m "resolve merge conflict in [filename]"
git push origin main
```

---

## Cheat Sheet — Commands You'll Use Every Day

| Command | What it does |
|---------|-------------|
| `git pull origin main` | Download your partner's latest changes |
| `git status` | See what files you've changed |
| `git add .` | Stage all your changes |
| `git add filename.md` | Stage one specific file |
| `git commit -m "message"` | Save your changes with a description |
| `git push origin main` | Upload your changes to GitHub |
| `git log --oneline -10` | See the last 10 commits |
| `git diff` | See exactly what you changed before committing |

---

## Rules to Keep It Clean

1. **Pull → Work → Pull → Push** — always in this order
2. **Commit often** — small commits are easier to undo than big ones
3. **Don't leave work half-done at end of day** — commit before closing your laptop
4. **Talk before editing shared files** — a quick "I'm editing the playbook" message prevents conflicts
5. **Never force push** — don't use `git push --force`, it overwrites your partner's work

---

## Quick Recovery Commands

**Undo your last commit (keeps your file changes):**
```bash
git reset --soft HEAD~1
```

**Discard all your uncommitted changes (can't undo this):**
```bash
git checkout -- .
```

**See what changed in a specific commit:**
```bash
git show [commit-hash]
```

---

## Recommended Setup for Your Repo

Consider protecting `main` on GitHub:
1. Go to `https://github.com/rdmcodes/Consulting/settings/branches`
2. Add a branch protection rule for `main`
3. Enable **"Require a pull request before merging"** — this lets you review each other's work before it lands on main (optional but good habit as you grow)

---

*Keep this file in the repo so both teammates can reference it.*
