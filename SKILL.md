---
name: pickup
description: Read a HANDOFF.md and restore session context — or infer context from git history when no handoff exists. The entry ritual — the opposite of /handoff. Use at the start of a new session after a handoff, machine switch, or context clear.
---

# /pickup — Session Pickup

> Consume a HANDOFF.md and orient the session so work continues exactly where it left off.

When invoked, perform the following steps in order. Do not ask for confirmation — just execute.

---

## Step 0 (optional): Remote Project Routing

Some users keep projects on remote machines and SSH into them. If you want `/pickup <project>` to transparently route to a remote host, drop a config file at `~/.claude/pickup-routes.json`:

```json
{
  "routes": [
    {
      "prefix": "work",
      "host": "myserver",
      "ssh_alias": "ssh myserver",
      "project_root": "~/projects",
      "remote_home": "/home/me"
    }
  ]
}
```

When the first word of the argument matches a `prefix`, the skill runs all subsequent searches (`find`, `git`, `cat`) via that SSH alias. Paths are reported with the host prefix (e.g. `myserver:~/projects/foo/`) so it's clear the project is remote.

If the file doesn't exist, skip this step entirely. Local search (Step 1) is the default flow.

---

## Step 1: Locate HANDOFF.md

> If Step 0 matched a configured remote prefix, run these searches on the remote machine via SSH instead of locally.

**If the argument is a file path** (starts with `/`, `./`, or `~/`):
- Use that exact path (locally or via SSH if the path is on a remote machine).

**If the argument is a project name** (e.g., `MyProject`, `my-app`):
- Search `~/projects/` recursively (2 levels deep) for a directory matching the name (case-insensitive).
  ```bash
  find ~/projects -maxdepth 3 -type d -iname "*<argument>*" 2>/dev/null
  ```
- For each match, check if `HANDOFF.md` exists in that directory.
- Also check `~/HANDOFF.md` — read its first 5 lines and check if the Project path contains the argument.
- If exactly one HANDOFF.md found — use it.
- If multiple found — list them with dates, ask user which one.
- If none found in project dirs but `~/HANDOFF.md` matches — use that.

**If no argument**:
1. Check current working directory for `HANDOFF.md`
2. Check `~/HANDOFF.md`
3. Search `~/projects/` subdirectories (2 levels deep) for recent `HANDOFF.md` files
4. If multiple found — list them with dates, ask user which one

If no HANDOFF.md is found but a **project directory was located** (local or remote):
- Proceed to **Step 1B: Cold Pickup** (infer context from the codebase).

If no HANDOFF.md is found AND no project directory matches:
- Report clearly: "No HANDOFF.md or project found. Provide a path or project name: `/pickup <project>`"
- Stop.

---

## Step 1B: Cold Pickup (No Handoff — Infer from Codebase)

When a project directory exists but has no HANDOFF.md, reconstruct context from git history and filesystem signals.

### Step 1B.1: Multi-Repo Discovery

The search term may match multiple related projects. They form an ecosystem worth surfacing.

**Scan all matched directories** from Step 1's `find` results. For each one, get a one-line summary:

```bash
# For each matched directory:
# If it's a git repo:
git -C [dir] log --oneline --format="%h %ad %s" --date=relative -1 2>/dev/null

# If NOT a git repo, use filesystem mtime on key files:
ls -lt [dir]/*.js [dir]/*.ts [dir]/*.py [dir]/*.vue 2>/dev/null | head -1
```

Build a **sibling activity table** sorted by most recent activity:

```
| Project        | Last Activity | Type        | Signal                         |
|----------------|---------------|-------------|--------------------------------|
| project-alpha  | 2 hours ago   | filesystem  | server.js modified             |
| project-beta   | 22 hours ago  | git commit  | feat(auth): add OAuth flow...  |
| project-gamma  | 5 days ago    | filesystem  | server.js modified             |
```

**Primary project**: The one that best matches the argument (exact match preferred, then most recently active).
**Sibling projects**: All others — shown in the output so the user sees the full ecosystem.

### Step 1B.2: Scan Primary Project

