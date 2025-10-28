# 🪝 git-anchored-clone

**`git-anchored-clone`** is a Bash utility for cloning a Git repository and attaching a **synthetic root (anchor) commit** to a chosen base commit using **Git replace refs** — without rewriting any history.

It provides a way to create a *virtual root* for shallow or partial clones, making repository history traversable without requiring a full fetch.

---

## 🔧 Features

- Performs **shallow or full clones** of remote or local repositories.
- Supports anchoring at a **specific base commit** or limiting history by **depth**.
- Creates a **synthetic root commit** to act as a virtual starting point.
- Links commits using **replace refs** (non-destructive and reversible).
- Handles both remote and local repository sources.
- Automatically detects and deepens shallow clones as needed.

---

## 📦 Usage

```bash
./git-anchored-clone <remote-url> <target-dir> [[<branch>]@<base-commit>|<depth=N>]
```

### Examples

```bash
# Clone branch 'main' and attach synthetic root at commit 1dd3642
./git-anchored-clone https://github.com/some/project.git myrepo main@1dd3642

# Clone branch 'main' shallowly (depth=1)
./git-anchored-clone https://github.com/some/project.git myrepo main@

# Clone branch 'main' with depth=3
./git-anchored-clone https://github.com/some/project.git myrepo main@depth=3

# Clone default branch shallowly (depth=1)
./git-anchored-clone https://github.com/some/project.git myrepo

# Clone from a local directory
./git-anchored-clone local_dir/some/project myrepo
```

---

## ⚙️ Parameters

| Parameter | Description |
|------------|-------------|
| `<remote-url>` | Remote repository URL or local path. |
| `<target-dir>` | Destination directory for the cloned repository. |
| `<branch>` | (Optional) Branch name to clone. Defaults to the remote's HEAD. |
| `<base-commit>` | (Optional) Commit hash to attach the synthetic root to. |
| `<depth=N>` | (Optional) Limit clone depth to N commits. |

---

## 🧩 Behavior Summary

1. **Clone Phase**
   - Performs a shallow or full clone depending on options.
   - Detects whether the repository is shallow and deepens incrementally if needed.

2. **Base Commit Resolution**
   - Uses a specified base commit (`branch@<hash>`), inferred commit, or depth limit.
   - If necessary, repeatedly deepens history to reach the base commit.

3. **Synthetic Root Creation**
   - Creates an empty tree and commits it as a synthetic “anchor” commit.
   - Records metadata such as origin URL and branch@commit info.

4. **Virtual Grafting**
   - Uses `git replace --graft` to make the base commit a child of the synthetic root.
   - Removes `.git/shallow` to make the graph appear complete.

5. **Result**
   - The history is now virtually rooted at the synthetic anchor.
   - No actual commits or hashes are modified.

---

## 🧠 Example Output

```bash
Cloning branch 'main' (depth: 3) from https://github.com/some/project.git into myrepo...
---
Clone completed.

Creating synthetic root (anchor) commit...
Synthetic root (anchor) commit created: abc1234...
---
tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904
author Anchored Root <root@example.com> ...
committer Anchored Root <root@example.com> ...

Synthetic root (anchor) for shallowed clone
origin: https://github.com/some/project.git
branch: main@1dd3642
---

Virtual history (top 10 commits):
* abc1234 (replace) Synthetic root (anchor) for shallowed clone
* 1dd3642 Base commit
* ...
```

---

## 🧹 Cleanup and Inspection

| Task | Command |
|------|----------|
| List active replace refs | `git replace -l` |
| Remove the virtual link | `git replace -d <base-commit>` |
| View synthetic commit | `git cat-file -p <root-commit>` |
| Show anchored history | `git log --graph --oneline --decorate` |

---

## ⚠️ Notes

- The **synthetic root** exists only locally and is **not part of the remote**.
- Replace refs are **not transferred** when pushing or fetching.
- This technique is ideal for **analysis, testing, or visualization** of partial repositories.
- The process is **non-destructive** — no commits or tags are rewritten.

---

## 📜 License

MIT License © 2025  
Created for developers who need lightweight Git history.
