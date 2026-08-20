---
name: design-to-done
license: MIT
metadata:
  version: 1.0.0
description: The operating system for how projects get run in this repo. Every project lives in its own numbered folder, moves through six explicit states (design, build, review, patch, manual-qa, done), and carries its own design.md and progress.md. Use this skill whenever starting a new project or feature, writing or grilling a design doc, picking up work that was left in progress, implementing something that has a design, reviewing a change-set, fixing review feedback, or logging progress. Also use it whenever the user mentions a project folder, design.md, progress.md, "where are we on X", or asks what to do next. If work is substantial enough to survive more than one sitting, it belongs in this workflow. Load this skill again at every state transition, and whenever resuming project work later in a long session, even if it was already loaded earlier, because its contents do not survive context compaction and working from memory of it drifts.
---

# Design to Done

## The idea behind all of this

You have no memory. Every session you wake up in the middle of someone else's work, and that someone else was you. The conversation that produced the current state of the code is gone. What survives is what got written to disk.

That single fact drives the whole framework. Two files carry a project: `design.md` holds the intent, `progress.md` holds the state. If those two files are honest and current, any agent can be dropped into the project cold and be productive within minutes. If they drift, the project is effectively lost even though all the code is still there.

So the working assumption is: **you could be shot down at any moment.** Not as a dramatic flourish, but as a design constraint. Write as though the next person to open these files knows nothing and has no way to ask you anything.

But that constraint does not mean writing everything down. It means writing each thing down *in the right file*. There are three durable homes, and every fact you produce belongs to exactly one:

- **`design.md`** holds decisions. What we're building, and why it's that way rather than some other way.
- **The code** holds mechanics. How it works lives in the code, its comments, and its tests.
- **`progress.md`** holds standing. Where the project is, and what happens next.

Most of what passes through your head during a session belongs to none of them. The options you considered and discarded, the file you read to check an assumption, the half-formed plan you revised twice, the current draft of your thinking: that is the working state of a step, and it dies with the step. Persist the outcome, not the process that produced it.

The second idea is a hard separation between **deciding** and **doing**. Design is where thinking happens, where options are weighed and killed. Build is where a settled decision gets executed. When those two blend, you get implementations that quietly invent product decisions nobody made, and design docs that describe what the code happens to do rather than what it should do. Keeping them apart is what makes the design file trustworthy as a source of truth.

The third idea is that **state lives in the file, not in the conversation.** The header at the top of `progress.md` is what the project actually is right now. You read it to decide what to do next. You overwrite it the moment reality changes. Never carry state only in your head or in the chat.

Note the difference in how the two halves of `progress.md` behave. The header is **overwritten**: it always describes the present, and its old contents are worthless the moment they stop being true. The log is **appended**: entries are permanent, so an entry earns its place only if it will still be worth reading months from now. Transient state goes in the header, where it gets replaced. It does not go in the log, where it accumulates forever.

---

## Anatomy of a project

```
projects/
└── 0004_quote-followup-nudges/
    ├── design.md      # what we're building and why. the source of truth.
    ├── progress.md    # status header + milestone log. where we stand.
    └── assets/        # raw material: notes, screenshots, pdfs, dumps
```

**Numbering.** Four digits, zero-padded, sequential. Scan `projects/` for the highest existing number and take the next one. The number never changes and never gets reused, even if the project dies. Slug is short, lowercase, hyphenated, and describes the thing rather than the ticket.

**`design.md` does not exist at project creation.** A new project folder is just `progress.md` (status `design`) and an empty `assets/`. The design file accretes, one decision at a time, during the design conversation. Stubbing it out with empty headings in advance invites filling in a template rather than earning each decision.

**`assets/` is the junk drawer, on purpose.** Anything unstructured goes there without ceremony: a pasted error log, a screenshot of a competitor's UI, a PDF spec, a half-formed voice-note transcript. It has no schema and needs none. Its job is to make sure raw material has somewhere to land so it doesn't end up polluting `design.md`, which has to stay clean and decided.

---

## The six states

A project is always in exactly one of these. The current one is declared in the header of `progress.md` and kept accurate by you.

| State | Meaning |
|---|---|
| `design` | No `design.md` yet, or decisions remain open. Thinking is not finished. |
| `build` | Design is settled. Implementation can start or is underway. |
| `review` | Implementation is done. Review can start or feedback has been given. |
| `patch` | Review or QA surfaced critical/blocking issues. They are being fixed. |
| `manual-qa` | Review is clear of critical/blocking issues. Waiting on the user to verify by hand. |
| `done` | The user has approved it. Finished. |

