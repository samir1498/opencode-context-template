---
name: pc-code-quality-debug
description: Systematic code quality audit — service depth, DRY, N+1, UI duplication, module coupling, error handling, test coverage, and blind spots often missed. Use when asked to debug code quality, audit architecture, or review PRs.
---

# Code Quality Debug Skill

Step-by-step audit template. Follow in order, record findings, then report using the template at the bottom.

---

## Audit Procedure

### Phase 1: Architecture Scan

For each service layer module, check every public function:

- [ ] Does it delegate 1:1 to a repo with no added logic? → **too shallow**
- [ ] Does it have real business logic (transactions, validation, composition)?
- [ ] Does it write raw SQL/queries instead of delegating to the repo, while the repo has the same functions unused? → **service bypasses repo**

For each repo/DAO module:
- [ ] Grep (`rg`) for every public function to find truly unused callers across ALL workspace crates
- [ ] Mark any function with zero callers as **dead code**

### Phase 2: DRY & Duplication Scan

- [ ] Same formula calculated in both service and repo? (e.g., `line_total = qty * price`)
- [ ] Same UI pattern repeated across N screens → compute `boilerplate_ratio`:
  - Estimate unique_content_lines per screen
  - `boilerplate_ratio = 1 − (sum(unique) / total_lines)`
  - >30% = extraction candidate
- [ ] Same modal/dialog boilerplate repeated (Window config, save/cancel buttons, error display)
- [ ] Action buttons doing the same thing but styled differently in different screens
- [ ] Pagination wiring (count + fetch + pagination_ui) repeated in every screen

### Phase 3: Data Flow

- [ ] N+1 queries: loop with per-entity query inside? Batch with JOIN instead.
- [ ] Model/DTO gaps: fields missing from create-DTOs forcing callers to use raw SQL? (e.g., `NewProduct` missing quantity → seed does `diesel::update` after create)
- [ ] Can initial entity state be set through the public API, or do callers need workarounds?

### Phase 4: Module Boundaries

- [ ] Cross-module queries: module A queries module B's tables directly?
- [ ] Enums defined with Display/FromStr but model stores String and comparisons use literals? → dead enum
- [ ] Business state struct living in a style/UI helper file?
- [ ] Importing internal data access from consumer layer instead of going through service?

### Phase 5: Error & Security

- [ ] Domain error types or raw driver errors / `.to_string()` displayed to user?
- [ ] Validation messages hardcoded when app uses i18n?
- [ ] `.unwrap()` / `.expect()` on user input in production paths?
- [ ] Hardcoded paths (DB in CWD instead of platform data dir)?
- [ ] Singleton initializer called in multiple places (panics on second call)?

### Phase 6: Build & Test

- [ ] `cargo check` / `cargo clippy -- -D warnings` — any warnings?
- [ ] `cargo fmt --check` — any formatting issues?
- [ ] Unused dependencies in Cargo.toml?
- [ ] Tests exist? Unit, integration, snapshot?
- [ ] Are services testable (take connection/context as parameter)?

---

## Report Template

```
## TOO SHALLOW — Zero Value Added

| Module | Why | Functions affected |
|--------|-----|--------------------|
| ...    | ... | ...                |

## DRY VIOLATIONS

| Location | What |
|----------|------|

## N+1 QUERIES

| Location | What |
|----------|------|

## MISPLACED CONCERNS / DEAD CODE

| Location | What |
|----------|------|

## INCONSISTENCIES

| Location | What |
|----------|------|

## BUGS

| Location | Severity | What |
|----------|----------|------|

## UI BOILERPLATE RATIO

| Screen | Lines | Unique content | Boilerplate |
|--------|-------|----------------|-------------|
| Total  |       |                | ~XX%        |

## WHAT'S GOOD

- ...
```

---

## Things I Often Forget To Check (blind spots from past audits)

- **Dead code (beyond obvious)** — `rg <function_name> --include '*.rs'` for EVERY public function. Don't assume. Including enums that are defined+imported but never matched.
- **Model/DTO gaps** — `NewProduct` missing `quantity_on_hand` forces seed to use raw SQL. Check if all create-DTOs let you set every DB column.
- **Cross-layer duplication** — same formula (`line_total = qty * price`) calculated in both service and repo. One copy is dead, the other is duplicated risk.
- **Service bypassing repo** — service writes raw queries when repo has the matching functions. The repo functions are dead code, and the service has unabstracted SQL.
- **`rg` before declaring dead** — grep across ALL workspace crates, not just the current module's directory.
- **Migration files** — do they match the current schema/models? Any stale migrations?
- **Feature flags** — `cargo tree -e features` to check for unnecessary feature bloat.
- **Linter output** — always run `cargo clippy -- -D warnings` as a final pass.
- **Platform assumptions** — hardcoded `/`, Unix-only paths, `dirs::data_dir()` vs CWD.
- **Concurrency** — mutable global state, `Send`/`Sync` bounds, re-entrant UI state.
- **Observability** — `log` vs `tracing`, structured or ad-hoc, repeat init panics.
- **Dependency freshness** — `cargo outdated`, major version gaps.
- **Build time** — expensive macros, large `include_bytes!` assets.
