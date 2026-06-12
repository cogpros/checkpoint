# checkpoint

Visible session state on demand. One trigger, one scannable dashboard: what you built, what's bearing down, what's left open, what's next.

```
🧭 Session Checkpoint

🛠️ Built
- Deploy pipeline split into build + release stages
- Rollback script with dry-run flag

⏰ Clock
- 🟠 Conference talk slides due — 5 days (2026-07-08)

🪢 Loose Ends
- Rollback script: needs one real fire on staging [VALIDATE]

🎯 Next
- Trigger a staging deploy to validate the rollback path

🏷️ Nameable as: split the deploy pipeline and shipped a tested rollback

⛵ Ship state: clean
```

## Why this exists

Long AI working sessions accumulate state faster than working memory holds it. "Where am I, what did I just do, what's next" fires constantly — and answering it usually means scrolling back through the transcript or asking three separate questions.

`/checkpoint` is a renderer. It composes the answers into one fixed-format dashboard you can scan in two seconds. No new analysis, no persistence, no session close — display layer only.

It comes out of cognitive prosthetics work: built for an operator with a traumatic brain injury whose re-orientation cost is the single biggest session tax. The format that survives that test case helps everyone — the difference is degree, not kind.

## What it is / is not

| | |
|---|---|
| **Is** | A mid-session orientation dashboard. Ephemeral. Fixed format. |
| **Is not** | A session-close protocol, a log, a todo system, or a planner. |

The ephemeral constraint is the design: the moment a checkpoint persists to disk, it starts competing with whatever you use for session closure, and the boundary between "glance at state" and "seal the session" rots.

## Install

This is an [Agent Skill](https://code.claude.com/docs/en/skills) (SKILL.md format, supported by Claude Code and a growing set of agent runtimes).

```bash
# Claude Code
mkdir -p ~/.claude/skills/checkpoint
curl -o ~/.claude/skills/checkpoint/SKILL.md \
  https://raw.githubusercontent.com/cogpros/checkpoint/main/SKILL.md
```

Or clone and symlink:

```bash
git clone https://github.com/cogpros/checkpoint.git
ln -s "$(pwd)/checkpoint" ~/.claude/skills/checkpoint
```

## Quick start

In any session:

```
checkpoint
```

That's it. The agent sweeps open items, picks the single next step, reads your deadlines file, extracts the session delta, and renders the dashboard.

## The Clock

The ⏰ Clock section reads hard dates from a plain text file — one `YYYY-MM-DD|label` line per deadline:

```
# ~/.config/checkpoint/deadlines.txt
2026-07-08|Conference talk slides due
2026-07-15|TLS cert renewal
```

Override the path with `CHECKPOINT_DEADLINES_FILE`. Entries within 14 days render with escalation markers (🔴 ≤3 days, 🟠 ≤7, 🟡 ≤14).

The Clock is deterministic by rule: it reads the file, never invents a date, never extrapolates one from conversation. If you mention a deadline mid-session that isn't in the file, checkpoint surfaces the gap instead of rendering it. On a prospective-memory aid, an invented calendar entry is the worst possible failure — so it's structurally forbidden, not just discouraged.

## How it works

Five phases:

1. **Sweep** — open items (tagged `[VALIDATE]` / `[SAVE-AFTER]` / `[PASSIVE]` / `[DEFERRED]` / `[FUTURE-FIRE]`) + the single next step. Composes companion skills if you have them; sweeps inline if you don't.
2. **Clock read** — deterministic file read of your deadlines.
3. **Session delta** — built / audited / wired, extracted from conversation context.
4. **Render** — the fixed dashboard format. Empty sections drop; nothing pads.
5. **Hand back** — the dashboard ends the turn. No "want me to…" follow-ups.

Two details that earn their bytes:

- **The Nameable line.** A productive session is a session that produced something nameable; unnamed sessions dissolve into "I worked all day and can't say on what." One specific line, or an honest "Nothing nameable yet."
- **Ship state** is one word — clean / drifting / blocked / depleted — with a grounded rubric for each, including a rule that the agent never diagnoses depletion unprompted. When the user names depletion, the Next item drops altitude to one small winnable move.

## Companion skills

Checkpoint composes; it doesn't absorb. It pairs naturally with:

- an **open-items audit** skill (it calls yours if present, sweeps inline if not)
- a **next-step** skill (same)
- your **session-close** protocol (checkpoint routes there if the session is actually ending)

None are required. The skill degrades gracefully to inline sweeps.

## Limitations

- The Clock only knows what's in the deadlines file. Garbage in, garbage out — but never garbage invented.
- A thin session renders a thin dashboard. That's signal, not a bug.
- The fixed format is non-negotiable by design. If you want a different shape, fork it — but the uniformity is most of the value.

## Versioning

- **0.1.0** (2026-04-30) — initial renderer: built/audited/wired + loose ends + next + ship state.
- **0.2.0** (2026-06-12) — Clock section, loose-end category tags, Nameable line, ship-state rubric, depleted protocol.

Planned: configurable extra count-file sources for the Clock (surface any integer state your stack exposes as a one-line presence marker).

## Related

- [ghost-hours](https://github.com/cogpros/ghost-hours) — measurement framework for AI-assisted productivity, same cognitive prosthetics lineage
- [ratatoskr](https://github.com/cogpros/ratatoskr) — gated web-fetch pipeline for agent contexts

## License

MIT. Pollock 2026, Raven Systems Inc.
