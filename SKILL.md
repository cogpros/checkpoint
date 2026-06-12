---
name: checkpoint
version: "0.2.0"
description: "Use when: (1) the user says 'checkpoint' or 'summary and checkpoint' mid-session, (2) a visible state pulse is needed without closing the session, (3) a natural break point needs 'where am I, what did I just do, what's next' answered in one beat. Composes an open-items sweep + next-step pick + session delta + deadline clock into an emoji-formatted dashboard. Ephemeral display only — no writes, no events, no session close. NOT for: end-of-session closure, thread compression, or one-off recall."
author: "Dustin Pollock"
license: "MIT"
tags: [orientation, session-state, dashboard, cognitive-prosthetics]
metadata:
  cogpros-primitive: "Checkpoint"
triggers:
  - "/checkpoint"
  - "summary and checkpoint"
  - "checkpoint"
---

# Checkpoint

Visible session state on demand. Where am I, what did I just do, what's bearing down, what's next — one beat.

## Why This Skill Exists

Long working sessions accumulate state faster than working memory holds it. "Where am I, what did I just do, what's next" is a question that fires constantly mid-session — and answering it usually means scrolling back, re-reading, or asking three separate questions.

`/checkpoint` is the renderer. It composes existing orientation primitives into one scannable two-second dashboard. No new logic — display layer only.

This skill comes out of cognitive prosthetics work: it was built for an operator with a traumatic brain injury whose re-orientation cost is the single biggest session tax. The format that survives that test case is the format that helps everyone — the difference is degree, not kind.

v0.2.0 added the Clock: a re-orientation surface that answers "where am I" but not "what's bearing down" misses prospective memory — remembering to remember. The first real fires of v0.1.0 kept needing deadlines added by hand; now the renderer reads them deterministically from a file.

## When to Use

- Mid-session, at a natural break, when the next move is unclear
- After a substantive build to confirm what shipped and what's still open
- When momentum is fine but altitude is wrong — you need the whole board
- Before context-switching to a different thread
- Returning to a session after walking away — the dashboard IS the pickup

NOT for:
- End-of-session closure (use your session-close protocol)
- Thread compression or archival
- One-off recall ("what did I say about X" — just answer it)

## The Protocol

### Phase 1 — Sweep

Gather two inputs:

1. **Open items** — everything incomplete, pending validation, deferred, or scheduled to fire later. If your stack has a dedicated open-items skill (e.g., a `/loose-ends` audit), call it as a subroutine. Otherwise sweep the conversation inline. Tag each item with one of five categories:
   - `[VALIDATE]` — shipped, needs a real-world fire to confirm
   - `[SAVE-AFTER]` — record to write once validation lands
   - `[PASSIVE]` — should self-verify on normal use
   - `[DEFERRED]` — consciously postponed
   - `[FUTURE-FIRE]` — scheduled to happen later
2. **The single next step** — one move, not a menu. If your stack has a next-step skill (e.g., `/whats-next`), call it; otherwise pick the one step the session context supports.

Don't deliver subroutine output verbatim. The results feed Phase 4.

### Phase 2 — Clock read (deterministic)

Read the deadlines file — default `~/.config/checkpoint/deadlines.txt`, override with the `CHECKPOINT_DEADLINES_FILE` environment variable. Format: one `YYYY-MM-DD|label` line per known hard date.

Render every entry within **14 days** of today, soonest first, with day-count and escalation markers: 🔴 ≤3 days, 🟠 ≤7, 🟡 ≤14. If the file is missing or nothing is in window, drop the section.

This is a file read, not a judgment call. See the Clock rules below.

### Phase 3 — Session delta read

Read the current conversation context. Extract:
- **Built:** what shipped this session (files written, commits, decisions reached)
- **Audited:** any review pass, scoring, or commitment made
- **Wired:** integration points, registry updates, runtime state changes that affect what fires next (model switches, new credentials, restarted services)

State what happened; don't editorialize. If a section has nothing, drop it.

### Phase 4 — Render

Output the dashboard in this exact format:

```
🧭 **Session Checkpoint**

🛠️ **Built**
- <delta items, one line each>

🧪 **Audited**
- <review/scoring items if any, drop section if none>

🔗 **Wired**
- <integration items if any, drop section if none>

⏰ **Clock**
- 🔴 <label> — <N> days (<date>)
- 🟡 <label> — <N> days (<date>)

🪢 **Loose Ends**
- <item> [TAG]
- (or "None open" if the sweep returned empty)

🎯 **Next**
- <the single next step>

🏷️ **Nameable as:** <what this session would be called if it ended now — one line, specific>

⛵ **Ship state:** <one word: clean / drifting / blocked / depleted>
```

Section order is fixed. Emoji-as-type-tag is fixed (they function as visual anchors, not decoration). Content terse — one line per item, no prose.

**The Nameable line.** A productive session is a session that produced something nameable; unnamed sessions dissolve. One line, specific enough to survive ("wired the deploy hook and filed the migration ticket", not "made progress on infrastructure"). If the session genuinely produced nothing nameable yet, write "Nothing nameable yet" — that's signal, render it honestly.

