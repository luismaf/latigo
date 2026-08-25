# latigo

*A supervisor for fleets of terminal coding agents: hands out the work, checks it landed, and knows when to shut up.*

**Keep a fleet of terminal coding agents working.**

[Herdr](https://herdr.dev) puts coding agents into terminal panes and tells you
what each one is doing. It does not decide *who works on what*. That gap is
where fleets die: a pane finishes, goes idle, and stays idle until a human
notices. With a dozen panes, the human is the bottleneck — and idle panes are
the most expensive failure in the whole setup. They are also the quietest, which
is why they last so long.

`latigo` (Spanish for *whip*) is the missing supervisor:

1. it finds the panes that went idle,
2. **claims** one work item for each, atomically,
3. hands it over — and **verifies the message actually landed**,
4. and shuts up when there is no work, instead of inventing some.

```
$ latigo sweep
free:3  pending:41  (dev2 dev5 dev7)
latigo: WHIP dev2 item=IT-018 Port the CSV importer to the new schema
latigo: WHIP dev5 item=IT-022 Add index on invoices(customer_id, issued_at)
latigo: WHIP dev7 item=IT-023 Reconcile API types with the server DTOs
```

---

## Why this exists

Three ways a naive whip destroys a fleet, all of them observed in production:

| Failure | What happens | What `latigo` does |
|---|---|---|
| **It whips the humans** | `herdr agent list` returns *every* pane, including the one where a person is having a conversation with an assistant. It gets an automated "why are you idle" — mid-sentence, and billed to them. They rarely find it motivating. | Only panes running the **worker kind** (`opencode` by default) are workers. A `claude` pane is a conversation, never a worker. This invariant survives panes being renumbered and roles renamed; id-based exclusion lists do not. |
| **It whips other projects** | No `cwd` filter, so panes working on unrelated repos get handed this project's tickets. | A worker must have its `cwd` at the project root, or be explicitly adopted in `adopted-panes.conf`. |
| **It hands the same item to everyone** | Reading the first pending row without claiming it means a sweep that finds four idle panes tells all four to do the same ticket. Four panes colliding on one file is worse than four idle panes. | The item is claimed atomically through `latigo board take` before it is ever named in a message. |

And two that come from the terminal itself:

- **Sending a message is two calls, and the second one fails.** The text can sit
  typed in the agent's prompt box, unsent, for as long as you let it — agents are
  patient that way. `latigo send` uses `--wait --until working`, so it returns
  only when the agent actually started;
  otherwise it retries with focus + Enter, and if *that* fails it **reads the
  pane and stops**. A pane stuck in a menu goes deeper with every extra Enter —
  it needs `esc`, not persistence.
- **Status lies in both directions.** A pane between two tool calls reads as
  `idle`; a pane that crashed can read as `done`. Which is why nothing here trusts a
  single status read, and neither should you.

## The three valves

Without these, a watchdog becomes harassment, and a cornered agent starts
inventing work to look busy:

- **GRACE** — a pane must sit idle for a while before it is pushed. An agent
  between two tool calls is idle for seconds; talking to it there cuts its
  thread and you pay for the prompt anyway.
- **COOLDOWN** — never two whips to the same pane inside N seconds. A hard
  ceiling on spend per pane.
- **SILENCE** — empty board means everyone is left alone, with a single notice.
  A quiet worker with everything done is the correct state, not a fault to be
  corrected with messages. Ask any employee.

## Install

Requires `python3`, `git`, and [Herdr](https://herdr.dev) on `PATH`. No
libraries, no daemon, no database, no service to keep alive at 3am.

```bash
git clone <this-repo> ~/src/latigo
ln -s ~/src/latigo/bin/latigo ~/.local/bin/latigo   # anywhere on your PATH
```

## Use

Everything lives in `<project>/.latigo/`. Run it from inside a Herdr pane.

```bash
# 0. start the project with the rules already in place (safe to re-run)
latigo init

# 1. put workers in the project's panes (adopts unnamed ones, adds tabs to reach -n)
latigo deploy -n 6

# 2. load the board. Items are files: the message stays small and the request
#    is reproducible, reviewable and reusable.
latigo board add "Port the CSV importer to the new schema

Read src/import/*.rs first. Keep the public API. Do not touch the schema."

# 3. one sweep, or leave it running
latigo sweep
latigo sweep --loop 120

# 4. look at the fleet
latigo roster
latigo board list
```

### Commands

| Command | What it does |
|---|---|
| `latigo init [--dry]` | Start a project with the house rules, the lessons file and the boss guide already written. Never overwrites. |
| `latigo sweep [--loop [SEC]] [--dry]` | One pass over the fleet, or a loop. `--dry` reports without sending. |
| `latigo roster` | The fleet: name, pane, kind, status, whether it counts as this project's. |
| `latigo send <agent> <text>` | Send a message and verify it landed. Useful on its own. |
| `latigo board add\|take\|done\|release\|blocked\|list\|show\|pending` | The work board. |
| `latigo deploy [-n N] [--kind K] [--prefix P]` | Adopt unnamed workers, create tabs until there are N. |

Flags: `--dir PATH` picks the project; otherwise `$LATIGO_ROOT`, then the git
root of the current directory, then the current directory.

### Configuration (all optional)

| File in `.latigo/` | Purpose |
|---|---|
| `no-whip.conf` | Panes or agent names that must never be whipped. One per line, `#` comments. A manual override; the worker-kind filter is the real defence. |
| `adopted-panes.conf` | Panes that belong to this project despite a different `cwd`. They get an absolute `cd` prefixed to every request, or relative paths would resolve elsewhere and fail silently. Keep these lists **disjoint** between projects: a pane in two lists receives two items and loses one. |
| `rules.md` | House rules appended to every whip. Standing rules belong in a file, not hardcoded in the tool. |

## What a project starts with

`latigo init` is the point where a project inherits what the last one learned,
instead of rediscovering it. It writes three files:

- **`rules.md`** — appended verbatim to every request the fleet hands out, so the
  rules travel with the work instead of sitting in a document nobody opens. What
  is in there matters more than the tool:
  - **A build is not progress. A test run is not progress.** They are
    instruments. Nobody is paying for a compiler to be exercised.
  - **Never fill time with a build or a test suite to look busy.** Having nothing
    to do is a sentence you are allowed to say. Invented work costs money, burns
    the machine, and creates review load for someone else.
  - Verify the cheap way first: read the code, check the types of what you
    touched, run one targeted test, look at the running app.
  - **One machine, one build at a time.** Two link steps at once is not slower,
    it is a freeze — and somebody is sitting in front of that machine.
  - Only work that moves the project toward its objective counts.
  - English everywhere; 99% code, 1% prose; prolific and professional.
- **`LESSONS.md`** — the mistakes not to repeat, each one already paid for. Add
  to it whenever something costs you an hour. That is what it is for.
- **`BOSS.md`** — the boss role. Its first line is the one that matters: an empty
  board means every worker is silent, the whip does not invent work, so keeping
  the board stocked *is* the job.

Re-running `init` never overwrites: existing files are reported and left alone.

## The board

A TSV plus one Markdown file per item:

```
.latigo/board.tsv        state \t id \t created \t origin \t pane \t title \t attempts
.latigo/items/IT-007.md  the full request
```

Two properties worth knowing:

- **Writes are atomic**: temp file → `fsync` → `os.replace`, with a `.bak` and a
  guard that aborts any operation that would shrink the board. Truncate-then-write
  loses the whole board if the process dies in between. It does happen.
- **Items are not handed out from the head of the queue.** With many panes asking
  for work, the head becomes a meat grinder: the same item goes out again and
  again, burning its attempt budget without anyone actually working it, while the
  rest of the queue is never read. `take` picks the pending item with the *fewest
  attempts*, oldest first. After `ATTEMPT_CAP` bounces an item is marked
  `blocked` and stops circulating — an item nobody can close would otherwise
  recirculate forever, and every lap costs a prompt. The queue develops opinions
  about that ticket long before you do.

## What it deliberately does not do

- It does not invent work. An empty board means silence, not busywork dressed up
  as a refactor.
- It does not write to your repo, run your build, or commit anything.
- It does not answer dialogs. A pane waiting on an approval prompt is reported,
  not clicked through — read it and decide yourself. Nothing here should be
  allowed to say yes on your behalf.

## License

MIT. See [LICENSE](LICENSE).
