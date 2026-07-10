# 01 — Foundation: `db.py` + `config.py`

These two files set up **where** data lives (the SQL engine/session) and **how** the app is
configured (env vars → a typed `Settings` object). Nothing here holds business data; every
other file depends on them.

---

## `db.py` — engine, session factory, declarative base

**Purpose:** create one SQLAlchemy engine + session factory that works identically on Postgres
(prod) and SQLite (dev), and expose the `Base` all models inherit from.
**Inputs:** `settings.database_url`. **Outputs:** `engine`, `SessionLocal`, `Base`, `get_db()`.
**Tables touched:** none directly — it's the plumbing every table uses.

**L1-7** Module docstring: Postgres is the prod target; SQLite is a first-class fallback
(`DATABASE_URL=sqlite:///./mallory.db`). The two dialect shims below make the same models run
on both.
**L9** `from __future__ import annotations` — lets type hints be strings (lazy eval); standard
top-of-file line, present in nearly every file. (Annotated once here; skipped elsewhere.)
**L11** `from collections.abc import Iterator` — type for `get_db()`'s yield.
**L13** Import `create_engine` (builds the DB connection pool) and `event` (hook DB lifecycle
events).
**L14** Import `JSONB` — Postgres JSON column type; models use it for list/dict columns.
**L15** `from sqlalchemy.ext.compiler import compiles` — lets us override how a type renders in
SQL per dialect.
**L16** Import ORM pieces: `DeclarativeBase` (base class for models), `Session` (unit of work),
`sessionmaker` (session factory).
**L18** `from .config import get_settings` — pull the resolved config.

**L21-23** `@compiles(JSONB, "sqlite")` + `_jsonb_as_json_on_sqlite(...)` — **dialect shim #1.**
When any model column is declared `JSONB`, on SQLite emit `JSON` instead (SQLite has no JSONB).
So one model definition works on both DBs. Returns the literal SQL type string `"JSON"`.

**L26** `_settings = get_settings()` — load config once (it's `@lru_cache`d, so this is cheap
and shared).
**L28** `_is_sqlite = _settings.database_url.startswith("sqlite")` — branch flag: are we on
SQLite? Drives the next three decisions.
**L29-34** `engine = create_engine(...)` — the connection pool. Args:
- `_settings.database_url` — where to connect.
- `pool_pre_ping=not _is_sqlite` — on Postgres, test a connection before using it (avoids stale
  dropped connections); pointless on SQLite (file-based), so off there.
- `connect_args=... if _is_sqlite else {}` — SQLite only: `check_same_thread=False` (FastAPI
  serves requests on multiple threads sharing the connection) + `timeout=30` (wait for locks).
- `future=True` — use SQLAlchemy 2.0 style.

**L36-46** `if _is_sqlite:` block — **dialect shim #2:** register a `connect` event so every new
SQLite connection runs three PRAGMAs:
- **L43** `journal_mode=WAL` — write-ahead logging: concurrent readers don't block the one
  writer (the scheduler). Critical because the crawler POSTs pages while the pipeline runs.
- **L44** `busy_timeout=30000` — wait up to 30 s for a held write lock instead of instantly
  failing with "database is locked".
- **L45** `synchronous=NORMAL` — faster writes, safe under WAL.
- Postgres needs none of this (comment L41), so the whole block is skipped there.

**L47** `SessionLocal = sessionmaker(bind=engine, autoflush=False, expire_on_commit=False,
future=True)` — the **session factory**. Key choices:
- `autoflush=False` — the ORM won't silently flush pending changes before every query; the
  pipeline flushes explicitly (this is why `runner.py` calls `db.flush()` at specific points).
- `expire_on_commit=False` — objects stay usable (attributes readable) after `commit()`.

**L50-51** `class Base(DeclarativeBase)` — the shared declarative base. **Every model in
`models/` subclasses this**, and importing them registers their tables on `Base.metadata`
(used by `init_db` to `create_all`).

**L54-60** `def get_db() -> Iterator[Session]` — FastAPI dependency. Opens a session
(`SessionLocal()`), `yield`s it to the request handler, and **always** closes it in `finally`.
This is the `Depends(get_db)` you'll see in every API route.

---

## `config.py` — typed settings from env / `.env`

**Purpose:** one `Settings` object (pydantic-settings) holding every tunable — DB URL, which
LLM provider + models, seed dir, scheduler, MinIO, patent APIs, CORS. Plus a one-word
`LLM_TARGET` switch that fills in a whole preset of Ollama URLs/models.
**Inputs:** environment variables / `.env` file. **Outputs:** a cached `Settings` instance via
`get_settings()`. **Tables touched:** none.

