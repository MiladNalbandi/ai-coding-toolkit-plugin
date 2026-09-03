# Diagrams — the shared ASCII vocabulary

> **Lead with the picture.** A plan, a spec, or a migration that opens with a diagram gets
> read; one that opens with three paragraphs gets skimmed. Every skill in this toolkit
> draws with the same six templates and the same markers, so a reader learns the notation
> once.

## Rules

- **Max 80 columns** for any diagram; 72 for side-by-side blocks. Wider wraps in a
  terminal, in a PR comment, and inside a nested list.
- ASCII and box-drawing characters only. No images, no Mermaid — the destinations are
  terminals and plain markdown files.
- **Fixed change markers**, everywhere: `+` added · `-` dropped · `~` changed ·
  `←` annotation · `→` flow.
- Fixed status markers: `✔` done · `▶` current · `⬜` not started · `✘` failed ·
  `◼` warning · `⏳` pending.
- The diagram goes **above** the prose that explains it.
- Where a diagram is required and nothing structural changes, write `No structural change`.
  Never leave the section blank and never skip it silently.
- Label real values — real column types, real row counts, real file paths. A diagram with
  placeholder nouns (`TableA`, `ServiceB`) is worth less than the prose it replaced.

---

## T1 — Table before / after

Required for every schema-touching change (`database-migrations` D1, SDD Step 1 and 2.5,
clarify-loop § 2.5).

```
BEFORE                          AFTER
┌────────────────────────┐      ┌──────────────────────────────────┐
│ posts                  │      │ posts                            │
├────────────────────────┤      ├──────────────────────────────────┤
│ id         bigint PK   │      │ id            bigint PK          │
│ title      varchar(255)│ ═══> │ title         varchar(255)       │
│ body       text        │      │ body          text               │
│ created_at timestamptz │      │ created_at    timestamptz        │
└────────────────────────┘      │+author_id     bigint NULL        │ ← new
                                │+published_at  timestamptz NULL   │ ← new
                                │~title         varchar(255)→text  │ ← widened
                                │-legacy_slug                      │ ← drop M4
                                └──────────────────────────────────┘
  fks:     + author_id → users.id (NOT VALID, validated in M3)
  indexes: + posts_author_id_idx    (btree, author_id)
           + posts_published_idx    (btree, published_at)
                                    WHERE published_at IS NOT NULL
  rows: ~2.4M · est. lock: none (every step online)
```

The footer lines are part of the template, not optional: **fks**, **indexes**, **rows**
(actual count, from the database — see `mcp-toolkit`), and the **estimated lock** per the
migration plan. A new table draws only the AFTER box, with `BEFORE: (does not exist)`.

## T2 — Relationships

```
users ──1──<∞ posts ──1──<∞ comments
  │                            │
  └──────────1──<∞ ────────────┘   (comments.author_id → users.id)
```

`──1──<∞` reads "one to many". Draw only the tables the change touches plus their
immediate neighbours; a whole-schema ERD is noise.

## T3 — Migration timeline (expand / contract)

One column per deploy step. Required whenever a change is not purely additive.

```
 M1 expand        M2 backfill       M3 enforce        M4 contract
 ──────────       ───────────       ──────────        ───────────
 add col NULL  →  fill in batches → NOT NULL + FK  →  drop old col
 deploy A         job (idempotent)  deploy B          deploy C
 reversible       reversible        reversible        IRREVERSIBLE
 lock: none       lock: none        lock: brief       lock: brief
                                    needs A live      ≥1 release after B
```

Every column states: what runs, where it runs, whether it reverses, and the lock it takes.
An irreversible step is spelled `IRREVERSIBLE` in capitals — that word is the point.

## T4 — Request / layer flow

Required in SDD Step 7 (the changed path), Workflow 1 design, and once per option in the
architecture workflow.

```
POST /api/posts
  → route → handler(thin) → validator → policy → use-case ──┐
                                                   │        │
                                       repository ─┘   domain event
                                            │
                                        postgres
```

Mark the layers the change actually touches with `*`, so a reviewer sees the blast radius:
`→ validator* → policy → use-case*`.

## T5 — Plan / file map

Required above every plan-approval gate (PBV Step 1) and for the agent→file assignment in
any parallel build, where it doubles as the conflict check.

```
 T-01 schema      database/migrations/2026_09_03_add_author.php   [new]
 T-02 model       app/Models/Post.php                             [edit]
 T-03 validator   app/Http/Requests/StorePost.php                 [new]
 T-04 use-case    app/Actions/CreatePost.php                      [new]
 T-05 handler     app/Http/Controllers/PostController.php         [edit]
      └─ T-04 blocked by T-02 · T-05 blocked by T-03,T-04
```

One file per row. If a path appears twice, those two tasks cannot run in parallel — that
is what this diagram is for.

## T6 — AC progress board

Printed by the AC loop at every gate (see `ac-loop.md`).

```
  AC-001 ✔ test+feat   AC-004 ⬜             AC-007 ⬜
  AC-002 ✔ test+feat   AC-005 ⬜             AC-008 ⬜
  AC-003 ▶ at gate     AC-006 ⬜             AC-009 ⬜
  ────────────────────────────────────────────────────
  2/9 done · 41 tests green · 0 skipped
```

---

## Where each template is required

| Skill / step | Template |
|--------------|----------|
| `clarify-loop` § 2.5 Data & migration impact | T1 (when data scope ≠ "no schema change") |
| SDD Step 1 — spec, *Data model* section | T1 + T2 |
| SDD Step 2.5 — data design | T1 + T3 |
| SDD Step 5 — the AC loop gate | T6 |
| SDD Step 7 — review | T4 for the changed path |
| Workflow 1 Step 3 — design | T4, or T5 for parallel mode |
| Workflow 3 — architecture | one T4 per option, side by side |
| Workflow 5 Step 1 — plan (above the gate) | T5 |
| Workflow 5 Step 3 — agent assignment | T5 |
| `database-migrations` D1 / D3 | T1 + T2 + T3 |

## Checking width

```
awk 'length > 80 { print FILENAME ":" NR " (" length ")" }' <file>
```

`awk` counts bytes, so box-drawing characters inflate the count. For an exact character
count use `python3 -c "…len(line)…"` or `wc -L` on a UTF-8-aware system.
