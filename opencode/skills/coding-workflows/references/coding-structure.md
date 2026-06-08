# Coding Structure — how to lay out and order implementation

Use this whenever a workflow says "implement" (Workflow 1 Step 4, Workflow 5 Step 3).
It keeps every change consistent and reviewable. It is the lightweight twin of
`spec-driven-development` Step 5 — if you need the full rigor (contract-first, frozen
spec), switch to that skill.

## The layer order (build in this sequence)

Implement bottom-up so each layer can be tested before the next depends on it:

```
schema / migration
  → model / entity
    → test factory / fixture
      → authorization rule
        → use-case / action (business logic)
          → input validator
            → output serializer / DTO
              → handler / endpoint
                → route registration
```

**Why this order:** data shape is decided first; business logic sits in the middle and
stays framework-agnostic; the handler/route is the *thinnest* layer at the end — it only
wires HTTP to the use-case. Never put business rules in the handler.

## File map (adjust names to the project's conventions)

| Layer | Typical location |
|-------|------------------|
| Schema / Migration | `database/migrations/NNN_*.sql` or ORM migration |
| Model / Entity | `src/models/FeatureName.*` |
| Validator / Input | `src/validators/featureName.*` |
| Authorization | middleware / policy / guard layer |
| Use-case / Action | `src/actions/FeatureName.*` |
| Serializer / DTO | `src/serializers/featureName.*` |
| Handler + Route | `src/handlers/featureName.*` + route registration |

> Per-language idioms (Laravel, Django, FastAPI, Go, Node, Rust) live in
> `spec-driven-development/references/stack-mapping.md`.

## Placement rules

- **Authentication / authorization** → middleware, policy, or guard — never inside the handler body.
- **Input validation** → a dedicated validator layer, before the use-case runs.
- **Business rules** → the use-case/action. Keep them out of the handler and the model.
- **Output shaping** → a serializer/DTO. Never return raw model objects; decide what fields leave the boundary.
- **Entry points stay thin** — a handler parses input, calls one use-case, serializes the result. That's it.

## Sequential vs parallel build

Ask first (default: sequential for small changes):

1. **Sequential** — one layer at a time in this context. Safe, simple, easy to review.
2. **Parallel** — one file per agent via the `multi-agent` skill. Faster; needs layer isolation.

**Conflict rule for parallel:** list every file each agent will write. If a file appears
twice, serialize those two agents. Never let two agents share a file. After agents return:
merge → wire imports (handler imports action, action imports validator) → run tests → fix
inline (don't re-spawn for small fixes).

## YAGNI is load-bearing

Add **no** field, endpoint, abstraction, helper, permission, or behavior that the
requirements / ACs did not ask for. The structure above is a skeleton to fill only as far
as the task demands — not a checklist to max out.
