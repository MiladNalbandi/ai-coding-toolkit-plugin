# Frontend in SDD

The loop does not change for frontend work — spec → contract → failing tests →
validation/security → implement → refactor → review/smoke → ADR. What changes is the
roles each step produces, the test layers, and three concerns that are first-class on the
client and nearly absent on the server. Read this alongside `references/workflow.md`.

Applies to both **SPA/component** frontends (React, Vue, Svelte, Angular, SolidJS) and
**server-rendered** views (Blade, Django templates, Rails ERB, Razor). The differences are
called out where they matter.

## The seam: the frontend consumes the contract, it does not own one

There is no second OpenAPI document. The frontend is a **consumer** of the backend's
contract (`docs/api/openapi.yaml`). The discipline is:

- Generate a typed API client from the contract (e.g. `openapi-typescript`,
  `orval`, `openapi-generator`) rather than hand-writing fetch calls and types.
- When the contract changes, regenerate; type errors then point at every call site that
  must adapt. This is how the two sides stay honest.
- The frontend may have its own **component contract** — the prop/event API of a
  component, captured as types plus stories (Storybook) — but that is internal, not the
  network contract.

If the frontend lives in the same repo, it shares `docs/specs/` and `docs/adr/` with the
backend; only its tests and toolchain are separate. If it is a separate repo, it still
references the backend's contract by version.

## Three concerns the spec must pin down (frontend-only)

A UI spec's acceptance criteria must cover, beyond the usual:

1. **Every UI state.** Enumerate loading, empty, error, partial/optimistic, and success.
   "What does the screen show while the request is in flight / when the list is empty /
   when the API returns 422 / when it returns 500?" Unspecified states are the number-one
   source of frontend bugs.
2. **Accessibility.** Keyboard reachability and order, visible focus, focus management on
   route/modal changes, labels and roles for assistive tech, color contrast, and reduced-
   motion respect. Write these as ACs (`AC-0xx Submitting with the keyboard alone works`).
3. **Responsive behavior.** Which breakpoints, and what changes at each.

Use `assets/component-spec-template.md` — it is the feature spec with these sections built
in.

## Frontend roles (the layering)

Honor the role; use the framework's idiom. The point is the same as on the backend:
keep the thing that renders thin, and push data, state, and rules out of it.

| Abstract role          | React                       | Vue                      | Svelte                  | Angular                  | Server-rendered          |
|------------------------|-----------------------------|--------------------------|-------------------------|--------------------------|--------------------------|
| Route / page           | route component / page      | page component + router  | `+page.svelte`          | routed component         | controller + template    |
| Data / container       | hook + query (TanStack)     | composable + Pinia       | load fn + store         | service + resolver       | view-model / presenter   |
| Presentational unit    | pure function component     | `<script setup>` comp.   | component               | dumb component           | partial / include        |
| Client state           | context / store (Zustand)   | Pinia store              | store                   | service / signal store   | session / flash          |
| API client             | generated client            | generated client         | generated client        | generated client         | n/a (server calls direct)|
| Input validation (UX)  | zod/yup schema + RHF        | vee-validate + zod       | schema + form lib       | reactive forms validators| HTML attrs + server echo |
| Output / formatting    | view helpers / formatters   | filters / computed       | derived stores          | pipes                    | template helpers         |
| Design tokens          | CSS vars / theme            | CSS vars / theme         | CSS vars / theme        | CSS vars / theme         | CSS vars / theme         |

Keep presentational components pure: data in via props, intent out via events/callbacks.
Data fetching, mutations, and business decisions live in the container/data layer, the way
use-cases hold them on the backend.

## Failing tests — frontend layers

| Layer            | Tool examples                          | Purpose                                            |
|------------------|----------------------------------------|----------------------------------------------------|
| Component/unit   | Vitest/Jest + Testing Library          | Render from props; logic in hooks/composables.     |
| Interaction      | Testing Library `user-event`           | Real user flows: type, click, submit, tab.         |
| Accessibility    | `jest-axe` / axe in Playwright         | No violations; focus and roles correct.            |
| End-to-end       | Playwright / Cypress                   | Whole path through a real (or mocked) backend.     |
| Visual regression| Playwright snapshots / Chromatic       | Catch unintended visual change.                    |
| Contract / mocks | MSW seeded from the OpenAPI client     | Frontend tests run against contract-shaped mocks.  |

Red-first still holds: write the interaction test against the spec's ACs before the
component exists. Test by visible behavior and accessible roles ("the user sees an error
message", "the Submit button is disabled while saving"), not by implementation details
like internal state or CSS classes. Query by role/label so the test doubles as an a11y
check.

## Validation and security on the client

- **Client validation mirrors the server's rules for UX, never replaces them.** The
  server (and its contract) remains the only trusted boundary. Derive the client schema
  from the same rules the spec lists; do not invent stricter or looser ones.
- **Output escaping / XSS.** Render untrusted content through the framework's escaping;
  treat any `dangerouslySetInnerHTML` / `v-html` / `{@html}` as a reviewed exception that
  needs a sanitizer and ideally an ADR.
- **No secrets in the bundle.** API keys, tokens for privileged operations, and internal
  URLs do not ship to the client. The spec's security section says what the client may
  hold (e.g. a short-lived access token in memory) and what it may not.
- **Auth state is handled explicitly.** Unauthenticated and expired-session states are UI
  states with their own ACs (redirect, re-auth prompt), not crashes.
- **CSRF / same-site** for cookie-based auth, per the backend's scheme.

## Implementation order (frontend)

```
generate/refresh API client from contract
→ client state model (store/signals)
→ data layer (queries/mutations, loading+error handling)
→ presentational component(s)
→ form schema + wiring (if input)
→ styling via design tokens
→ route/page registration
```

Stop when the spec's ACs — including every enumerated UI state and a11y criterion — are
green. YAGNI applies: no speculative props, no "flexible" config a screen does not need.

## Refactor — frontend toolchain

Run formatter → linter → type check → tests (see `references/stack-mapping.md` for the
frontend row). Add an accessibility lint (`eslint-plugin-jsx-a11y` or framework
equivalent) and run the visual-regression suite. Public component APIs and the consumed
contract must not change here unless the spec/contract changed first.

## Review and smoke — frontend

Review confirms: every UI state from the spec is implemented and tested; a11y ACs pass;
the component reads data through the generated client (no ad-hoc fetch with hand-typed
shapes); presentational components are pure; no secrets in the bundle; untrusted content
is escaped. Smoke: the page mounts, the route resolves, the happy path renders against the
real or mocked API, one failure path shows its error state, keyboard navigation reaches
the primary action, and the console is clean (no errors/warnings).

## ADRs that are specifically frontend

State management library; styling approach (CSS modules vs utility CSS vs CSS-in-JS);
rendering strategy (CSR vs SSR vs SSG vs streaming); component library adoption;
form/validation library; data-fetching/caching strategy; the API-client generation tool.
Record the choice and its tradeoffs the same way as any other ADR.
