# 🔁 Git Workflow: Rebase a PR Branch After Another PR Has Been Merged

When your teammate creates two pull requests (e.g., PR#1 and PR#2) targeting the `main` branch:

- You approve and merge PR#1 into `main`
- Before approving PR#2, you must **update PR#2’s branch with the latest code from `main`** (which now includes PR#1)

Follow this step-by-step guide to keep PR#2 up to date.

---

### 1. 🛰️ Fetch the Latest From Remote  
*(To get the latest state of `main` and feature branches)*

```bash
git fetch origin
```

---

### 2. 🧪 Create a Local Branch From the Remote PR Branch *(if not already checked out)*

```bash
git checkout -b feature/branch1 origin/feature/branch1
```

**(or use the cleaner `switch` command)**

```bash
git switch -c feature/branch1 origin/feature/branch1
```

This:
- Creates a new local branch named `feature/branch1`
- Sets it to track the remote `origin/feature/branch1`

---

### 3. 🔄 Rebase the PR Branch Onto the Latest `main`  
*(This makes sure PR#2 is cleanly stacked on top of the latest `main` — including PR#1)*

```bash
git rebase origin/main
```

This replays your local branch’s commits on top of the updated `main` branch.

---

### 4. 🔧 Resolve Conflicts (If Any)

If there are any conflicts during the rebase:

```bash
# Fix the conflicted files manually
git add <file>
git rebase --continue
```

Repeat until all conflicts are resolved and rebase is complete.

---

### 5. 📤 Push the Rebasing Changes Back to Remote

Since rebasing rewrites commit history, you’ll need to force push — **safely**:

```bash
git push --force-with-lease
```

> ⚠️ Use `--force-with-lease` instead of `--force`.  
> It safely pushes your changes only if the remote hasn’t changed unexpectedly since your last fetch.

---

### ✅ Final Outcome

- PR#2 is now **rebased cleanly on top of the latest `main`**
- GitHub/GitLab UI will show:
  - No merge conflicts
  - A clean and correct commit history
- CI/CD will run on the **most up-to-date code**
- Your PR is safe and ready for final review/merge

---

### 🧠 Bonus: How to Check If You're Up to Date

To verify that your branch includes all changes from `main`:

**Commits you are ahead of `main`:**

```bash
git log origin/main..HEAD --oneline
```

**Commits you are behind `main`:**

```bash
git log HEAD..origin/main --oneline
```

If both show no output, your branch is perfectly in sync with `main`.

---

### 🗂 Summary Table

| Step | Action | Description |
|------|--------|-------------|
| 1 | `git fetch origin` | Fetch the latest remote state |
| 2 | `git checkout -b feature/branch1 origin/feature/branch1` | Create local branch from remote |
| 3 | `git rebase origin/main` | Rebase on latest `main` |
| 4 | Resolve conflicts, `git add`, `git rebase --continue` | Finish rebase |
| 5 | `git push --force-with-lease` | Push safely after rebase |