The normal path runs `design → build → review → manual-qa → done`, with `patch` as the loop-back that catches anything the review or the user rejects. Bouncing between `review` and `patch` several times is expected and healthy, not a sign of failure.

**Your first move in any session is always the same:** open `progress.md`, read the header, read the last few log entries, and let that decide what you do. Do not guess from the conversation or from the shape of the code.

**Re-read this file before every state transition.** Long sessions get compacted, and when they do, the text of this skill is summarized out of your context while the project files on disk stay exactly as they were. The failure is quiet: you keep working, you keep writing to `progress.md`, and the entries slowly revert to whatever your defaults are. Assume that has happened rather than assuming it hasn't. If you cannot bring to mind the specific rules for the state you are entering, you are not remembering them, and the fix is four seconds of reading.

---

## Design

> Grill me on the design until we reach shared understanding.

This state is a conversation, not a document-generation task. The goal is that by the end, you and the user hold the same picture in your heads, and `design.md` is the artifact proving it.

**How to run it:**

- **Ask one question at a time, then stop and wait.** Not three questions with sub-bullets. One. A wall of questions gets a shallow batch answer, which is exactly the failure mode this process exists to prevent.
- **Each question is formatted exactly like this:**

  ```
  **<emoji: an emoji matching the question> <question: direct, clear, simple>**
  <context: what the tension is, what the realistic options are, what each one
  costs. Build the user up to the point where they can decide just by reading
  what you wrote. A bare question offloads work onto the user that you should
  have done.>
  <recommendation: your own answer with short reasoning, so they have something
  concrete to push against.>
  ```
- **Update `design.md` after every single decision, before asking the next question.** This is not bookkeeping. If the session dies mid-design, everything settled so far survives.
- **Walk the tree depth-first.** Resolve dependencies in order. A decision that unblocks three others comes before the three.
- **Continue until every meaningful branch is resolved.** Meaningful is the operative word. Stop when what remains is only the stuff a competent developer would decide correctly on their own.

**What goes in `design.md`:**

Product-level decisions. What we're building, and why. Skip anything an intelligent developer can trivially infer, and skip anything that is really an implementation detail wearing a design costume.

Write it the way you'd explain the project to a coworker over coffee. Narrative, direct, concrete. Prose over bullet-fragments. The reader should finish it *understanding* the project, not merely instructed by it. A design doc that reads like a checklist has failed even if every item on it is correct, because the next person can follow it without ever grasping why any of it is that way.

**Keep it current.** The design file is the single source of truth for the life of the project. When a build-time discovery invalidates a design decision, the file gets updated. Code that has drifted from the design is either a bug or an undocumented decision, and both need resolving in the file.

When the user considers it ready, set the state to `build`. The decisions themselves do not go in the log, because they are already in `design.md`. How the design actually went does: a branch you explored and abandoned, a question the user answered in a way that redirected everything after it, research or design work you handed to a subagent or a separate session and what came back from it. None of that is recoverable from `design.md`, which records only where you landed.

---

## Build

Start by becoming an expert. Not "read the relevant files" — become someone who could answer any question about this area of the codebase without looking.

Read `design.md`. Then read the code it touches, and the code that touches that. Understand existing changes and the intent behind them. Do not stop at the surface. When you think you understand, turn on yourself: ask the questions that would expose a shallow model. Why is this state held here and not there? What else reads this? What happens on the second call? Then go back into the codebase and close the gaps you just found. The bar is that you can implement the change with full confidence, not cautious optimism.

Only then, write yourself an internal implementation plan. This is a private todo list for organizing larger pieces of work, nothing more. It is not a second source of truth and it does not get committed to the design. **`design.md` is always the authority.**

Then implement it, fully. Every aspect of the design file, realized in a reasonable way.

Keep the header current as you go, and log as you go. Build is the state that generates the most history worth keeping and the state where losing it hurts most: an approach that failed, a file you touched and reverted, work you delegated, a constraint the user added mid-build. Write it down while it is happening, not in a summary at the end. Explanations of how the codebase works still belong in the code, and decisions still belong in `design.md`.

Move the project to `review` when the design is fully implemented.

---

## Review

You are a pragmatic senior engineer reviewing a pull request. Emphasis on pragmatic.

Read the full change-set of the current branch against main before forming any opinion. Understand what this work is trying to do and why. Build a mental model of the intent, from the files changed and from `design.md`.

