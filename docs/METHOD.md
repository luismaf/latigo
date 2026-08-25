# The method

`latigo` is a small tool. Most of the value is in how you run the fleet around
it. This is that method, written down.

## The guiding principle

> An idle pane is the most expensive failure.

A minute of an idle agent costs more than a minute of a working one, because
the fleet exists to parallelise. Everything below follows from that.

## The supervisor's loop

1. **Start the whip** — `latigo sweep --loop 120` in its own pane. Do not poll
   by sleeping blindly; let the sweep report.
2. **Read what changed** — new commits mean somebody closed something; a free
   pane means work must go out now; reports mean somebody needs an answer.
3. **Chain immediately.** The next request should exist *before* the current one
   closes. Never wait for a pane to finish in order to start thinking about what
   it does next.
4. **Check the slow ones.** A pane that has gone a long stretch with no commit
   and no report gets a nudge: "commit whatever you have, or answer in one line
   what you are on". Time without output is the measure — a busy-looking screen
   is not evidence of progress.
5. **Restart what is hung.** Same frozen thought, context not growing: start a
   new session and resend the request. A stuck pane is worse than a restarted one.

## Requests go in files

Send `read and execute .latigo/items/IT-042.md`, not three paragraphs typed into
a terminal. The message stays small, the request is reviewable, reproducible and
diffable, and the worker keeps the full text to refer back to. `latigo board add`
does exactly this.

A good request has four parts:

- **Context** — what already exists, what someone else already found out.
- **What to build** — concrete, in order.
- **Boundaries** — what not to touch. Above all: which files belong to other
  workers right now. Two agents editing one file is the most common self-inflicted
  wound in a fleet, and the fix is ownership, not merge skill.
- **Definition of done** — how it gets verified, and what to report.

## New session, or keep the context?

Starting a fresh session for every task is wrong, and never starting one is also
wrong. The test is a single question: **does the new request benefit from what
this session already knows?**

- **Fresh session** when the context is large and the new request is simple or
  unrelated, when the pane switches topic entirely, or when it is degraded
  (slow, repetitive, burnt).
- **Keep the context** when the pane has been on one complex thread for a while.
  There the context *is* the knowledge — decisions taken, files read, reasons
  why. Clearing it makes the agent re-explore everything from scratch: slower,
  and worse.

## Spend

- Put the volume on the cheap workers. Reserve the expensive ones for what only
  they can do.
- When a worker reports a usage limit, its work moves elsewhere. Do not keep
  prompting it: the messages queue up and fire all at once at reset, applying
  stale orders to a state that has moved on.
- Every whip costs a prompt. That is why GRACE, COOLDOWN and SILENCE exist, and
  why an item that keeps bouncing gets marked `blocked` instead of recirculating.

## Talking to a pane

Sending a message is two operations and the second is the unreliable one. In
order:

```bash
herdr agent prompt <pane> "read and execute .latigo/items/IT-042.md" \
  --wait --until working        # returns only if it actually started
herdr agent focus <pane>        # without focus the Enter may not submit
herdr agent send-keys <pane> enter
herdr agent read <pane> --lines 20   # STOP and look
```

Never insist blindly. If two attempts left it idle, read the pane. It is usually
sitting in a menu or a selector, and every extra Enter pushes it deeper in;
`esc` is what gets it out. `latigo send` implements this sequence.

## The invisible pane

A worker whose `cwd` is not the project root is not idle by choice — it is
invisible. It never appears in any report, because a pane that is never
considered is never reported. Find them with `latigo roster`, which shows every
pane and marks which ones count as this project's. Adopt the strays in
`.latigo/adopted-panes.conf` rather than restarting them: restarting throws away
the session.

## Naming

Agent names must match `[a-z][a-z0-9_-]{0,31}` — lowercase, no dots, no capitals.
Name a pane for what it does (`qa-sweep`, `importer`), not for the tool running
inside it. The name is the better address: it keeps working when panes get
renumbered.

## What not to do

- Do not touch another worker's files. Assign ownership per module and commit
  with explicit paths, never a blanket "add everything" on a shared tree.
- Do not send the same request twice without reading the pane in between.
- Do not restart panes while the agent tool's database is being maintained;
  work written to a file that is being replaced is lost.
- Do not treat silence as failure. A worker with nothing pending is correct.
