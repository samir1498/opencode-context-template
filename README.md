# opencode-context-template

Two patterns I use with CLI AI agents (opencode, Claude CLI) for persistent memory across sessions and machines.

---

## 1. Context Repo (`personal-context/`)

A folder that survives context windows. The agent reads it at session start, writes to it during work, and commits changes so other machines can sync.

### Convention (not a tool)

The agent always reads `MAP.md` first. That file lists everything: what's in progress, what plans exist, where references live.

Folders inside:

| Folder | Purpose |
|--------|---------|
| `progress/` | `now.md` (current snapshot), `daily.md` (intraday log), `YYYY-Www.md` (weekly logs) |
| `plans/` | Scoped execution plans with tasks |
| `ideas/` | Rough captures, brainstorming |
| `references/` | Stable docs, checklists, recurring knowledge |
| `archive/` | Completed or stale items |

Rules:
- Text-only. No binaries.
- When `daily.md` or `now.md` exceed 300 lines: archive them, start fresh.
- Agent commits and pushes context changes.
- The context repo is a **separate git repo** from your code repos — it has its own remote, syncs independently.
- On each machine: pull before work, push after.

---

## 2. Research Repo

Separate from the context repo. Research is **investigation output** — you asked a question, got an answer, saved it. Context is **live planning** — what you're doing right now.

### Convention

```
personal-research/
├── domain-name/
│   ├── 2026-06-17-topic-name.md
│   └── another-finding.md
└── README.md
```

Rules:
- One topic per file. Self-contained — the agent reads one file and has all the context.
- No deep nesting. Two levels max (domain/filename.md).
- Timestamp prefix when chronology matters (`YYYY-MM-DD-slug.md`).
- Domain folders = categories that make sense for your work (engineering, marketing, seo, etc.)
- Agent writes findings back to the relevant file, never creates new files without asking.

---

## 3. Plugins

Two opencode plugins that make this system work across sessions:

| Plugin | What it does | Where it goes |
|--------|-------------|---------------|
| `open-mem` | Cross-session memory via SQLite. Stores decisions, bugs, features, discoveries — survives context window resets. Query with `mem-find`. | Project `.opencode/opencode.json` — install per project |
| `opencode-skills-collection` | 100+ community skills for common tasks (AI, backend, cloud, testing, etc.). Skills auto-load without explicit import. | Global `~/.config/opencode/opencode.json` — install once |

### Install

```bash
# Global — community skills
opencode plugin install opencode-skills-collection

# Per project — cross-session memory
cd <your-project>
opencode plugin install open-mem
```

---

## 4. Skills

Reusable agent instructions that the AI loads by name. Not tied to any specific project — just patterns that work anywhere.

| Skill | What it does |
|-------|-------------|
| `pc-context` | Read/update the context system: view now.md, append to daily.md, archive files |
| `pc-plan` | Create, read, or list execution plans in the plans folder |
| `pc-diagnose` | Debugging loop: reproduce → minimise → hypothesise → fix |
| `pc-git-guardrails` | Git safety rules embedded in agent behavior |
| `pc-tdd` | Test-driven development: red-green-refactor |
| `pc-pr` | Pull request format: branch naming, commit strategy, PR body template |
| `pc-grill-me` | Stress-test a plan or design by asking hard questions |
| `pc-zoom-out` | Get a higher-level perspective on unfamiliar code |
| `pc-coding-style` | Code conventions: naming, commits, comments |

The `pc-` prefix just means these belong to the personal-context system.

---

## How to start

1. Clone this repo wherever you want.
2. Initialize `personal-context/` as its own git repo with a remote (GitHub, etc.).
3. On each machine where you work, clone the context repo separately.
4. On first session, tell your agent about the system.
5. Research can be a folder in your workspace or a separate repo — whatever fits.