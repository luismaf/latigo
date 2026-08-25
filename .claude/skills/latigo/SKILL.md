---
name: latigo
description: Run a fleet of coding agents in Herdr terminal panes — deploy workers, load a work board, keep idle panes working, and send messages that actually land. Use when asked to deploy or orchestrate a team of agents with Herdr, to act as boss/supervisor of a fleet, to stop devs sitting idle, or to hand out work across panes. Requires HERDR_ENV=1.
---

# latigo — supervising a Herdr fleet

Herdr puts coding agents in terminal panes. It does not decide who works on
what. `latigo` fills that gap. Use this skill whenever the user asks to deploy,
orchestrate, supervise or "whip" a team of agents.

Check first — every command here needs a live session:

```bash
test "${HERDR_ENV:-}" = 1 && command -v latigo
```

If `latigo` is not installed: `ln -s <repo>/bin/latigo ~/.local/bin/latigo`.

## The five invariants

These are not style preferences. Each one comes from a fleet breaking.

1. **Only worker-kind panes are workers.** A `claude` pane is a conversation —
   the user's own session or the supervisor's. Whipping one interrupts a human
   mid-sentence and bills them for it. Filter by `agent == "opencode"` (the
   worker kind), never by pane id or name: ids change when sessions reopen,
   which is how id-based exclusion lists rot.
2. **Filter by `cwd`.** A pane whose working directory is not the project root
   belongs to another project, or is invisible and needs adopting. Never hand
   this project's work to it blindly.
3. **Claim the item before naming it.** Without an atomic claim, one sweep hands
   the same ticket to every idle pane, and several agents editing one file is
   worse than several idle agents. Use `latigo board take`.
4. **Sending is two calls and the second one fails.** `herdr agent prompt` can
   leave the text typed in the box, unsent. Always verify with
   `--wait --until working`. If it does not start: `focus`, then `enter`, then
   **read the pane and stop**. A pane stuck in a menu goes deeper with every
   extra Enter; it needs `esc`.
5. **`agent_status` lies in both directions.** A pane between two tool calls
   reads `idle`; a crashed one can read `done`. Never conclude from one read.

## Normal flow

```bash
latigo init                        # first: the house rules and lessons, once
latigo deploy -n 6                  # adopt unnamed workers, add tabs up to 6
latigo board add "$(cat request.md)"   # work goes in as files, one per item
latigo sweep                        # one pass
latigo sweep --loop 120             # or leave it running in its own pane
latigo roster                       # who exists, status, whose project
latigo board list                   # what is pending, taken, blocked
latigo send dev3 "read and execute .latigo/items/IT-007.md"
```

Run `latigo sweep --dry` first when unsure: it reports what it *would* do.

## Starting a project

Run `latigo init` before anything else. It writes `rules.md` (attached to every
request the fleet hands out), `LESSONS.md` (mistakes not to repeat) and
`BOSS.md`. The rules it installs are the working standard, and they hold whether
or not the tool is in play:

- A build is not progress; a test run is not progress. Never use either to look
  busy — having nothing to do is a sentence you are allowed to say.
- Verify cheaply first: read the code, type-check what you touched, one targeted
  test, look at the running app. One machine, one build at a time.
- Only work that moves the project toward its objective counts.
- English everywhere — code, comments, identifiers, commits, docs.
- 99% code, 1% prose. Prolific and professional.

## Supervising, not just whipping

The tool handles the mechanics. The judgement is yours:

- **Chain ahead.** The next request should exist before the current one closes.
- **Requests go in files**, not typed into a terminal: small message, full
  context, reproducible, reviewable.
- **Give every request boundaries** — above all which files belong to other
  workers right now. Ownership prevents collisions; merge skill only cleans up
  after them.
- **Measure by output**, not by a busy-looking screen. A long stretch with no
  commit and no report earns a nudge; a frozen thought with a static context
  earns a fresh session and a resend.
- **New session or not?** Ask whether the new request benefits from what this
  session already knows. If yes, keep the context — it *is* the knowledge. If
  no, start clean.
- **Silence is a valid state.** A worker with nothing pending is correct, not
  broken. Never invent work to fill a pane.

## The three valves

Built into `sweep`, tunable with `--grace` and `--cooldown`:

- **GRACE** — a pane must be idle a while before it is pushed (it may just be
  between tool calls).
- **COOLDOWN** — never two whips to one pane inside the window.
- **SILENCE** — empty board, one notice, then leave everyone alone.

## Configuration

In `<project>/.latigo/`: `no-whip.conf` (manual exclusions),
`adopted-panes.conf` (workers whose cwd is elsewhere — they get an absolute
`cd` prefixed), `rules.md` (house rules appended to every whip).

Keep adoption lists **disjoint** across projects: a pane in two lists receives
two items and loses one.

## When something looks wrong

```bash
latigo roster                                  # invisible panes show up here
herdr agent read <pane> --source visible       # look before insisting
herdr agent get <pane>
latigo board list                              # blocked items stopped circulating
```

An item marked `blocked` bounced past the attempt cap: nobody could close it.
Read it — usually it is badly specified, already done, or depends on something
that does not exist. Fix the request, then `latigo board release <id>`.
