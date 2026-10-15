---
name: build-signal-engine
description: Build one person's personal AI and dev news digest (a signal engine with a fixed diet) - X first, then GitHub releases, Hacker News, and RSS, with seen-item memory, code filters, a cheap-model keep/drop pass, and a 10-item daily digest. Interviews only for their accounts, repos, feeds, and keep/drop examples, then builds it into their repo one stage at a time on real data. Use when the user wants a personal AI news digest, a daily dev briefing, an information diet, to keep up with AI without doomscrolling, or something that reads X, GitHub, HN, and RSS and says what matters.
argument-hint: "[optional: path/to/repo]"
arguments: [repo]
---

# Build a signal engine

**What the user typed:** $ARGUMENTS

> If a path was given, it is the repo to build in; confirm it in one sentence. If not, ask
> whether to start a new folder or use the current one. Then go straight to the profile.

Build one person's daily AI and dev news digest. Every morning it pulls X, then GitHub
releases, Hacker News, and RSS; remembers what it has already seen; drops the rest with
code filters and a cheap-model keep/drop pass; and writes one digest of at most ten items.

**Keeping up is a filtering problem, not a reading problem.** The engine encodes this
person's filter and runs it on those four sources.

**Build it into their repo. Do not hand them a plan.** Every stage ends with code committed
and output from a real run they can look at.

---

## Output discipline

The profile and the build are long. What you say is not.

| Moment | Budget |
|---|---|
| Between questions | Nothing. Ask the next one. |
| Finishing a stage | Two lines: what now exists, with the real number from the run, and the next step. |
| A file you wrote | One line: its path. |
| Command output | Never paste it. The verdict and the number. |

Never announce a plan before doing it, restate an answer as a paragraph, or explain why the
skill works this way unless asked. The reasoning lives in `references/`, for you.

---

## The shape

```
X, then GitHub releases, Hacker News, RSS
    -> store (seen-item memory, stable ids)
    -> code filters
    -> cheap-model keep/drop
    -> reading pass writes at most 10 items
    -> digests/YYYY-MM-DD.md, once a day
```

Two splits carry the design. Say each once, early, in one sentence:

- **Numbers in code, judgment in the keep/drop pass, reading in the digest model.** Dates,
  points, drafts, prereleases, and "already seen" are code. "Is this new AI/dev news for
  this person" is the cheap model. Summarising is the digest model. Mixing them up is the
  most common bug.
- **The question is the policy.** What the engine keeps is the words of the keep/drop
  questions, written from this person's examples. Changing the question changes the engine.
  There is no model setting that fixes a vague question.

One person, one profile, one SQLite file. No second reader and no shared tenant.

## What this skill does not build

Leave these out. Do not add a source, a stage, or a model path for them.

- Any source besides X, GitHub releases, Hacker News, and RSS.
- Weekly roll-ups, content-idea sections, shadow-week programs, competitor monitors,
  research watches, and multi-user digests.
- A typed decision-model service, or a larger model on every item. Keep/drop is one cheap
  model. The digest is one call per run.
- Permission-skip or safety-skip flags on any model or agent call. See Safety.

When the user asks for one of those, say this skill is the personal AI/dev digest and stop
widening it. A source whose terms forbid automated access, and that has no official API,
stays out.

---

## Construction order

| Stage | What exists at the end | Proof before moving on |
|---|---|---|
| 0 | The profile, written down | `PROFILE.md` the user agreed to |
| 1 | X, end to end, with seen-item memory | a real run: N items, all with stable ids; a re-run adds ~0 |
| 2 | GitHub releases, Hacker News, RSS | each source's real count, and a re-run that adds ~0 |
| 3 | Code filters | how many each filter dropped |
| 4 | The cheap-model keep/drop pass | their own keep/drop examples scored, misses listed |
| 5 | The digest | at most 10 items from today's real data |
| 6 | Delivery + daily schedule | one run by hand; the schedule exists only after they agree |

**X first, end to end, before the other three.** A pipeline built source-wide but never run
end to end fails at the join between stages. If the profile says X is off, Stage 1 proves
the store and an empty X fetch, then the other three are built in the same order.

**Anything that bills per row is capped on every run** and only runs on the daily schedule.
That is X, when it bills, and every keep/drop call.

---

## Stage 0. Profile

Read `references/interview.md` and write `PROFILE.md` from `templates/PROFILE.md`. The diet
is already decided. Ask only for the accounts, repos, feeds, examples, and delivery
destination this person has to supply. Read the profile back as a proposal and get a yes
before Stage 1.

## Stages 1-3. Sources, seen-item memory, code filters

Read `references/sources.md`. Build X first. Every source maps to `templates/item_schema.md`.
Dedupe and seen-item memory run before anything calls a model. Store in SQLite unless they
already run something else. Code filters drop by fact and report how many each one dropped.

## Stage 4. Cheap-model keep/drop

Read `references/decisions.md`. Every new item that survived the code filters gets the
questions from `PROFILE.md` in one cheap-model call. Code routes each answer to keep or
drop. A failed call is not a drop.

Prove it on their examples before trusting it. Tune the words of the question first.

## Stages 5-6. Digest, delivery, schedule

Read `references/digest.md`. The digest model reads only what keep/drop kept, writes at most
ten items in the format from `PROFILE.md`, and keeps every link. Start from
`templates/digest_prompt.md`.

Deliver `digests/YYYY-MM-DD.md` first, then the destination they named. Ask before
registering the daily schedule. The scheduled command is the engine program, not an agent
process.

---

## Safety

Before any model call or any agent process, during the build and inside the engine, read
`references/safety.md` and follow it.

Permission checks and provider safety checks stay on. Default commands and default request
bodies do not include `--dangerously-skip-permissions`, `yolo`, or any equivalent. The only
opt-in for extra agent arguments is `agent_args_override`. It is empty unless the user set
it. Do not set it for them. Do not add a switch that turns model safety off.

---

## Operating facts

- Keys live in the repo's gitignored env file and the engine loads them from there. Never
  read or output that file with any tool or command (cat, sed, grep, Get-Content, Read,
  `open()`), not even "masked": a masking regex that is slightly wrong prints every key. To
  check a key, load the file with python-dotenv inside a program and print only the key's
  name and whether it is set. If one is missing, ask the user to add it themselves rather
  than pasting it into the chat.
- The live SQLite file is the seen-item memory. Test every run against a throwaway database,
  never the one that will go live, and never delete the live file to force a clean run.
- On Windows, force UTF-8 output from the start (`PYTHONIOENCODING=utf-8` or reconfigure
  stdout). Titles from X and Hacker News carry emoji and non-English text that crash the
  console.
- A cheap model varies between runs. A row that was kept once and dropped the next time is
  not, by itself, proof the question is wrong.
- Per-row fetch cost can exceed the whole model bill. Keep/drop saves on reading, not on
  fetching. Cap X before spending time on stricter filters.

## Resources

- `references/interview.md` - the short profile interview and the defaults to confirm
- `references/sources.md` - X, GitHub releases, Hacker News, RSS, seen-item memory, code filters
- `references/decisions.md` - the cheap-model keep/drop pass and how to prove it
- `references/digest.md` - the 10-item digest, delivery, and the daily schedule
- `references/safety.md` - permission and safety checks, and `agent_args_override`
- `templates/PROFILE.md` - the file the interview produces and every stage reads
- `templates/item_schema.md` - the one item shape every source maps to
- `templates/digest_prompt.md` - the starting digest prompt