Then put yourself in the user's shoes and walk through the feature the way a real person will use it. The happy path first, then the ordinary variations people actually hit.

Then zoom out. How does this land in the rest of the codebase? Did everything that needed updating get updated? Are there other features touching the same data, the same state, the same UI, that now behave differently without anyone noticing? Ripple effects are where the expensive bugs live.

For every finding, ask honestly: **would I actually block a PR for this?** If the real answer is "no, I'd mention it in passing," label it accordingly. Do not pad a review with nitpicks to look thorough. One real finding beats ten theoretical ones, and a review full of noise trains everyone to skim the next one.

**Focus on:**
- Functional correctness above all. If this shipped today, would a user hit a bug in normal use?
- Logic errors, off-by-one mistakes, race conditions that will realistically occur, missing handling for common failure modes.
- Broken contracts between components: a caller passing what the callee doesn't expect, a missing field, a wrong type.
- State management: things falling out of sync, stale data rendered, updates that don't propagate.

**Deprioritize:**
- Style, naming, and "I'd have done it differently," unless readability genuinely suffers.
- Edge cases requiring a bizarre sequence of events.
- Performance work that doesn't matter at current scale.
- Missing abstractions and refactors that would be nice but aren't needed for correctness.

**Severity:**
- 🔴 **CRITICAL** — a real bug users will hit in normal usage, or a serious oversight breaking core functionality. Must be fixed before merge.
- 🟡 **BLOCKING** — significantly degrades the experience, causes confusing behavior, or sets a maintainability trap we'll regret soon. Should be fixed, open to discussion.
- 🟢 **MINOR** — nice-to-have, style note, or unlikely edge case. Never holds up the PR.

**Format:** group findings by severity, most critical first. For each one, state the problem, where it is, and why it matters, in plain language. Show the code if it helps. Suggest a fix where you can.

If the change is solid, say so plainly. **A clean review is a valid outcome.** Manufacturing problems to justify the review is worse than finding nothing.

Deliver the full review to the user. In `progress.md`, log the verdict and the count, not the findings themselves: they are about to be either fixed or dismissed, and the patch entry is what will still matter afterwards. If there is nothing critical or blocking, move the project to `manual-qa`. Otherwise, put the open findings in the header and move to `patch`.

---

## Patch

Begin exactly as you do in build: rebuild deep understanding of what this work is doing. Do not go straight to the diff and start swatting comments.

Take the most recent review and work the CRITICAL and BLOCKING items. For each one, first verify it: is it real, and is fixing it reasonable? Be practical. Reviews are fallible, including your own. A finding that turns out to be wrong, or whose fix costs more than the problem, gets a documented decision rather than a silent skip.

Fix everything that survives that check.

Log what got fixed, briefly, and what you rejected and why. The rejections are the durable half: a finding dismissed without a recorded reason gets raised again by the next reviewer. If a rejection came from a design decision the reviewer had missed, the fix is to make `design.md` clearer, not to explain it in the log.

When done, move the project back to `review` for a fresh round.

---

## Manual QA

This state belongs to the user. Your job is to hand off cleanly and then wait.

Say what changed, what you'd like verified, and anything worth watching for. Then stop. Do not roll straight into more work.

- If the user gives feedback, record it in `progress.md` as a log entry, faithfully and in their framing, and set the state to `patch`.
- If the user approves, set the state to `done` and write a final log entry recording that they cleared it, along with any context worth preserving.

After a patch that came from manual QA rather than a code review, go back to `manual-qa` rather than `review`, unless the fix was broad enough to warrant a fresh review pass. The user asked for something specific; hand it back to them.

---

## Progress

**You are writing this to yourself.** Not to a manager, not to a reviewer. To the version of you that opens this project cold in three weeks with none of today's context, and has to pick the work back up without redoing it and without walking into the same walls twice. The user may glance at the file to check in, but they are not the audience. You are.

That changes what belongs here. Future-you does not need reassurance that things are going well. Future-you needs the handful of facts that are true about this project and are not recoverable by reading `design.md` or the code: what is actually finished versus what merely looks finished, where the code and the design have drifted apart, and what you learned the hard way that cost you something to learn.

It also changes what restraint is for. Every entry you write is context that future-you has to read and reconcile before doing anything. A log of forty entries is not thorough, it is a haystack, and the useful facts get buried in it. Protecting the signal in this file is protecting your own ability to think later.

**Format:** a header describing the present, then log entries marking the past, newest at the bottom, separated by `---`.