For the primary project, run git commands (if it's a git repo) or filesystem commands (if not):

**If git repo:**
```bash
# 1. Recent commits with dates
git -C [project path] log --oneline --format="%h %ad %s" --date=relative -15

# 2. Current branch and tracking
git -C [project path] branch --show-current
git -C [project path] branch -vv

# 3. Working tree status
git -C [project path] status --short

# 4. Uncommitted or staged changes
git -C [project path] diff --stat
git -C [project path] diff --cached --stat

# 5. Remote sync status
git -C [project path] log --oneline @{upstream}..HEAD 2>/dev/null || echo "No upstream tracking"
git -C [project path] reflog show --format="%h %gd %gs" --date=relative | grep -m3 "push\|pull\|fetch" 2>/dev/null

# 6. Recently modified tracked files (last 7 days)
git -C [project path] diff --name-only HEAD@{7.days.ago}..HEAD 2>/dev/null
```

**If NOT a git repo:**
```bash
# Recently modified files (filesystem mtime, top 15, excluding common build artifacts)
find [project path] -maxdepth 3 \( -name node_modules -o -name .nuxt -o -name .git -o -name .output -o -name dist \) -prune -o \( -name "*.ts" -o -name "*.js" -o -name "*.vue" -o -name "*.py" -o -name "*.md" -o -name "*.json" -o -name "*.css" -o -name "*.html" \) -print | xargs ls -lt 2>/dev/null | head -15

# Directory listing for structure
ls -la [project path]/
```

### Analysis

From those signals, determine:

1. **Last work cluster**: Group the last 5-10 commits by theme. Identify the most recent coherent body of work. Use commit message prefixes (`feat`, `fix`, `refactor`, `docs`) to classify.
2. **Active area**: Which directories/files were touched most recently? This is the likely focus area.
3. **Dirty state**: Any uncommitted changes, untracked files, or recently modified files? These are the strongest signal — they're literally where work was interrupted.
4. **Cross-repo activity**: Note if sibling projects were modified more recently than the primary. This often means the user was actually working on a sibling.
5. **Predicted last task**: Synthesize a 1-2 sentence prediction of what was last being worked on.

### Read Project CLAUDE.md

If the project has a `CLAUDE.md`, read it for project context (stack, architecture, conventions).

### Then proceed to Step 4 (Present Orientation)

Use the adapted format below instead of the handoff-based format:

```markdown
## Picked Up: [Project Name] (cold — no handoff)

**Path**: [absolute path]
**Branch**: [branch] | **Last commit**: [hash — message]
**Git status**: [clean / N files changed / N uncommitted]
**Remote**: [ahead N / in sync / behind N / no upstream]

### Ecosystem Activity
[Sibling activity table — all matched projects sorted by last activity]

### Last Work (inferred)
[1-2 sentence prediction of the most recent work focus across the ecosystem]

### Commit Log (last 15) — [primary project name]
[Show the full `git log` output verbatim in a code block]

### Push / Pull Status
- **Last push**: [date/time from reflog, or "unknown"]
- **Unpushed commits**: [count and list, or "none — in sync with remote"]

### Work Clusters
[Group the last 10-15 commits into 2-4 themed clusters]

### Active Files
[Top 3-5 files by recent modification — the likely entry points]

### Dirty State
[Uncommitted changes, untracked files, or "Clean — no pending work"]

### Ready
Context inferred from git history and filesystem. No handoff file was found.
What do you want to work on?
```

**Do NOT enter Plan Mode** for cold pickups — there's no specific task to plan. Just orient and ask.

---

## Step 2: Read and Parse

Read the HANDOFF.md fully. Extract:

- **Project path** from the Project section
- **Branch** and **last commit**
- **What we were doing** (the narrative summary)
- **State**: Done, In Progress, Blocked
- **Next steps** (the ordered list)
- **Key files** (the table)
- **Resume Prompt** (at the bottom)

---

## Step 3: Load Key Context

For each file in the **Key Files** table, read it if it exists and is under 200 lines.
Skip files over 200 lines (too large) — note them for the user instead.

If the project path has a git repo:
```
git -C [project path] status --short
git -C [project path] log --oneline -5
```

---

## Step 4: Present Orientation

Output a concise orientation block — not a copy of HANDOFF.md, a **distillation**:

```markdown
## Resumed: [Project Name]

**Path**: [absolute path]
**Branch**: [branch] | **Last commit**: [hash — message]
**Git status**: [clean / N files changed]

### What Was Happening
[1-2 sentence synthesis of the task and approach]

### State at Handoff
- Done: [key completed items, comma-separated]
- In progress: [exactly where it was left]
- Blocked: [if any]

### Next Action
[The single most immediate thing to do — specific file, command, or decision]

### Ready
Context loaded. Key files read. Git oriented.
To continue: [paste the resume prompt action or just say what to do]
```

---

## Step 4.5: Enter Plan Mode

After presenting the orientation, automatically enter Plan Mode using the `EnterPlanMode` tool.

This puts the session into planning state so the user can review the handoff context and decide on approach before any code changes happen.

---

## Step 5: Archive HANDOFF.md

After successful load, rename `HANDOFF.md` to `HANDOFF.archived.md` in the same directory.

This signals: "this handoff was consumed." It preserves history without cluttering future `/pickup` searches.

Tell the user: `HANDOFF.md archived as HANDOFF.archived.md`

---

## Notes

- **The goal is speed** — from zero context to executing in under 30 seconds.
- **Do not re-read the full history** — trust what HANDOFF.md captured.
- **Do not ask clarifying questions** unless HANDOFF.md is ambiguous about what to do next.
- **If multiple HANDOFF.md exist** — ask which one before loading anything.
- **If HANDOFF.md is malformed** — note what's missing and load what you can.
