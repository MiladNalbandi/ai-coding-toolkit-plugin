# Database Concerns in SDD

The user often cares specifically about data-backed features. This file covers the
database-facing parts of the loop. It is language-agnostic; substitute your ORM, migration
tool, and test runner from `references/stack-mapping.md`.

## The data-access layer is isolated

Only the repository / data-access layer touches the database. Entry points, use-cases, and
serializers must not issue queries directly. Enforce this with an architecture test:
"the web layer must not import the ORM." This keeps the loop's layering honest and makes
the database swappable in tests.

## Schema and migrations come first in implementation

In step 5 the schema/migration is the first artefact. Rules:

- **Migrations are forward-only and reviewed.** Treat a merged migration like an ADR —
  superseded by a new migration, not edited.
- **Expand/contract for production tables.** Add columns nullable, backfill, then enforce
  constraints in a later migration — never a single destructive change on a hot table.
- **Index intentionally.** If the spec implies a lookup or sort, the migration adds the
  index, and an ADR records why (composite order, partial index, GIN/GIST, etc.).
- **Constraints encode domain rules.** A domain invariant from the spec ("a published item
  has a published_at") should be a DB constraint *and* an application check, not just one.

## Transaction boundaries belong to the use-case

A use-case that changes more than one row wraps its work in a transaction. The boundary is
the use-case, not the entry point and not the repository method. On success it may emit a
domain event; on a domain violation it throws a domain exception (a typed error), never a
generic one, so callers and tests can distinguish failure modes.

## Validation that depends on the database

Uniqueness, foreign-key existence, and "no overlapping range" are validation rules (step 4)
*and* security-relevant (mass assignment, enumeration). Represent them in the validator,
the contract, and the tests. Prefer a DB constraint as the last line of defense so a race
cannot violate the invariant even if the application check passes.

## Tests and the database

- **Feature tests run against a real database** of the same engine as production where the
  feature uses engine-specific behavior (full-text search, JSON operators, window
  functions). Do not test Postgres-specific SQL against SQLite.
- **Gate engine-specific tests** so the suite still runs elsewhere, e.g. skip when the
  driver is not the target engine, with a clear reason.
- **Smoke test asserts the database is reachable** and one real read/write path works.
- Use factories/fixtures for setup; assert at most one database fact per feature test
  (the rest is asserted through the response/contract).

## Search and other engine features

When a feature needs search, the cheapest correct option is often the database you already
run (e.g. a generated full-text column + index, queried through the data-access layer)
rather than a new search service. Record the choice and its tradeoffs in an ADR:
zero new infrastructure and strong consistency on one side; weaker relevance and
single-language defaults on the other. Introduce a dedicated search engine only when the
spec's relevance or scale requirements actually demand it — and write the ADR that says so.

## Connection and config

- Connection details come from environment/config, never hardcoded; the spec's security
  section should note any read-replica, pooling, or least-privilege requirements.
- Long-running or batch jobs that hold connections belong in the async layer, not the
  request path; the spec should say so under domain rules or out-of-scope.
- Errors from the database must not leak SQL, schema, or connection strings into responses
  (security checklist, error handling).
