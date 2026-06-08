# Spec NNN — <Feature name>

* **Status:** Draft | Frozen | Superseded by NNN
* **Date:** YYYY-MM-DD
* **Contract:** docs/api/openapi.yaml#<operationId>  (or: "no API contract impact")

## Goal

<One sentence describing the problem being solved.>

## User stories

- As a <role>, I want <capability>, so that <benefit>.
- As a <role>, I want <capability>, so that <benefit>.

## Acceptance criteria

Numbered, observable, testable, specific. Tests reference these IDs.

```
AC-001 <observable outcome>
AC-002 <observable outcome>
AC-003 <invalid input> returns <status/error>.
AC-004 <unauthorized actor> cannot <action>.
AC-005 A user cannot <act on> another user's <resource>.
AC-006 Unauthorized access returns <status> and reveals no private data.
AC-007 Responses do not expose internal-only fields (<list them>).
```

## Endpoints   <!-- HTTP-facing features only; delete otherwise -->

| Method | Path                | Request body          | Success      | Errors            |
|--------|---------------------|-----------------------|--------------|-------------------|
| POST   | /api/v1/<resource>  | <CreateRequest>       | 201 <Resource>| 401, 403, 422    |

## Validation rules

| Field      | Required | Type   | Format / range            | Notes                       |
|------------|----------|--------|---------------------------|-----------------------------|
| <field>    | yes/no   | <type> | <email/uuid/date/len/...> | <enum, uniqueness, cross-field> |

Failure response: <status code> with <error shape>.

## Authorization rules

- Who may perform this: <roles / ownership condition>.
- Who may not: <explicitly>.
- Ownership: <users act only on their own records? admins exempt?>.

## Domain rules

- <Business invariant, e.g. "a draft has no published_at">.
- <Invariant>.

## Edge cases

- <Failure mode the implementation must handle>.
- <Concurrency / duplicate / not-found case>.

## Security concerns

- <Abuse case>.
- <Data exposure risk and the field(s) involved>.
- <Mass-assignment / rate-limiting / audit need>.

## Out of scope

- <Explicitly excluded this iteration>.

---

<!-- Append dated amendment blocks below once the spec is frozen. Never edit above. -->

## Amendment — YYYY-MM-DD

`AC-00X` changed to <…>.
Reason: <…>.
