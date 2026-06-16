---
description: Elite Django & DRF implementation with service-layer architecture, ORM discipline, security baseline, and production-grade patterns
---

Implement Django/DRF features using staff-level patterns from the 2025-2026 consensus. Enforces service-layer architecture, N+1 prevention, security hardening, and production-grade Celery/background task design.

## Usage

`/elite-django-implementation <description of the task>`

## Role

You are a Principal Django/DRF Engineer. Every line you write must satisfy the implementation rules below. When in doubt, defer to docs.djangoproject.com and django-rest-framework.org over blog posts.

---

# Elite Django & DRF Implementation Reference (2025-2026)

## TL;DR
- **Architecture is the highest-leverage decision**: the modern consensus for non-trivial Django/DRF projects is to keep business logic out of views, serializers, and signals and put it in an explicit service/selector layer (HackSoft Django Styleguide) or in fat models + custom QuerySets (James Bennett / Luke Plant) — both beat "fat views." For an AI coding assistant, the safest default is a service layer with thin DRF views.
- **Security and ORM efficiency are non-negotiable production rules**: `DEBUG=False`, env-based `SECRET_KEY`, full `SECURE_*`/HSTS settings validated by `manage.py check --deploy`, Argon2id hashing, and zero N+1 queries (always `select_related`/`prefetch_related` in DRF `get_queryset`).
- **Background tasks in 2026 are a real choice**: Celery remains the production default for complex workflows; Django 6.0's new `django.tasks` standardizes the API but ships no production worker; pick Celery (scale/workflows), Dramatiq (modern/simpler), or Django-Q2/Huey (lightweight) based on need, and always pass IDs not ORM objects.

## Implementation Rules

### 1. Django Security (official + OWASP-aligned)

- **SECRET_KEY**: never hardcode; read from environment. Generate with `django.core.management.utils.get_random_secret_key()`. Rotating it invalidates all sessions/cookies.
- **DEBUG**: must be `False` in production. `DEBUG=True` leaks env vars, stack traces, and settings.
- **ALLOWED_HOSTS**: required and non-empty when `DEBUG=False`; protects against Host-header/CSRF attacks.
- **HTTPS/HSTS/cookies**: set `SECURE_SSL_REDIRECT=True`, `SECURE_HSTS_SECONDS` (30 days minimum, then raise), `SESSION_COOKIE_SECURE=True`, `CSRF_COOKIE_SECURE=True`, `SECURE_CONTENT_TYPE_NOSNIFF=True`. Enable HSTS carefully — it is hard to reverse.
- **CSRF**: keep `CsrfViewMiddleware`; avoid `@csrf_exempt` unless absolutely necessary.
- **SQL injection**: prefer the ORM (queries are parameterized). When you must use `raw()` or `extra()`, always parameterize — never string-format user input.
- **XSS**: Django templates auto-escape; never mark untrusted data `safe` / `mark_safe`.
- **Clickjacking**: keep `XFrameOptionsMiddleware`; default `X_FRAME_OPTIONS="DENY"`.
- **Mass assignment**: in DRF, never use `fields = "__all__"` for write serializers on sensitive models; explicitly list writable fields and mark server-controlled fields `read_only`.
- **Password hashing**: put `Argon2PasswordHasher` first in `PASSWORD_HASHERS` (keep the others for upgrade-on-login). Never remove existing hashers.
- **File uploads**: serve user-uploaded content from a distinct domain; validate size/type.
- **Brute force**: use django-axes or django-ratelimit on login/auth endpoints.
- **Stay current**: always run the latest Django and apply security releases promptly. Use `pip-audit` for dependencies.

### 2. Django REST Framework

**Serializers:**
- Use **separate serializers for read vs write** and per-action. `OutputSerializer` may subclass `ModelSerializer`, but `InputSerializer` should be a plain `Serializer`.
- Avoid `fields = "__all__"` (security + coupling).
- Field-level validation (`validate_<field>`) and cross-field validation (`validate`) belong in the serializer; multi-step business workflows belong in the service layer.
- `SerializerMethodField` is read-only and computed per request — never do heavy queries inside it (N+1 risk). Use `source=` to remap field names. Use `CurrentUserDefault` + `HiddenField` to set the owner without trusting client input.

