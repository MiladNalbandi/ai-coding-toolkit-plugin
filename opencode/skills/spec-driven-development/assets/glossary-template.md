# Glossary

Terms used consistently across specs, ADRs, and code. Keep this list small and current;
when a term in a spec is ambiguous, define it here rather than re-explaining it each time.

| Term         | Meaning in this repo                                                  |
|--------------|----------------------------------------------------------------------|
| **Action**   | Single-purpose unit that performs one state change. (a.k.a. use-case) |
| **Use-case** | Application operation orchestrating validation, domain, and data access. |
| **Spec**     | Markdown file in `docs/specs/` describing intended behavior.          |
| **Contract** | The API document (OpenAPI / proto / GraphQL); machine-readable spec.  |
| **ADR**      | Architecture Decision Record. Immutable. Superseded, never edited.    |
| **AC**       | Acceptance criterion; a numbered, testable outcome in a spec.         |
| **Repository**| The only layer that reads/writes the database.                       |
| **Serializer**| The only layer that shapes data for responses.                       |
| **Smoke test**| Small, fast check that the app boots and the happy path works.       |
| **<DomainTerm>** | <Project-specific meaning, e.g. "Draft: a Post with status=draft and published_at IS NULL">. |

<!-- Add the domain nouns that recur in this project's specs. Define each once, here. -->