**Ship state rubric** (grounded, not vibes):
- **clean** — loose ends closing faster than opening, Next is unambiguous
- **drifting** — more ends opened than closed, or the session changed topic 3+ times without finishing one
- **blocked** — Next requires something the session cannot produce right now (an external person, a credential, a decision input)
- **depleted** — the user has named depletion. **The user names it first** — never diagnose depletion unprompted from message length alone; when uncertain between drifting and depleted, render drifting.

**Depleted protocol.** When ship state is depleted, the 🎯 Next item MUST drop altitude: one small winnable move, completable in minutes, stated in plain words. Not the strategically-correct next thing — the smallest thing that produces a win. The rest of the dashboard stays full (the board is still the board); only Next changes shape.

### Phase 5 — Hand back

Stop. Don't ask "want me to do X next." Don't pitch follow-up skills. The dashboard is the deliverable. The user decides what fires next.

## Rules

- **Ephemeral only.** No file writes. No event emissions. No memory updates. The moment a checkpoint persists, it competes with your session-close protocol and the boundary rots. (The Clock READS the deadlines file; it never writes it.)
- **Format is fixed.** Section order, emoji tags, terseness — non-negotiable. The format IS the value. A scannable two-second read is the whole point.
- **Clock is deterministic.** The deadlines file is a file read. Never invent a deadline, never extrapolate one from conversation. If the user names a new hard date mid-session, surface that it isn't in the file — adding it is their call, not checkpoint's.
- **No new logic.** This skill renders. If you find yourself doing analysis a subroutine should do, route back to it.
- **Mid-session only.** If the session is actually closing, route to your session-close protocol instead.
- **Drop empty sections.** Don't pad.
- **Ship state is one word.** Use the rubric.
- **Loose ends carry their tags.** The tag tells the user whether the item needs them, needs time, or needs nothing.

## Example Output

```
🧭 **Session Checkpoint**

🛠️ **Built**
- Deploy pipeline split into build + release stages
- Rollback script with dry-run flag
- Staging smoke test wired into CI

🧪 **Audited**
- Release-stage permissions reviewed — least-privilege confirmed
- Flaky test quarantined with linked issue

⏰ **Clock**
- 🟠 Conference talk slides due — 5 days (2026-07-08)
- 🟡 TLS cert renewal — 12 days (2026-07-15)

🪢 **Loose Ends**
- Rollback script: needs one real fire on staging [VALIDATE]
- Pipeline docs update [DEFERRED]
- Cert auto-renewal check fires Jul 15 [FUTURE-FIRE]

🎯 **Next**
- Trigger a staging deploy to validate the rollback path

🏷️ **Nameable as:** split the deploy pipeline and shipped a tested rollback

⛵ **Ship state:** clean
```

## Verification

Before delivering, confirm:
- [ ] Open-items sweep and next-step pick both ran (or one explicitly skipped with reason)
- [ ] Deadlines file was read (or noted missing)
- [ ] Section order matches spec
- [ ] No empty sections present
- [ ] Loose ends carry their category tags
- [ ] Nameable line is specific, or honestly "Nothing nameable yet"
- [ ] Ship state is exactly one word and matches the rubric
- [ ] No file writes happened
- [ ] Output ends at the dashboard — no follow-up prompts, no offers

## Known Limitations & Gotchas

- **Format-drift temptation.** You will be tempted to add "let me know if you want X" or trailing summaries. Don't. The dashboard ends, full stop.
- **Subroutine creep.** Open-items audits return prose; this skill compresses to one-line entries. Don't deliver subroutine output verbatim — that's two skills firing, not one rendering.
- **Persistence temptation.** "Should we log this?" No. The moment checkpoint writes to disk it becomes a worse session-close. Ephemeral is the design. (The first real fire of v0.1.0 wrote a checkpoint file — exactly this failure mode. The renderer renders.)
- **Clock invention.** A deadline mentioned in conversation but absent from the deadlines file does NOT go in the Clock — surface the gap instead. On a prospective-memory aid, an invented calendar entry is the worst possible failure.
- **Depletion over-diagnosis.** Short messages ≠ depleted. Many users compress input by design. The user names depletion first; checkpoint never announces it for them.
- **Empty session.** If nothing substantive happened yet, the dashboard will be thin. That's signal — return it anyway.

## Origin

2026-04-30. The operator hand-built the emoji checkpoint format in a working session and wanted it repeatable as a single trigger. The initial framing was "absorb the open-items and next-step skills into one." The design cut: don't absorb, compose. The component skills already existed with clean scopes; the missing primitive was the renderer.

The format itself (emoji-as-type-tag, fixed section order, terse content) is the whole value proposition: it lowers the cost of "where am I" from a multi-question query to a single trigger.

v0.2.0 (2026-06-12): Clock section (deterministic deadline reads), loose-end category tags, Nameable line, ship-state rubric, depleted protocol. Driven by the first real fires: every hand-improvised improvement that survived contact became spec; the one improvisation that violated spec became a named gotcha.

Cognitive prosthetics primitive "Checkpoint." Pollock 2026.