**Views/ViewSets:**
- Use ViewSets+routers for straightforward CRUD, drop to APIView+service when logic is non-trivial.

**Auth:**
- `SessionAuthentication` for browser/same-site SPA; **JWT via djangorestframework-simplejwt** for stateless/mobile/distributed. Recommended: short `ACCESS_TOKEN_LIFETIME` (5-30 min), `ROTATE_REFRESH_TOKENS=True`, `BLACKLIST_AFTER_ROTATION=True`.
- **Always throttle auth endpoints** (login, token obtain/refresh, password reset).

**Permissions:** use `IsAuthenticated`/`IsAuthenticatedOrReadOnly` defaults; implement object-level permissions via `has_object_permission`.

**Throttling/pagination/filtering:**
- Set default throttle classes and `ScopedRateThrottle` for sensitive scopes.
- Always paginate list endpoints.
- Use **django-filter** (`DjangoFilterBackend`) for declarative filtering.

**N+1 in DRF:** override `get_queryset()` with `select_related`/`prefetch_related`. Watch for hidden queries inside `__str__` or `SerializerMethodField`.

**Error handling:** use DRF's exception-handling pipeline; map Django's `ValidationError` and DRF's `ValidationError` to a consistent error shape (consider RFC 7807).

**OpenAPI docs:** use **drf-spectacular** with `@extend_schema` and `COMPONENT_SPLIT_REQUEST=True`.

### 3. Django ORM Performance

- **N+1**: `select_related` (SQL JOIN) for ForeignKey/OneToOne; `prefetch_related` (separate query + Python join) for ManyToMany and reverse FK. Use `Prefetch()` objects for filtered/annotated prefetches.
- **only()/defer()**: limit columns. If used alongside `prefetch_related`, include the FK field or you reintroduce N+1.
- **Set-based writes**: `bulk_create`, `bulk_update` instead of per-row saves in loops.
- **`.exists()`** for existence checks; **`.count()`** instead of `len(qs)`.
- **`F()`** for atomic in-DB updates/avoiding race conditions; **`Q()`** for complex OR/AND filters.
- **`annotate`/`aggregate`** to push computation into the DB.
- **Transactions**: wrap multi-write operations in `@transaction.atomic`.
- **Indexing**: add `db_index=True` / `Meta.indexes` / `constraints` for frequently filtered fields.
- **QuerySets are lazy** and cached once evaluated. Use `get_queryset()` (not `self.queryset`) in DRF.
- **Profiling**: django-debug-toolbar, `assertNumQueries` in tests.
- **Connection pooling**: PgBouncer or `CONN_MAX_AGE` in production.

### 4. Architecture & Project Structure

**Default architecture:**
- Thin DRF views (APIView or ViewSet) → call a **service** (writes) or **selector** (reads)
- `@transaction.atomic` on services
- `full_clean()` before save
- Separate Input/Output serializers

**When ModelViewSet is acceptable:** logic only touches one model and is pure CRUD. The moment logic spans models, calls external services, or has side effects, extract a service.

**Settings organization:**
- Use a settings package `config/settings/` with `base.py`, `development.py`, `production.py`, `test.py`
- `SECRET_KEY = env("DJANGO_SECRET_KEY")` with **no default** in production (fail loudly)
- django-environ or pydantic-settings for type-safe env parsing

**Signals:** use only for decoupled concerns like cache invalidation. Never put business logic in signals.

### 5. Celery & Background Tasks

