# ormus-pickup

> Restore session context at the start of a new Claude Code session. The entry ritual — the inverse of /handoff.

A Claude Code skill that reads a `HANDOFF.md` from the previous session, loads the relevant files, and orients Claude to continue work exactly where it left off. When no handoff exists, it falls back to inferring context from git history and filesystem signals.

## Why

Handing off context is one half of the loop. Picking it back up is the other. `/handoff` writes a structured session-state file. `/pickup` consumes that file and bootstraps the next session in 30 seconds — no re-explaining, no re-exploring.

For projects that never got a handoff written (you forgot, the session ended abruptly, or it's a new project), `/pickup` does a "cold pickup": scans recent commits, identifies the most recently active work cluster, and infers what you were probably doing. Often enough to continue without missing a beat.

## Install

```bash
git clone https://github.com/HermeticOrmus/ormus-pickup ~/.claude/skills/pickup
```

Or as a Claude Code plugin:

```bash
claude plugin marketplace add HermeticOrmus/ormus-pickup
```

Restart Claude Code so the skill registry picks it up.

## Usage

```
/pickup                    — find the most recent HANDOFF.md and load it
/pickup MyProject          — find HANDOFF.md in a specific project (or cold pickup)
/pickup ~/path/HANDOFF.md  — load a specific handoff file
```

The skill executes in 5 steps:

1. **Locate** `HANDOFF.md` (by argument, project name, or cwd fallback)
2. **Read & parse** the handoff structure
3. **Load key context** — read files listed in the handoff (under 200 lines)
4. **Present orientation** — distilled summary of state, next action
5. **Archive** the consumed handoff as `HANDOFF.archived.md`

For projects without a handoff, **Step 1B** kicks in: scan git history, build a sibling activity table, infer the last work cluster, and present a "cold pickup" orientation instead.

## Optional: Remote routing

If you keep projects on remote SSH-accessible machines, drop a `~/.claude/pickup-routes.json`:

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

Then `/pickup work my-project` runs all searches via SSH on `myserver`. Paths get reported with a `myserver:~/projects/my-project/` prefix so you know it's remote.

If the config doesn't exist, the skill skips this step entirely. It's purely opt-in.

## Pairs with

- **[ormus-handoff](https://github.com/HermeticOrmus/ormus-handoff)** — the entry's other half. Run /handoff at session end, /pickup at session start.
- **[ormus-absorb](https://github.com/HermeticOrmus/ormus-absorb)** — distill conversation knowledge into persistent memory so /pickup loads it automatically.
- **[ormus-explore](https://github.com/HermeticOrmus/ormus-explore)** — token-efficient AST-based code search, useful during cold pickup when you need to read 6+ files.
- **[ormus-meta-prompting](https://github.com/HermeticOrmus/ormus-meta-prompting)** — categorical foundations for AI prompt engineering.

Together they form the **ormus session lifecycle** — composable Claude Code skills for serious cross-day, cross-machine work.

## License

MIT. See [LICENSE](LICENSE).

## Origin

Distilled from real cross-day Claude Code engagements where re-establishing context cost ten minutes of re-explaining at the start of every session. The cold-pickup path emerged after enough sessions ended without a handoff that the skill needed a fallback. The optional remote routing emerged from patterns where the same project lives on multiple machines and "pickup" needs to know which one to look at.
