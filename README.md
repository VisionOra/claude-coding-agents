# Claude Coding Agents

Production-grade slash commands for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that enforce staff-level engineering discipline. These commands turn Claude into a rigorous implementation partner — not just a code generator.

## What's Included

| Command | Purpose |
|---|---|
| `/elite-implementation` | Phased implementation workflow: understand → blast-radius analysis → plan → implement → self-review. No code is written until you approve the plan. |
| `/elite-review` | Comprehensive PR code review across 12 dimensions (correctness, security, ORM, concurrency, architecture, etc.) with severity-rated findings. |
| `/elite-django-implementation` | Django/DRF-specific implementation enforcing service-layer architecture, N+1 prevention, security hardening, Celery best practices, and the 2025-2026 community consensus. |

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

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-django-implementation.md \
  -o ~/.claude/commands/elite-django-implementation.md
```

### Option B: Per-project (scoped to one repo)

```bash
# From your project root
mkdir -p .claude/commands

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-implementation.md \
  -o .claude/commands/elite-implementation.md

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-review.md \
  -o .claude/commands/elite-review.md

curl -sL https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/elite-django-implementation.md \
  -o .claude/commands/elite-django-implementation.md
```

### Option C: One-liner (global)

```bash
mkdir -p ~/.claude/commands && for f in elite-implementation elite-review elite-django-implementation; do curl -sL "https://raw.githubusercontent.com/VisionOra/claude-coding-agents/main/${f}.md" -o ~/.claude/commands/${f}.md; done
```

## Usage

Open Claude Code in any project and run:

```
/elite-implementation Add user authentication with JWT tokens
```

```
/elite-review 42
```

```
/elite-django-implementation Add a paginated product search API with filtering
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

### `/elite-django-implementation <task description>`

Django/DRF-specific implementation command covering 8 areas:

| # | Area | What It Enforces |
|---|---|---|
| 1 | Security | OWASP-aligned settings, Argon2id, CSRF/XSS/SQLi prevention, `check --deploy` |
| 2 | DRF Patterns | Separate read/write serializers, throttling, pagination, drf-spectacular |
| 3 | ORM Performance | `select_related`/`prefetch_related`, `F()`/`Q()`, bulk ops, query count assertions |
| 4 | Architecture | Service/selector layer, thin views, `@transaction.atomic`, split settings |
| 5 | Celery | Idempotent tasks, pass IDs, acks_late, backoff+jitter, `on_commit` |
| 6 | Testing | pytest-django, factory_boy, `assertNumQueries`, mocked externals |
| 7 | Deployment | Gunicorn+nginx+WhiteNoise+PostgreSQL+Redis, backward-compatible migrations |
| 8 | Code Quality | ruff, mypy+django-stubs, type hints, pre-commit |

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
