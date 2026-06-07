# Spec NNN — <UI feature name>

* **Status:** Draft | Frozen | Superseded by NNN
* **Date:** YYYY-MM-DD
* **Consumes contract:** docs/api/openapi.yaml#<operationId>  (or: "no API calls")
* **Design:** <link to design file / Storybook story, if any>

## Goal

<One sentence: what the user is trying to accomplish on screen.>

## User stories

- As a <role>, I want <on-screen capability>, so that <benefit>.

## Acceptance criteria

Numbered, observable from the rendered UI, testable. Tests reference these IDs.

```
AC-001 <Primary happy path: what the user sees/does and the result.>
AC-002 The Submit action is disabled while the request is in flight.
AC-003 Invalid <field> shows a field-level error and does not submit.
AC-004 The whole flow is operable with the keyboard alone.
AC-005 Unauthorized/expired session redirects to <…> rather than erroring.
AC-006 No internal-only fields from the API are rendered.
```

## UI states   <!-- enumerate ALL; unspecified states cause most frontend bugs -->

| State              | What the user sees                                  |
|--------------------|-----------------------------------------------------|
| Loading            | <skeleton / spinner / disabled controls>            |
| Empty              | <empty-state message + primary action>              |
| Error (validation) | <field-level errors from the 422 response>          |
| Error (server)     | <non-blocking error UI for 5xx; retry affordance>   |
| Partial / optimistic | <what shows before the server confirms, if any>   |
| Success            | <confirmation / navigation / updated list>          |

## Data dependencies

| Reads / writes              | Endpoint (operationId)        | On failure shows |
|-----------------------------|-------------------------------|------------------|
| <what data this screen needs| <createResource / listItems>  | <which state>    |

## Validation rules (client mirrors server)

| Field   | Required | Type   | Format / range            | Client message            |
|---------|----------|--------|---------------------------|---------------------------|
| <field> | yes/no   | <type> | <must match server rule>  | <user-facing message>     |

Client validation is for UX only; the server remains the trusted boundary.

## Accessibility

- Keyboard: <tab order, shortcuts, Enter/Escape behavior>.
- Focus: <where focus goes on load, after submit, on error, on modal open/close>.
- Labels/roles: <each control's accessible name; live region for async results>.
- Contrast & motion: <meets contrast target; respects reduced-motion>.

## Responsive behavior

| Breakpoint | Layout change                                |
|------------|----------------------------------------------|
| <mobile>   | <stacked / hidden nav / etc.>                |
| <desktop>  | <two-column / etc.>                          |

## Security concerns

- <Untrusted content rendered? how is it escaped/sanitized?>
- <Anything the client must NOT hold (secrets, privileged tokens, internal URLs)?>
- <Auth/CSRF expectations for these calls.>

## Out of scope

- <Explicitly excluded this iteration.>

---

<!-- Append dated amendment blocks once frozen. Never edit above. -->
