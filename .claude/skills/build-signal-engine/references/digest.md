# The digest, delivery, and the daily run

## The reading pass

One model call per run, over items the keep/drop pass kept and has not already put in a
digest. Rank them before the call: keeps in the order X, then GitHub releases, then Hacker
News, then RSS. Send at most a few times the digest cap (15 is enough). The model does not
go looking for more.

Start from `templates/digest_prompt.md`. Fill the focus, the cap, and the format from
`PROFILE.md`.

- Give the model title, source, url, and the first ~300 characters of the body. Fetch a
  full body only for an item that reached this pass and whose excerpt is not enough to say
  what changed.
- The model merges items that are the same story (a post, a release, and a feed entry that
  seen-item memory did not already collapse) into one entry, and writes one or two
  sentences.
- **Every link stays.** The digest is a pointer.
- The cap is the length cap in `PROFILE.md`. For this skill that cap is 5. Do not raise it.
  If fewer than five items were kept, the digest is shorter. Never pad with drops or with
  items that have no decision.
- Tell it to say plainly when an item is a rumor or a minor change, and never to call a
  release a launch unless the item says something launched.

Invoke the model the way `references/safety.md` requires.

## Format

```markdown
# <Date> - <one line on what mattered today>

## <Group>
- **<Item in plain words>** - one or two sentences on what changed and why it matters.
  [source](url)

## Also worth a look
- one-liners with links, only if they fit under the cap of 5
```

The cap counts items, not headings. "Also worth a look" is still inside the five.

## Delivery

Write `digests/YYYY-MM-DD.md` on every run. That file is the record. Then send it to the
destination in `PROFILE.md` (a notes folder, email, a chat message). A delivery failure
must not lose the digest: the file is written first.

Set `in_digest` to that date on every id the file includes. Those ids are seen. They do not
come back tomorrow.

## Schedule

The full run, once a day, early morning local time: fetch X, then GitHub releases, then
Hacker News, then RSS; seen-item memory; code filters; keep/drop; digest; delivery.

The scheduled command is the engine program. It is not an agent CLI. Do not wrap it in a
coding agent, and do not put permission-skipping flags on the scheduler, the program, or
the model client. `agent_args_override` stays empty unless the user set it themselves
(`references/safety.md`).

Ask before registering the schedule (cron, Task Scheduler, launchd, or a hosted scheduler).
Once it is registered, X and the keep/drop pass bill every day with nobody watching. Show
the one command, and run the engine once by hand if they would rather register it themselves.

Free sources (GitHub, Hacker News, RSS) stay on that same daily run. Do not add an hourly
scan. A second schedule is how caps get multiplied.

Log every run: items per source, dropped per filter, keep and drop counts, model calls,
and wall time.
