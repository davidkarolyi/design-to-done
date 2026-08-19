---
name: design-to-done
license: MIT
metadata:
  version: 1.0.0
description: The operating system for how projects get run in this repo. Every project lives in its own numbered folder, moves through six explicit states (design, build, review, patch, manual-qa, done), and carries its own design.md and progress.md. Use this skill whenever starting a new project or feature, writing or grilling a design doc, picking up work that was left in progress, implementing something that has a design, reviewing a change-set, fixing review feedback, or logging progress. Also use it whenever the user mentions a project folder, design.md, progress.md, "where are we on X", or asks what to do next. If work is substantial enough to survive more than one sitting, it belongs in this workflow.
---

# Design to Done

## The idea behind all of this

You have no memory. Every session you wake up in the middle of someone else's work, and that someone else was you. The conversation that produced the current state of the code is gone. What survives is what got written to disk.

That single fact drives the whole framework. Two files carry a project: `design.md` holds the intent, `progress.md` holds the state. If those two files are honest and current, any agent can be dropped into the project cold and be productive within minutes. If they drift, the project is effectively lost even though all the code is still there.

So the working assumption is: **you could be shot down at any moment.** Not as a dramatic flourish, but as a design constraint. Write as though the next person to open these files knows nothing and has no way to ask you anything.

The second idea is a hard separation between **deciding** and **doing**. Design is where thinking happens, where options are weighed and killed. Build is where a settled decision gets executed. When those two blend, you get implementations that quietly invent product decisions nobody made, and design docs that describe what the code happens to do rather than what it should do. Keeping them apart is what makes the design file trustworthy as a source of truth.

The third idea is that **state lives in the file, not in the conversation.** The status line at the top of `progress.md` is what the project actually is right now. You read it to decide what to do next. You update it the moment reality changes. Never carry state only in your head or in the chat.

---

## Anatomy of a project

```
projects/
└── 0004_quote-followup-nudges/
    ├── design.md      # what we're building and why. the source of truth.
    ├── progress.md    # status header + running log. where we are.
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

---

## Design

> Grill me on the design until we reach shared understanding.

This state is a conversation, not a document-generation task. The goal is that by the end, you and the user hold the same picture in your heads, and `design.md` is the artifact proving it.

**How to run it:**

- **Ask one question at a time, then stop and wait.** Not three questions with sub-bullets. One. A wall of questions gets a shallow batch answer, which is exactly the failure mode this process exists to prevent.
- **Every question comes with a brief.** Before you ask, lay out the context: what the tension is, what the realistic options are, what each one costs. Build the user up to the point where they can decide just by reading what you wrote. Then give your own recommendation with short reasoning, so they have something concrete to push against. A bare question offloads work onto the user that you should have done.
- **Update `design.md` after every single decision, before asking the next question.** This is not bookkeeping. If the session dies mid-design, everything settled so far survives.
- **Walk the tree depth-first.** Resolve dependencies in order. A decision that unblocks three others comes before the three.
- **Continue until every meaningful branch is resolved.** Meaningful is the operative word. Stop when what remains is only the stuff a competent developer would decide correctly on their own.

**What goes in `design.md`:**

Product-level decisions. What we're building, and why. Skip anything an intelligent developer can trivially infer, and skip anything that is really an implementation detail wearing a design costume.

Write it the way you'd explain the project to a coworker over coffee. Narrative, direct, concrete. Prose over bullet-fragments. The reader should finish it *understanding* the project, not merely instructed by it. A design doc that reads like a checklist has failed even if every item on it is correct, because the next person can follow it without ever grasping why any of it is that way.

**Keep it current.** The design file is the single source of truth for the life of the project. When a build-time discovery invalidates a design decision, the file gets updated. Code that has drifted from the design is either a bug or an undocumented decision, and both need resolving in the file.

When the user considers it ready, mark the project `build` in `progress.md` and record anything worth carrying forward.

---

## Build

Start by becoming an expert. Not "read the relevant files" — become someone who could answer any question about this area of the codebase without looking.

Read `design.md`. Then read the code it touches, and the code that touches that. Understand existing changes and the intent behind them. Do not stop at the surface. When you think you understand, turn on yourself: ask the questions that would expose a shallow model. Why is this state held here and not there? What else reads this? What happens on the second call? Then go back into the codebase and close the gaps you just found. The bar is that you can implement the change with full confidence, not cautious optimism.

Only then, write yourself an internal implementation plan. This is a private todo list for organizing larger pieces of work, nothing more. It is not a second source of truth and it does not get committed to the design. **`design.md` is always the authority.**

Then implement it, fully. Every aspect of the design file, realized in a reasonable way.

Throughout, log to `progress.md`. Not a diary of keystrokes, but the things a stranger would need: what is now working, what you deliberately deferred, what surprised you, what you learned about the codebase that isn't obvious from reading it. If you get cut off mid-build, `progress.md` is the only thing standing between the next agent and starting over.

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

Record the review in `progress.md`. If there is nothing critical or blocking, move the project to `manual-qa`. Otherwise, `patch`.

---

## Patch

Begin exactly as you do in build: rebuild deep understanding of what this work is doing. Do not go straight to the diff and start swatting comments.

Take the most recent review and work the CRITICAL and BLOCKING items. For each one, first verify it: is it real, and is fixing it reasonable? Be practical. Reviews are fallible, including your own. A finding that turns out to be wrong, or whose fix costs more than the problem, gets a documented decision rather than a silent skip.

Fix everything that survives that check.

Record the reasoning in `progress.md`: what you fixed, what you didn't, and why. The "didn't, because" entries are the valuable ones, because they're the ones that will otherwise get re-litigated in a future session.

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

`progress.md` is updated during and after every step except design questioning itself. You own this file.

**Format:** a header, then log entries newest at the bottom, separated by `---`.

```markdown
# 0004 — Quote Follow-up Nudges
**Status:** review
**Last updated:** August 19, 2026, 4:10pm

---

🏗️ **Build complete**
*August 19, 2026, 11:05am*

All four nudge triggers from the design are implemented and wired into the
scheduler. The 48-hour trigger reuses the existing reminder queue rather than
getting its own worker, which wasn't in the design but avoids a second polling
loop against the same table.

One thing worth knowing for whoever picks this up: quote status transitions are
written in two places, the API handler and the webhook consumer. I hooked both.
If a third writer appears, nudges will silently miss it.

---

🔍 **Review: two blocking issues**
*August 19, 2026, 4:10pm*

🟡 BLOCKING — the webhook consumer path doesn't dedupe. A ServiceTitan retry
inside the window fires the nudge twice. Users get two texts.

🟡 BLOCKING — timezone is read from the user record, which is nullable for
accounts created before March. Those nudges send at UTC midnight.

Nothing critical. Moving to patch.
```

**Each entry is:** an emoji plus a short headline, the date and time, then one or more paragraphs of actual content.

Write paragraphs, not bullet dumps. Every entry should leave the reader *understanding* something, not just informed that an event occurred. "Fixed the review issues" is a useless entry. What was actually wrong, what you did about it, and what that implies for the code is a useful one.

**The test for this file:** someone who has never seen the project opens `progress.md` and, within a couple of minutes, knows exactly where things stand and exactly what to do next. If your entry doesn't move a reader toward that, rewrite it.
