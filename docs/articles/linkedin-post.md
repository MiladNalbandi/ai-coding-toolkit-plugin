# LinkedIn post

> Suggested image: `images/ac-loop.png` (or `images/cover.png` for a plain text-card look).
> Carousel option: cover → ac-loop → gate-options → schema-diff → db-pipeline.

---

There's a specific moment where AI-assisted development goes wrong, and it's not the one
people expect.

It's not the agent writing bad code. Models write decent code now.

It's the moment the agent says "done" and hands you a diff across 14 files, 9 behaviors and
a schema change — and asks what you think.

You can't review that. You skim it, you spot two things, you approve, and the other seven
behaviors enter your codebase unread. The agent did the work. You didn't do the review,
whatever the approval says.

So in the next version of my open-source workflow toolkit, I changed one thing:

𝗧𝗵𝗲 𝘂𝗻𝗶𝘁 𝗼𝗳 𝘄𝗼𝗿𝗸 𝗶𝘀 𝗻𝗼 𝗹𝗼𝗻𝗴𝗲𝗿 𝘁𝗵𝗲 𝗳𝗲𝗮𝘁𝘂𝗿𝗲. 𝗜𝘁'𝘀 𝘁𝗵𝗲 𝗮𝗰𝗰𝗲𝗽𝘁𝗮𝗻𝗰𝗲 𝗰𝗿𝗶𝘁𝗲𝗿𝗶𝗼𝗻.

For each criterion, one at a time:

→ write its failing test, commit it: test(AC-003)
→ write the minimum code that passes, commit it: feat(AC-003)
→ stop at a gate

Two commits. Then it asks.

The gate is the part that took longest to get right. Seven options — approve, human review,
AI review, refactor while green, approve the rest, reject back to red, and the one I use
most:

📝 "Add my recommendation, then AI review on top."

You type a change. It lands as its own commit FIRST. Then the reviewer runs against the
amended diff. Order matters — otherwise the reviewer keeps critiquing code you already
decided to change, and you spend the review arguing with a ghost.

Four of the seven options loop back to the same question, so you can chain them:
AI review → apply → add my own note → review again → refactor → approve.

Same release, second change: schema migrations leave the code loop entirely and get their
own pipeline. It opens by drawing the before/after table diff — with the real row count
pulled from the database, because ADD COLUMN NOT NULL DEFAULT is trivial on 400 rows and an
outage on 40 million. Then: expand → backfill → enforce → contract, one migration per
deploy, every step with a real down() that gets rehearsed up → down → up before anything
touches a real database.

The pattern underneath both:

𝗠𝗮𝗸𝗲 𝘁𝗵𝗲 𝘂𝗻𝗶𝘁 𝗼𝗳 𝗿𝗲𝘃𝗶𝗲𝘄 𝘀𝗺𝗮𝗹𝗹 𝗲𝗻𝗼𝘂𝗴𝗵 𝘁𝗵𝗮𝘁 𝗮 𝗵𝘂𝗺𝗮𝗻 𝗰𝗮𝗻 𝗮𝗰𝘁𝘂𝗮𝗹𝗹𝘆 𝗱𝗼 𝗶𝘁 — 𝘁𝗵𝗲𝗻 𝗺𝗮𝗸𝗲 𝘁𝗵𝗲 𝘁𝗼𝗼𝗹 𝘀𝘁𝗼𝗽 𝘁𝗵𝗲𝗿𝗲.

Not "review more carefully". Cut the work into pieces the size of a decision, and put a real
decision point at the end of each one.

The agent is very good at doing the work. It's bad at knowing when you needed to look. That's
the part worth designing.

MIT licensed, works with Claude Code and opencode. Link in the comments.

#AI #SoftwareEngineering #DeveloperExperience #TDD #CodeReview #ClaudeCode #DatabaseMigrations

---

## First comment (post separately, so the link doesn't hurt reach)

Repo + the full write-up:
https://github.com/MiladNalbandi/ai-coding-toolkit-plugin

Curious about one thing in particular — if you use a coding agent daily: do you review per
feature, or have you already cut it smaller? And what made you switch?

---

## Shorter variant (if you want it under ~1,300 characters)

The agent says "done" and hands you 14 files, 9 behaviors and a schema change.

You can't review that. You skim it, spot two things, approve — and seven behaviors enter
your codebase unread.

So I changed the unit of work in my open-source toolkit: not the feature, the acceptance
criterion.

→ failing test → commit
→ minimum code → commit
→ stop at a gate

Two commits per criterion, then it asks. The gate has seven answers, and four of them loop
back to the same question — so you can chain "AI review it → apply → now add my note →
review again → refactor → approve" instead of a binary approve/reject.

Migrations get their own pipeline: draw the before/after table diff with the real row count
first, then expand → backfill → enforce → contract, every step with a down() that gets
rehearsed before it ever runs for real.

The pattern: make the unit of review small enough that a human can actually do it, then make
the tool stop there.

The agent is very good at doing the work. It's bad at knowing when you needed to look.

#AI #SoftwareEngineering #TDD #DeveloperExperience