**L1** Docstring: config loaded from environment / `.env`.
**L5** `from functools import lru_cache` — to memoize `get_settings()`.
**L7-8** Import pydantic `model_validator` (post-load hook) and `BaseSettings`/
`SettingsConfigDict` (env-driven settings base).

**L10-51** `_LLM_PRESETS` — the **one-line LLM switch**. Each key (`farm`, `local`,
`local-docker`, `lan50`) is a bundle of Ollama base URL + model names + timeout. Setting
`LLM_TARGET=lan50` pulls that whole bundle in (resolved in `_apply_llm_target` below). Notable:
- **L13-20** `farm` — remote hosted Ollama (`ollama.i3softlab.com`), needs an API key; serves
  `text-model` + `vlm-model`; 120 s timeout to ride out slow tails. No embed model.
- **L21-30** `local` — this host's Ollama on `localhost:11434`, small `qwen2.5:3b` for
  everything; local `bge-m3` embeddings; vision model absent so captioning no-ops.
- **L31-40** `local-docker` — same but reached from *inside* a container
  (`host.docker.internal`).
- **L41-50** `lan50` — the `192.168.5.50` LAN box with the full model set (7b fast / 14b deep /
  vl 7b vision / nomic embed).

**L54** `class Settings(BaseSettings)` — the config object.
**L55** `model_config = SettingsConfigDict(env_file=".env", ..., extra="ignore")` — read
`.env`, ignore unknown env vars (don't crash on extras).

Settings fields (each is `name: type = default`, overridable by env var of the same name
upper-cased):
**L58** `database_url` — default points at a local Postgres (`mallory:mallory@localhost:5432`).
**L61** `llm_provider` — `stub` (deterministic, no key) | `ollama` | `anthropic` | `openrouter`.
This picks which provider class `get_llm()` returns.
**L64** `llm_target` — the preset key ("" = use explicit `OLLAMA_*` fields).
**L65-66** `anthropic_api_key`, `anthropic_model` (default `claude-sonnet-4-6`).
**L69-71** OpenRouter key/model/base_url (OpenAI-compatible; used for chat/enrichment/reports).
**L75-84** Ollama config: `ollama_base_url`, `ollama_api_key`, and **three model roles** —
`ollama_model_fast` (extract/classify, high volume), `ollama_model_deep` (synthesis/verdicts,
heavy reasoning), `ollama_model_vision` (image captions). Plus a **separate embed endpoint**
(`ollama_embed_*`) because the farm has no embed model, so embeddings stay local even when
chat/vision point at the farm.
**L85-87** `llm_timeout_s=120`, `llm_num_ctx=8192`, `llm_cache_enabled=True`.
**L90** `seed_dir="./seed_data"` — where the `ref_*` seed JSON lives (also `spec_slots.json`,
read by `spec_extract.py`).
**L92-95** In-process scheduler: `scheduler_enabled` (off by default), `scheduler_interval_s`
(120). When on, `main.py` runs `process_pending` on a loop.
**L98** `crawler_ingest_url` — fallback asset proxy when MinIO isn't set.
**L100-106** MinIO/S3: where the crawler wrote blob assets. If `minio_endpoint` set, L2 reads
asset bytes straight from MinIO by `s3://mallory-raw/...`; blank ⇒ proxy through the crawler.
**L108-112** Patent sources tried in order: SerpApi → USPTO ODP → keyless Google Patents.
**L115** `cors_origins` — comma-separated origins for the Layer 3 client.

**L117-119** `cors_origin_list` property — split `cors_origins` on commas into a clean list
(used by `main.py`'s CORS middleware).

**L121-130** `_apply_llm_target` — `@model_validator(mode="after")`, runs after env load.
**Logic:** look up the preset for `llm_target`; if found, for each preset field, set it **only
if the user didn't set it explicitly** (`self.model_fields_set` is the snapshot of env-provided
fields). So a preset fills gaps but an explicit `OLLAMA_*` env var always wins. Returns `self`.

**L133-135** `get_settings()` — `@lru_cache` so the `Settings()` object (and its `.env` read) is
built once per process and reused everywhere.

**L138-147** `if __name__ == "__main__"` **self-check** (the ponytail runnable check): asserts
that `lan50` resolves its base_url + deep model, that an explicit `ollama_base_url` overrides
the preset, and that an empty target is a no-op. Run `python -m mallory_engine.config` to
verify the switch logic didn't break.
