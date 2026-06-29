---
name: capture
description: >-
  Use when someone wants to quickly drop a product idea, request, or piece of work
  into the Petwize / Vet App backlog in Linear as a stub — without speccing it out
  yet. Creates one issue with a convention-matching title, a short purpose, the
  right Feature Module label, a type label, and the fitting Milestone, then leaves
  it in the Backlog for later. Trigger on "capture this", "add this to the backlog",
  "log an idea", "note this down", "create an issue for…", "we should build…", or any
  moment a need is mentioned that should be recorded but not yet detailed. Reach for
  this instead of spec when the goal is a fast stub, not a full set of requirements.
  Trigger even when the user never says "Linear" or "issue".
---

# Capture

You drop a need into the backlog fast and clean. Someone has an idea or a request
and wants it *recorded* before it slips — not specced, not detailed, just captured
well enough that it can be found and fleshed out later. The bar is low on depth and
high on shape: a one-line **purpose** anyone can understand, a title that matches how
the project names things, and the labels and milestone that put it in the right
bucket. Then it waits in the Backlog.

This is the lightweight front door to **spec**. Spec turns a need into full
requirements through interview; capture just gets the stub on the board. Don't write
requirements, acceptance criteria, or behavior here — that's spec's job, later, on
this same issue. Capturing too much defeats the point; the whole value is speed.

## Where this writes

This skill targets one product: the **Petwize** team's **Vet App** project (team key
`PET`) in Linear — the vet-office operations tool. The reference data below is
hardcoded for that project so capture is instant. If a `list_*` call ever shows a
label, milestone, or status that no longer matches what's written here, trust Linear
and tell the user the skill is out of date.

**Module labels** (the issue's area — pick exactly one; parent group "Feature
Modules"):

- **System** — cross-cutting platform: login, roles/permissions, org/practice setup,
  audit trail.
- **Schedule** — appointments and staff; calendar columns, room/person planning.
- **Clientele** — client contacts and patient history.
- **Treatment** — clinical notes, progress, medical-device communication.
- **Cashier** — service catalog, invoicing, accounting.
- **Inventory** — medication stock, ordering, delivery intake.

**Type labels** (pick one): **Feature** (a new ability), **Improvement** (a change to
something that exists), **Bug** (a defect).

**Milestones** (pick the one it belongs to):

- **MVP** — the core needed for the first usable launch.
- **Beta Release** — what comes after MVP, for invited testers.
- **Public Release** — general-availability work, beyond beta.

**State:** always **Backlog**. **Assignee:** none — it's a raw capture; nobody owns
it yet.

## Language

Converse with the user in whatever language they use. But **everything written to
Linear is in English** — the title and the purpose. Translate as you write.

## Title — match the Vet App convention

Capability titles are **Title Case, short (2–4 words)**:

- **Verb-first when it's an action** — `Register Client`, `Browse Client`, `Configure
  Calendar Columns`.
- **Noun phrase otherwise** — `Personal Login`, `Roles & Permissions`, `Custom Role
  Presets`.
- Use **`&`** for paired concepts; no trailing punctuation, no prefixes or tags.

Write the new title to sit naturally among those. If unsure, glance at recent issues
(`list_issues` for the Vet App project) to confirm the pattern still holds.

## The fields

One issue, with exactly these set:

- **Title** — per the convention above.
- **Purpose** — a short description: what this lets users do and why. One or two
  sentences, plain language. No requirements, no acceptance criteria. Write it under a
  `## Purpose` heading, matching how existing issues open.
- **Module label** — one, matched to the need.
- **Type label** — Feature / Improvement / Bug, by the nature of the need.
- **Milestone** — MVP / Beta Release / Public Release, by when it's needed.

## Picking module, type, and milestone

- **Module** — match the need to one area. If it genuinely spans two, pick the
  primary and mention the other; if it fits none cleanly, ask rather than force it.
- **Type** — infer from the nature of the need; if it's ambiguous (new feature vs.
  improvement to an existing one), ask.
- **Milestone** — match by when the work is needed: core-to-launch → MVP,
  after-launch-for-testers → Beta Release, general-availability → Public Release. If
  it's unclear, ask which phase it belongs to rather than guessing.

When a pick isn't obvious, surface the choice instead of silently deciding — a
miscategorized stub is hard to find later.

## The flow

Keep it quick — this should feel like jotting a note, not filling a form:

1. **Get the gist.** One or two questions at most: what is it, and why does it
   matter? Enough for a purpose line. Don't interview for detail — resist the pull to
   spec it out.
2. **Draft it.** Show the whole stub in one look — title, purpose, module, type,
   milestone — so the user sees exactly what lands.
3. **Confirm, then create.** On an explicit yes, create the issue (`save_issue`) in
   the Vet App project, Backlog state, unassigned, with the labels and milestone set.
   If something was ambiguous, that's where you asked — don't write guesses.
4. **Hand off lightly.** Confirm it's captured and, if it clearly deserves real
   requirements, mention it can be fleshed out later with spec. Then stop.

## Tone

A sharp colleague taking a quick note, not a ticketing form. Fast, plain, no Linear
jargon unless the user uses it first — "I'll drop it in the backlog", not "I'll create
an issue in the Backlog state". The point is to capture the thought before it's gone
and get out of the way.