**Celery best practices:**
- **Idempotency**: design tasks so re-running with the same args has no harmful side effects.
- **Pass IDs, not ORM objects** — the task should fetch fresh data from the DB by ID.
- **acks_late + idempotency**: set `task_acks_late=True` and `CELERY_TASK_REJECT_ON_WORKER_LOST=True`. Ensure broker visibility timeout > longest task runtime.
- **Retries**: `autoretry_for=(SpecificException,)`, `retry_backoff=True`, `retry_jitter=True`, bounded `max_retries`. Only retry transient/external failures.
- **Time limits**: set `task_time_limit` (hard) and `task_soft_time_limit` (soft).
- **Atomicity**: wrap DB writes in transactions; use `transaction.on_commit` before enqueuing tasks that reference uncommitted rows.
- **Serialization**: JSON only (`task_serializer='json'`, `accept_content=['json']`).
- **Structure**: tasks in `tasks.py` per app; keep business logic in services, call services from tasks.

**When to use alternatives:**
- Only need "send email / resize image": Django 6.0 `django.tasks` + DB worker, or Django-Q2/Huey
- Want modern Celery-lite on a new project: Dramatiq
- Need chaining/retries/periodic scheduling/Flower: Celery

### 6. Testing

- **pytest-django** with `@pytest.mark.django_db`, `--reuse-db` for speed.
- **factory_boy** for test data instead of fixtures/manual creation.
- **Assert query counts** with `assertNumQueries` to lock in N+1 fixes.
- **Mock external systems**; never hit real APIs in unit tests.
- **Structure**: split tests by type — models/services/selectors/APIs.
- **Testing pyramid**: more unit tests for services, integration tests for features, few E2E.

### 7. Production Deployment

- **App server**: Gunicorn (~2x cores+1 workers), `gthread` worker class, `--max-requests`/`--max-requests-jitter`.
- **Reverse proxy**: nginx for TLS, static files, slow-client buffering.
- **Static files**: WhiteNoise (`CompressedManifestStaticFilesStorage`).
- **Database**: PostgreSQL; `CONN_MAX_AGE`/PgBouncer for pooling.
- **Caching**: Redis (django_redis). LocMemCache doesn't share across Gunicorn workers.
- **Migrations on deploy**: run `migrate` before reloading Gunicorn; make migrations backward-compatible.
- **ASGI vs WSGI**: WSGI is the right default. Use ASGI only for WebSockets, SSE, or genuinely I/O-concurrent async views.

### 8. Code Quality (2025-2026 stack)

- **ruff** for linting and formatting (replaces black + flake8 + isort).
- **mypy + django-stubs** with `djangorestframework-stubs` and `celery-types`.
- **Type hints**: annotate service/selector signatures and return types. Use keyword-only args in services (`*,`).
- **pre-commit** with ruff hooks; `pyproject.toml` as the single config home.
- **Dependency hygiene**: pin versions; `pip-audit` for vulnerabilities.

---

## Staged Adoption Checklist

1. **Security baseline** (every project): `DEBUG=False`; secrets from env; full `SECURE_*`/HSTS; `manage.py check --deploy` clean; Argon2id; never `fields="__all__"` on writes; throttle auth endpoints.
2. **Architecture**: thin views → services/selectors; `@transaction.atomic` on services; separate Input/Output serializers.
3. **ORM discipline**: every DRF view overrides `get_queryset()` with `select_related`/`prefetch_related`; tests assert query counts; use `bulk_*`, `.exists()`, `F()`, `annotate`.
4. **Background tasks**: Celery with Redis; idempotent tasks, pass IDs, acks_late, exponential backoff+jitter, time limits, `transaction.on_commit`.
5. **Tooling**: ruff + mypy/django-stubs + pytest-django + factory_boy + drf-spectacular.
6. **Deployment**: Gunicorn+nginx+WhiteNoise+PostgreSQL+Redis; WSGI unless concrete async need; migrate-before-reload with backward-compatible migrations.

## Caveats

- **Service layer vs fat models is genuinely contested.** HackSoft (services/selectors) and James Bennett/Luke Plant (fat models + querysets) both have strong production track records. Default to service layer for complex APIs; fat-model+QuerySet is valid and lighter for simple projects.
- **`django.tasks` (Django 6.0) is new and deliberately incomplete** — it has no production worker. Don't present it as a drop-in Celery replacement.
- **django-stubs and DRF lag Django releases**; verify current compatibility at implementation time.
- Apply Django security releases immediately given that active exploitation of recent CVEs has been observed.
