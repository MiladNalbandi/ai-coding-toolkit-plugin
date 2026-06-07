# Coding Conventions

Single source of truth for *how* code is written here. Short on purpose — anything not
listed is whatever the formatter, refactorer, and static analyzer say. Fill the
`<...>` placeholders with this project's real language, version, and tools; delete rows
that do not apply.

## Language

- **<Language> <minimum version>+** features only. <List the modern features the project
  freely uses, e.g. readonly classes / typed enums; dataclasses / type hints; generics.>
- Strictest type and lint settings the language offers, applied uniformly across the repo.
- Public function/method signatures are fully typed. No untyped parameters or returns.
- Prefer immutability and "closed for extension" by default (final classes / frozen
  dataclasses / unexported structs) for types not designed to be extended.

## Layering

```
        ┌─────────────────────────────┐
request →│ entry point (thin)          │← routes
         │  ├─ input validation        │
         │  ├─ authorization           │
         │  ├─ use-case / action ──────┼─→ service ─→ repository (data access)
         │  └─ output serializer       │
         └─────────────────────────────┘
                                          events ↔ listeners ↔ jobs ↔ async
```

Hard rules (enforced by architecture tests):

1. **Entry points do not access the database directly.** They invoke a use-case.
2. **Use-cases are single-purpose**, named with a verb, wrap multi-row changes in a
   transaction, and may emit domain events.
3. **The data-access layer is the only place the ORM/DB is touched.**
4. **Output serializers are the only types allowed to shape data for responses.**
5. **Authorization gates every write path.** Entry points call the authorization check.
6. **Validators own input validation;** entry points never validate inline.

## Naming

| Kind             | Pattern               | Example                  |
|------------------|-----------------------|--------------------------|
| Use-case/action  | `VerbNoun`            | `PublishPost`            |
| Input validator  | `VerbResourceRequest` | `StorePostRequest`       |
| Output serializer| `ModelResource`/`DTO` | `PostResource`           |
| Authorization    | `ModelPolicy`/guard   | `PostPolicy`             |
| Event            | `NounPastTense`       | `PostPublished`          |
| Listener         | `VerbNoun`            | `NotifySubscribers`      |
| Async job        | `VerbNoun`            | `RebuildSearchIndex`     |
| Migration        | `verb_noun`           | `create_posts_table`     |
| Route name       | `resource.action`     | `posts.show`             |
| Test file        | `<Behaviour>Test`     | `RegisterUserTest`       |

## Tests

- One behavior per test. No "test everything" blocks.
- Parameterize validation tests with a dataset/table.
- Feature tests assert status, response shape, and at most one database fact.
- Unit tests construct dependencies directly — no container/framework magic in unit scope.
- Name tests after acceptance criteria where practical (`ac_003_...`).

## Comments

Comments are exceptions, not the rule. Write one only when **why** is non-obvious — a
workaround, an upstream bug, a surprising domain rule. Never explain *what*; names do that.

## Commits

`type(scope): subject` — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`. The body
answers **why**. If a commit closes an acceptance criterion, reference the spec:
`Refs: docs/specs/002-posts.md#AC-3`.
