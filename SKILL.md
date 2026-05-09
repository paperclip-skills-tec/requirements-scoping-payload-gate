---
name: requirements-scoping-payload-gate
description: "Requirements-phase gate that ensures scope documents and acceptance criteria for client components consuming server endpoints include a payload verification plan before implementation begins. Invoke when writing or reviewing acceptance criteria for any feature that adds or modifies a client component that fetches from a server endpoint — wizards, pickers, list-fetches, table fetches, form submissions — even when the shape looks obvious. Also invoke when reviewing a scope doc, feature spec, or issue description for completeness before handing off to a developer. Complements payload-shape-gate (the implementation-phase counterpart) by shifting the check upstream to requirements, so developers never start coding a fetch flow without a clear verification approach already agreed upon. The shift-left motivation is TEC-1001 PR #16: column-alias mismatches silently pass unit tests and are only caught late in QA or in production."
---

# Requirements-Phase Payload Verification Gate

## What this gate requires

Any scope document or issue description for a feature that adds or modifies a client
component fetching from a server endpoint **must** include a **Payload Verification Plan**
section that names, at minimum:

1. Each endpoint being consumed and the client component that consumes it.
2. Which verification mode will be used — **A**, **B**, or **C** (see Step 2 below).
3. A one-line rationale if mode A is not chosen.

The plan does not need to be complete code — it is a **commitment** made at scope time so
the developer and reviewer know what is expected before a single line of implementation
is written.

---

## Step 1 — Identify client-server interactions in the scope

Read the feature description and acceptance criteria. For each interaction, fill in one row:

| Flow type | Client component | Endpoint | New or modified? |
|-----------|-----------------|----------|-----------------|
| picker    | …               | GET /api/… | new / modified |
| list-fetch| …               | GET /api/… | new / modified |
| wizard    | …               | POST/GET /api/… | new / modified |
| table     | …               | GET /api/… | new / modified |

If a fetch flow is **unchanged** and already has passing verification, mark it "existing —
no new plan needed" and skip it.

---

## Step 2 — Choose the verification mode

Work top-down; commit to the first mode that is feasible given the project environment:

```
Can the developer import the Express app and run tests against the real DB?
  └─ YES → Mode A: Contract test  (preferred — highest confidence)

Can the developer start the server and capture a real HTTP response?
  └─ YES → Mode B: Captured fixture  (acceptable)

Can the developer run the full app in a browser against a live server?
  └─ YES → Mode C: Playwright smoke  (acceptable)

None of the above → Mode BLOCKED: name the blocker in the plan so it surfaces
  before implementation starts, not during in_review.
```

**Never plan for hand-rolled mock shapes.** A fixture invented from memory has the same
confidence gap as a pure unit test — it only proves the developer's assumptions match
themselves. If the plan says "we'll just mock the response", flag it as incomplete.

---

## Step 3 — Write the Payload Verification Plan block

Paste this template into the scope document's acceptance criteria section and fill it in:

```markdown
## Payload Verification Plan

| Flow | Client component | Endpoint | Verification mode |
|------|-----------------|----------|------------------|
| [e.g. list-fetch] | [e.g. WidgetList.jsx] | GET /api/widgets | Mode A — contract test |

**Mode rationale** (required if not A): [e.g. "shared DB — will capture fixture instead"]

**Shape fields to verify**: [list every field the client destructures, e.g. id, label, status]

**Gate status**: PLAN COMPLETE / PLAN INCOMPLETE — [reason]
```

### Gate status guidance

| Status | Meaning |
|--------|---------|
| `PLAN COMPLETE` | All fetch flows identified, mode chosen, shape fields listed. Developer can start. |
| `PLAN INCOMPLETE` | Missing flow, no mode chosen, or "hand-roll" planned. Resolve before handing off. |

A `PLAN INCOMPLETE` status should block handoff to development, the same way a `FAIL`
from `payload-shape-gate` blocks `in_review`.

---

## Reviewing an existing scope doc

When the scope doc already exists and you are reviewing it, check each criterion:

- [ ] Every fetch flow in the feature is listed (not just the "main" one — side panels, autocompletes, and eager-loading calls count)
- [ ] A verification mode is named for each flow (not "TBD")
- [ ] Shape fields are identified — at minimum the top-level keys the client component destructures
- [ ] Mode A is used, or there is a one-line rationale for B/C
- [ ] No flow plans to use a hand-rolled shape

If any box is unchecked, add a comment to the scope doc or issue explaining what is missing
and mark the gate as `PLAN INCOMPLETE`. Do not approve the scope for implementation until
all boxes are checked.

---

## Relationship to payload-shape-gate

This skill and `payload-shape-gate` are two phases of the same safety net:

| Phase | Skill | When it runs | What it checks |
|-------|-------|--------------|----------------|
| Requirements | `requirements-scoping-payload-gate` (this skill) | Scope writing / review | Plan named in acceptance criteria |
| Implementation | `payload-shape-gate` | Before `in_review` | Plan executed: real handler used, test passes |

A scope doc that passes this gate makes the developer's `payload-shape-gate` checklist
straightforward — they already know which mode to implement.

---

## Quick-reference

| I am… | What to do |
|-------|-----------|
| Writing acceptance criteria for a new fetch flow | Add a Payload Verification Plan block (Step 3) |
| Reviewing a scope doc before handoff | Run the review checklist above |
| Unsure whether a flow counts | If the client renders data from a server endpoint, it counts |
| The scope says "mock the API for tests" | Flag as incomplete — hand-rolled shapes don't satisfy this gate |

---

*TEC Custom Skill — maintained by the Deltek Technical Services Engineering team.*