```markdown
# 0004 — Quote Follow-up Nudges
*Workflow: design-to-done. Re-read the skill before changing Status.*

**Status:** patch
**Next:** fix the two blocking review findings, then back to review
**Open:** contractor off-switch has no UI, API only for now
**Last updated:** August 19, 2026, 4:10pm

---

🌱 **Design opened**
*August 18, 2026, 9:20am*

Speccing out automated follow-up for quotes that go quiet. The original ask was
one line, "nudge customers who haven't responded," and it turned out to be hiding
three separate decisions rather than one: what counts as quiet, who gets the
message when a quote has several contacts, and whether contractors can turn it
off per quote.

Six branches to resolve, two closed. The one that gates the rest is whether a
nudge attaches to a quote or to a customer, because a customer sitting on four
open quotes is the normal case here, not the edge case, and the answer changes
the data model. The remaining four all hang off it.

---

✅ **Design settled, moving to build**
*August 18, 2026, 4:45pm*

Design is closed and written up in design.md. The committed shape: four
time-based triggers, SMS only, nudges attached to the quote, and a per-quote off
switch for the contractor.

Two things ended up narrower than the original one-line ask, so the shipped
behavior will not match anyone's memory of it. Email is out for this round, on
deliverability cost. And a customer sitting on several quiet quotes gets several
nudges rather than one combined message, which falls out of attaching to quotes
rather than to customers. Both were the user's call, not drift.

---

🤝 **UI design delegated to a separate session**
*August 18, 2026, 5:30pm*

User asked that the notification UI be designed in a spawned Claude session
rather than by me, while I take the scheduler and trigger logic. Handed over the
relevant part of design.md plus the existing settings-page components as
reference. Waiting on it before I touch anything under `web/settings/`.

Meanwhile I am starting on the scheduler, which is independent of whatever
comes back.

---

↩️ **Tried cron, backed out, using the existing queue**
*August 18, 2026, 6:40pm*

First attempt put the triggers on a standalone cron worker in
`workers/nudge-cron.ts`. Got it running, then abandoned it. Two schedulers
polling the same quotes table meant the existing reminder worker and this one
could both claim a quote, and the fix for that was a lock we would then own
forever.

Reverted the file. Triggers now enqueue onto the existing reminder queue in
`lib/queue/reminders.ts` instead. Not a design change, design.md is silent on
scheduling mechanism. If someone proposes a dedicated worker again later, this
is why it was rejected the first time.

---

🚧 **Three of four triggers working, 48-hour one blocked**
*August 19, 2026, 9:40am*

Working end to end against staging: the 24-hour, 72-hour, and 7-day triggers.
Touched `lib/queue/reminders.ts`, `lib/nudges/triggers.ts`, and the migration in
`db/migrations/0031_nudge_log.sql`. All three staged, nothing committed.

The 48-hour trigger fires inconsistently and I found why. Quote status is
written in two places, `api/quotes/[id]/status.ts` and the webhook consumer in
`api/webhooks/servicetitan.ts`, and only the first emits the event the scheduler
listens for. So a quote that goes quiet through the webhook path is invisible.
Next step is hooking the second writer, which is one extra emit rather than a
rework.

Half-finished right now: `lib/nudges/triggers.ts` has a stub `onWebhookStatus`
that does nothing. That is deliberate, not a bug to hunt.

---

🎨 **UI design came back, integrated**
*August 19, 2026, 10:20am*

The spawned session returned a settings panel design: per-quote toggle living
on the quote detail page rather than in global settings, on the argument that
contractors think per job. Reasonable and it matches how the triggers attach to
quotes, so I took it as-is and recorded the decision in design.md.

It did not cover the empty state for accounts with no quotes yet. I picked
something obvious rather than spawning again for it. Worth a look during QA.

---

🏗️ **Build complete, handing to review**
*August 19, 2026, 11:05am*

All four triggers live in staging and firing correctly against seeded data, both
status writers hooked. Everything in design.md is delivered with one exception,
and it is the kind that gets forgotten: the contractor off switch exists and
works, but has no UI, so it is reachable only through the API. design.md
describes it as done. It is not, and reading that file alone would leave you
believing it is. Needs either a follow-up project or a small addition to this
one.

---

🔍 **Review: two blocking, nothing critical**
*August 19, 2026, 4:10pm*

Both findings are in delivery rather than in what we decided to build. Retries
can fire the same nudge twice, and accounts created before March have no timezone
on record, which lands their nudges at UTC midnight. Neither one touches the
design, so this is a patch round rather than a reopen. Both fixes are local to
the delivery path and neither one reaches the data model.
```

