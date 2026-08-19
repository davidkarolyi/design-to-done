# design-to-done

A single agent skill to turn a vague idea into shipped code without babysitting the agent through it.

`design-to-done` defines how projects get run: every project lives in its own numbered folder in your repo, moves through six explicit states, and carries its own `design.md` and `progress.md`.

The premise is that agents have no memory, so the filesystem has to be the memory. `design.md` holds intent, `progress.md` holds state. Any agent dropped into the project cold should be productive within minutes of reading them.

## Install

```bash
npx skills add davidkarolyi/design-to-done
```

Or globally, across all your projects:

```bash
npx skills add -g davidkarolyi/design-to-done
```

## What it does

Projects move through six states, declared in the header of `progress.md`:

| State | Meaning |
|---|---|
| `design` | Decisions are still open. |
| `build` | Design is settled, implementation underway. |
| `review` | Implementation done, under review. |
| `patch` | Critical or blocking feedback being fixed. |
| `manual-qa` | Clear of blocking issues, waiting on human verification. |
| `done` | Approved. |

The normal path is `design → build → review → manual-qa → done`, with `patch` as the loop-back for anything rejected.

Each state has its own operating instructions in `SKILL.md`. The design state in particular is a grilling conversation: one question at a time, each with a written brief and a recommendation, with `design.md` updated after every single decision.

Markdown only. No scripts, no dependencies, nothing executable.

## License

MIT. See LICENSE.
