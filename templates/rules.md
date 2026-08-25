# House rules

Appended to every request handed out in this project. These are the rules that
get broken most, which is why they travel with the work instead of sitting in a
document nobody opens.

## How we work

- **English, always.** Code, comments, identifiers, file names, commit messages
  and documentation. No exceptions, including in a project whose users speak
  another language — the user-facing strings live in the translation files, and
  nowhere else.
- **99% code, 1% prose.** The deliverable is working code. Do not write an essay
  where a patch is expected, do not restate the request back, and do not narrate
  your plan before doing it. A closing report is one short paragraph.
- **Be prolific, and be professional.** Ship a lot, and ship it to a standard you
  would defend in review: named things, handled errors, no dead code, no
  commented-out experiments left behind. Volume and quality are not a trade-off
  here; both are the job.

## What counts as progress

- Only work that moves the project toward its objective counts as progress.
- **A build is not progress. A test run is not progress.** A green check is an
  instrument, not a deliverable. Nobody is paying for a compiler to be exercised.
- **Never fill time with a build or a test suite to look busy.** If you have
  nothing to do, say so in one line and stop. A quiet worker with everything
  done is the correct state. Invented work costs money, burns the machine, and
  creates review load for somebody else.
- If the item you were given turns out to be already done, or wrong, say that
  and stop. Do not manufacture an adjacent task to justify the turn.

## Builds and tests are expensive

- **Do not run the full build or the full test suite.** They are the most
  expensive thing in this project and the least informative per second spent.
- Verify the cheap way, in this order: read the code, check the types of what
  you touched, run one targeted test, look at the running app.
- A change you are not about to exercise does not need to be built at all.
- Frontend-only change: no backend build. That is what hot reload is for.
- **One machine, one build at a time.** Several compilers at once on a shared
  machine is not slower, it is a freeze — and somebody is sitting in front of
  that machine. Take the build lane, and waiting your turn is the correct
  outcome.
- Where a supervisor compiles centrally, you do not compile at all: you report,
  and the errors come back to you with file and line.

## Your zone

- Work only in the files your item names. Two workers editing one file is the
  most common self-inflicted wound in a fleet, and it is prevented by ownership,
  not by merge skill.
- Commit with explicit paths. Never "add everything": the tree is shared, and
  you will carry somebody's half-finished work into your commit.
- Do not reorganise, rename or reformat outside your zone because you happened
  to be passing.

## Reporting

- Close your turn with what you did, which files you touched, what is missing
  and what blocks you. One short paragraph, not a transcript.
- If the deliverable is long, write it to a file and leave only the path.
- Report when you are blocked immediately. A worker stuck quietly is worse than
  a worker who says it cannot proceed.
