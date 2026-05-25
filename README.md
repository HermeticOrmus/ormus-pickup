> **Note (2026-05-23) — Consolidated into LibreSessionFlow**
>
> This standalone has been folded into the umbrella repo
> [LibreSessionFlow-Claude-Code](https://github.com/HermeticOrmus/LibreSessionFlow-Claude-Code),
> alongside 9 sibling session-lifecycle plugins. The current version of this
> functionality lives at `plugins/pickup/` in the umbrella.
>
> This repo remains live as a landing page. New work happens in the umbrella.

---

<p align="center">
  <img src="https://ormus.solutions/mascot/pixellab_liquid_to_seed.gif" alt="ormus-pickup" width="128" style="image-rendering: pixelated;" />
</p>

<h1 align="center">ormus-pickup</h1>

<p align="center">
  <em>Restore session context at the start of a new Claude Code session. Inverse of /handoff.</em>
</p>

<p align="center">
  <a href="https://github.com/HermeticOrmus/ormus-pickup/stargazers"><img src="https://img.shields.io/github/stars/HermeticOrmus/ormus-pickup?style=flat-square&color=aa8142" alt="Stars" /></a>
  <a href="https://github.com/HermeticOrmus/ormus-pickup/blob/main/LICENSE"><img src="https://img.shields.io/github/license/HermeticOrmus/ormus-pickup?style=flat-square&color=aa8142" alt="License" /></a>
  <a href="https://github.com/HermeticOrmus/ormus-pickup/commits"><img src="https://img.shields.io/github/last-commit/HermeticOrmus/ormus-pickup?style=flat-square&color=aa8142" alt="Last Commit" /></a>
  <img src="https://img.shields.io/badge/Claude_Code-aa8142?style=flat-square&logo=anthropic&logoColor=white" alt="Claude Code" />
</p>

---

> **Restore session context at the start of a new Claude Code session. The entry ritual — the inverse of /handoff.**

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

- **[ormus-handoff](https://github.com/HermeticOrmus/ormus-handoff)** — the other half. Run /handoff at session end, /pickup at session start.
- **[ormus-absorb](https://github.com/HermeticOrmus/ormus-absorb)** — distill conversation knowledge into persistent memory so /pickup loads it automatically.
- **[ormus-explore](https://github.com/HermeticOrmus/ormus-explore)** — token-efficient AST-based code search, useful during cold pickup when you need to read 6+ files.
- **[ormus-vibe-proof](https://github.com/HermeticOrmus/ormus-vibe-proof)** — security hardening for vibe-coded full-stack apps.
- **[ormus-meta-prompting](https://github.com/HermeticOrmus/ormus-meta-prompting)** — categorical foundations for AI prompt engineering.

Together they form the **ormus session lifecycle** — composable Claude Code skills for serious cross-day, cross-machine work.

## License

MIT. See [LICENSE](LICENSE).

## Origin

Distilled from real cross-day Claude Code engagements where re-establishing context cost ten minutes of re-explaining at the start of every session. The cold-pickup path emerged after enough sessions ended without a handoff that the skill needed a fallback. The optional remote routing emerged from patterns where the same project lives on multiple machines and "pickup" needs to know which one to look at.

---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well (hypothesis → scope → validate → reject working-but-wrong)
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [shell-safety-skills](https://github.com/HermeticOrmus/shell-safety-skills) — `set -euo pipefail` discipline + 15 failure-mode examples
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 + commit-msg hook + commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [typescript-conventions-skills](https://github.com/HermeticOrmus/typescript-conventions-skills) — TypeScript strict mode, discriminated unions, Result types
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token + context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff + pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
