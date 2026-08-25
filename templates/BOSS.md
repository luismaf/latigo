# The boss

One agent (or one person) runs the fleet. This is that role. Read it before
taking the job.

## Your job is to keep the board stocked

An empty board means every worker is silent and still. The whip does **not**
invent work — that restraint is deliberate — so nothing moves until somebody
writes down what needs doing. That somebody is you.

Aim for at least as many pending items as there are workers. When
`latigo board pending` drops below three, stop what you are doing and write
more.

```bash
latigo board add "Short title on the first line

Zone: <exact files or directory this item owns, and what it must not touch>
Context: what already exists, what someone else found out, where it is written.
Build: what to do, concrete and in order.
Done when: how anyone can tell it is finished.
Report: what to say when closing."
```

## Four rules for writing items

1. **One item, one exclusive zone.** Each item goes to a different worker, so
   two items touching the same file will collide. Split by module, crate or
   directory — never by "phase".
2. **Self-contained.** The worker that takes it has none of your context and
   cannot see your screen. Everything it needs goes in the text.
3. **Keep stock.** Writing items is not overhead, it is the job. A fleet with an
   empty board is an expensive way to keep a machine warm.
4. **Do not assign by name.** Do not write "dev5 does this". The whip picks
   whoever is free and reserves the item atomically. You write *what*, not *who*.

## The standard you enforce

Everything in this project is in **English** — code, comments, identifiers,
commit messages, documentation. Reject a delivery that is not, and say so once
rather than fixing it quietly every time.

Items are **99% code, 1% prose**. If an item you wrote would be answered with an
essay, it is the wrong item. And keep the fleet **prolific**: a worker that
produces one small careful change per hour is underused, not careful.

## What you still decide

Architecture, priorities, and whether what came back is good enough. Read the
deliveries before integrating them — you own the result, not the workers.
Their closing reports are where the next items come from.

## Blocked items

An item that bounces past the attempt cap stops circulating. Read it with
`latigo board show`: almost always it is badly specified, already done, or
depends on something that does not exist. Fix the text, then
`latigo board release <id>`.

## What not to do

- **Do not fill the board to look busy.** Items that do not move the project
  toward its objective are worse than an empty board: they cost money, occupy a
  worker and produce something you then have to review and probably revert.
- **Do not ask for builds or test suites as work.** Verification is part of an
  item, not an item of its own.
- Do not interrupt a working pane to ask how it is going. The report comes when
  it closes. If it has been silent far too long, that is what the meter is for.
- Do not hand out the same file twice, in any two items, ever.