**The workflow line at the very top never changes.** It is there because this file is the one thing you are guaranteed to open, which makes it the only reliable place to leave yourself a note that survives a compacted session. Carry it into every project.

**Everything else above the first `---` gets overwritten.** `Status` is the state. `Next` is the single sentence someone needs to resume. `Open` is whatever is currently unresolved, and it disappears when it resolves. This is where working state lives, and it is the reason the log does not need to carry it.

**Everything below is permanent, and it is a working journal, not a milestone list.** Append an entry when:

- you finish a real chunk of work, not just when a state ends
- you change approach, or try something and back out of it
- you hand work to a subagent, a separate session, a script, or the user, and again when it comes back
- the user redirects you, adds a constraint, or rejects something
- you discover something that changes the plan
- you are about to start something long or risky
- the state changes

The bar is not importance in hindsight. It is whether a session that resumed here without it would be missing something.

**The discipline here is not brevity, it is non-duplication.** Do not write down what `design.md` already decides or what the code already explains. Everything else about how the work actually went is yours to record, and you should be thorough about it. A long log of concrete history is fine. A short log of tidy milestones is not, however well written, because it reads as though the project advanced by magic.

**Name things.** An entry that says a phase is complete across three pages has told you nothing you could act on. An entry that names the files, the command, the error text, the deployment ID, the specific thing that broke and what fixed it, is worth rereading. Abstraction is the main failure mode in this file, and it is worse than verbosity, because vague summary looks like information while carrying none.

**Mid-work entries carry state, not just events.** If you are stopping mid-task or logging while work is in flight, say where you actually are: what is done, what is half-done, which files you have touched and which are staged, what you were about to do next, what is currently broken and expected to be. Future-you should be able to resume mid-task, not just mid-project.

**What an entry has to carry.** In a paragraph or two: what is genuinely done, what is left, what you learned that changes the picture, and above all *what diverged from the plan*.

Divergences are the highest-value content in this file, and the reason is specific. `design.md` is the source of truth, which means future-you will read it and believe the code matches it. Every gap between the two that you leave unrecorded is a trap you personally set. A feature that is built but unreachable, a design item quietly dropped, an approach abandoned mid-build: if it is not written down, the next session will either assume it works or waste a run rediscovering that it does not.

Learnings are the second reason to write. Something that cost you real effort to find out, and that neither the design nor the code makes obvious, is worth a sentence. A hard-won fact recorded once is cheap; the same fact re-derived every session is not.

**Don't invent schedules.** You have no reliable sense of how long anything takes in human time, so hours, days, and dates are guesses dressed up as facts, and future-you will read them as facts. Describe size and reach instead: whether the remaining work is contained or sprawling, how many pieces are left, whether a fix is local or touches the data model. That is the part you can be right about, and it is the part that actually informs what to do next.

**Record consequences, not content.** The trap is reading "don't restate decisions" as "say nothing." You do not need to write down what was decided about trigger timing or why that answer is right, because you will read `design.md` before you touch anything anyway. You do need to write down that it closed, what it committed the build to, and what it cost. Name the thing, then say what it changed.

The same discovery can legitimately appear in two places wearing two different hats. That the codebase writes quote status in two places belongs in the code, as a comment explaining why the scheduler hooks both. That it was invisible at design time and added unplanned work belongs here. Same fact, different jobs.

**Before writing any entry, check where its content actually belongs:**

- Is this a decision about what we're building, or the reasoning behind one? → `design.md`
- Is this an explanation of how the code works? → the code, its comments, or its tests
- Is this the current state of unfinished work? → the header, not the log
- Is this an option weighed during a design conversation and not chosen? → `design.md` if the reasoning matters, otherwise nowhere
- Is this an approach I actually attempted and abandoned? → the log, always
- Was this work done by a subagent, another session, a script, or the user? → the log, including what was asked and what came back
- Does the code now differ from what `design.md` promises? → the log, prominently

**The test for this file:** your context is wiped mid-task. You read the header and the recent entries and pick up exactly where you were, knowing what you had tried, what you had rejected, what was in flight, and what you were about to do. If anything in that list would have to be reconstructed from the code or from guesswork, the entry that should have carried it was too thin.

The opposite failure still exists, but it looks different from being long. It is restating decisions that live in `design.md`, explaining mechanics that live in the code, and narrating deliberation that went nowhere. Length is not the problem. Duplication and vagueness are.
