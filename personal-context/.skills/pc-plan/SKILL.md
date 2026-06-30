---
name: pc-plan
description: Read, create, or list execution plans in the personal-context/plans system
argument-hint: [list|read <name>|create <name>]
---

Manage execution plans in `personal-context/plans/`.

## MCP tools (preferred)
Use these MCP tools over raw CLI:
- `plan_list` — list all plans (filter by status/category)
- `plan_show` — full plan details by slug
- `plan_add` — create a new plan
- `plan_add_task` — add a task to a plan
- `plan_references` — show refs + backlinks
- `plan_validate` — validate all plan files
- `roadmap_list` / `roadmap_show`
- `research_list` / `research_show`

Fall back to `ctx <subcommand>` if MCP tools are unavailable.

## Behavior

- **"list" or no args** — Use `plan_list` MCP tool (fallback: `ctx list`)
- **"read <slug>"** — Use `plan_show` MCP tool (fallback: `ctx show <slug>`)
- **"create <title>"** — Use `plan_add` MCP tool with title, category, priority (fallback: `ctx plan add "<title>" --category <c> --priority <n> [--ref research:<slug>]`)

## Plan format

All plans use YAML frontmatter with these fields:

```yaml
---
title: Plan Title
slug: plan-slug              # from filename, used by CLI
status: active               # active | paused | done | cancelled
category: gam                # gam | blog | infra | java | ml | learning
created: 20260616            # YYYYMMDD
tldr: One sentence summary
priority: 50                 # 0-100 scale
tags: [tag1, tag2]
tasks:                       # optional
  - id: task-1
    desc: Description
    status: pending          # pending | in-progress | done | blocked
    refs: [research:slug]    # optional: link task to research finding
acceptance:                  # optional
  - id: ac-1
    desc: Testable condition
references:                  # optional: cross-links to research, plans, or URLs
  - research:playwright-react19-file-input-flakiness
  - plan:another-plan-slug
  - url:https://github.com/owner/repo/issues/123
---
```

## Cross-reference rules

- Use `research:<slug>` to reference a file in `personal-research/`
- Use `plan:<slug>` to reference another plan or roadmap
- Use `url:<full-url>` for external links
- Add references to tasks when the task is directly informed by research (`refs` field)
- Add references at the plan level when the whole plan depends on or relates to research
- After adding refs, use `plan_references` MCP tool to verify (fallback: `ctx plan references <slug>`)
- When completing a plan that was informed by research, update the research INDEX.md status if applicable

## After creating
- Run `plan_validate` MCP tool to verify (fallback: `ctx validate`)
- Commit: `git -C personal-context commit -am "plan: add <slug>"`
