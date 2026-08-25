# Mistakes not to repeat

Every line here was paid for once. The point of writing them down is that a new
project does not get to pay again, and a new worker does not have to discover
them by breaking something.

Add to this file whenever something costs you an hour. That is what it is for.

## Running a fleet

- **A pane running a conversation is not a worker.** Filter workers by the agent
  kind, never by pane id or name. Ids change when a session reopens, and
  name-based exclusion lists go stale silently — then an automated "why are you
  idle" lands in the middle of someone's sentence, billed to them.
- **A worker in the wrong directory is invisible, not idle.** It never matches
  the project filter, so it is never considered, never reported and never fed.
  Audit by listing every pane with its working directory, not by trusting that
  the ones you see are all of them.
- **Claim work before you name it.** Reading the first pending row without
  claiming it hands the same ticket to every idle pane in the sweep.
- **Do not dispatch from the head of the queue.** With many workers asking, the
  head becomes a meat grinder: the same item goes out over and over, burns its
  attempt budget without anyone actually doing it, and the rest of the queue is
  never read. Fewest attempts first, oldest to break ties.
- **An item nobody can close must stop circulating.** Cap the attempts, mark it
  blocked, and go read it: it is badly specified, already done, or depends on
  something that does not exist.
- **A dry board silently looks like "no work right now".** So does a dispatcher
  that crashed, and a loop pointed at a file that does not exist. Check that the
  machinery is alive, not just that nobody is complaining.

## Talking to an agent

- **Sending is two operations and the second one fails.** The message can sit
  typed in the prompt box, unsent, indefinitely. Verify that the agent actually
  started; do not assume delivery.
- **Status lies in both directions.** A pane between two tool calls reads idle;
  a crashed one can read done. Never conclude from a single read.
- **Never insist blindly.** If two attempts left it idle, read the screen. A pane
  stuck in a menu goes deeper with every extra Enter — it needs an escape, not
  persistence.
- **Do not prompt an agent that is out of quota.** The message queues and fires
  at reset, applying stale orders to a state that has moved on.
- **Do not interrupt the human's terminal with automated notices.** Write them
  to a file the human reads when they choose.

## Working on a shared machine

- **One build at a time.** Two link steps at once exhaust memory, the machine
  swaps to death, and the person in front of it pays for it. Waiting your turn
  costs nothing.
- **One commit at a time on a shared worktree.** Concurrent commits fight over
  the index lock, and the loser either fails or commits somebody else's
  in-flight files.
- **Builds and tests are the expensive part.** Reach for the cheap check first.
  Time not spent compiling is the single largest saving available.

## Writing state to disk

- **Truncate-then-write loses everything if the process dies mid-write.** Write
  to a temporary file, flush, replace atomically, and keep a backup.
- **Guard against shrinking.** If an operation that should only modify rows
  produces fewer rows than it started with, abort and write nothing.
- **Preserve mode and owner across an atomic replace**, or the file comes back
  with permissions nobody else can use, and everything that reads it stops.

## Paths and configuration

- **Never hardcode a repository path.** Resolve it from where the script lives
  or from the caller's context. A hardcoded path makes the tool useless on
  another machine and silently wrong on this one the day the repo moves.
- **Deploy shared tooling by symlink, not by copy.** Copies rot: one project
  ends up running a version that was fixed everywhere else weeks ago.
- **Per-project state is never shared.** The board, the queues and the roles are
  what make two projects different from each other.
