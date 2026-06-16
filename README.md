# Claude Coding Agents

Production-grade slash commands for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that enforce staff-level engineering discipline. These commands turn Claude into a rigorous implementation partner — not just a code generator.

## What's Included

| Command | Purpose |
|---|---|
| `/elite-implementation` | Phased implementation workflow: understand → blast-radius analysis → plan → implement → self-review. No code is written until you approve the plan. |
| `/elite-review` | Comprehensive PR code review across 12 dimensions (correctness, security, ORM, concurrency, architecture, etc.) with severity-rated findings. |

## Quick Setup

### Option A: Global (available in every project)

```bash
# Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Download the commands
curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-implementation.md \
  -o ~/.claude/commands/elite-implementation.md

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-review.md \
  -o ~/.claude/commands/elite-review.md
```

### Option B: Per-project (scoped to one repo)

```bash
# From your project root
mkdir -p .claude/commands

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-implementation.md \
  -o .claude/commands/elite-implementation.md

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-review.md \
  -o .claude/commands/elite-review.md
```

### Option C: One-liner (global)

```bash
mkdir -p ~/.claude/commands && for f in elite-implementation elite-review; do curl -sL "https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/${f}.md" -o ~/.claude/commands/${f}.md; done
```

## Usage

Open Claude Code in any project and run:

```
/elite-implementation Add user authentication with JWT tokens
```

```
/elite-review 42
```

### `/elite-implementation <task description>`

Enforces a 5-phase workflow:

1. **Phase 1 — Understand:** Restates scope, reads all relevant code, identifies conventions and assumptions.
2. **Phase 2 — Blast radius:** Maps callers, callees, data layer, API contracts, downstream consumers, and test coverage.
3. **Phase 3 — Plan:** Presents files to change, data flow, approach vs. alternatives, and risks. **Waits for your approval.**
4. **Phase 4 — Implement:** Writes modular, secure, efficient, production-grade code.
5. **Phase 5 — Self-review:** Re-reads the full diff against a checklist before declaring done.

### `/elite-review [PR-number-or-URL]`

Reviews a PR across 12 dimensions:

| # | Dimension | Examples |
|---|---|---|
| 1 | Correctness & Logic | Bugs, race conditions, edge cases |
| 2 | Django ORM & Database | N+1 queries, missing indexes, unbounded querysets |
| 3 | Transactions & Concurrency | Missing `atomic()`, read-modify-write races |
| 4 | Security | SQL injection, IDOR, hardcoded secrets, XSS |
| 5 | Django REST Framework | Serializer leaks, missing pagination/throttling |
| 6 | Celery & Background Tasks | Non-idempotent tasks, missing retries |
| 7 | Error Handling & Resilience | Swallowed exceptions, missing timeouts |
| 8 | Logging & Observability | Duplicate logs, missing context, wrong levels |
| 9 | Migrations | Non-nullable without default, table locks |
| 10 | Architecture & Modularity | Fat views, god models, DRY violations |
| 11 | Production Readiness | DEBUG leaks, missing env vars, idempotency |
| 12 | Testing | Missing coverage, flaky tests, real network calls |

Findings are rated by severity:
- 🔴 **BLOCKER** — Must fix before merge
- 🟠 **MAJOR** — Should fix before merge
- 🟡 **MINOR** — Fix soon
- 🔵 **SUGGESTION** — Optional improvement

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed
- `gh` CLI (for `/elite-review` to fetch PR diffs)

## Customization

These commands are plain markdown files. Fork and edit them to:

- Adjust the review dimensions for your stack (React, Go, Rails, etc.)
- Add or remove implementation phases
- Change severity definitions
- Add project-specific conventions

## License

MIT
