# Signal engine profile

<!-- One person. Personal AI/dev news digest. Written by the build-signal-engine
     interview. Every stage reads this file. Edit it freely; the engine follows it. -->

## Who this is for

One person.

Focus: AI and software-development news I would act on this week.

What I would do differently if I knew on the day:

## Sources

Order is fixed: X, then GitHub releases, then Hacker News, then RSS.
Do not add sources.

### X

- On: yes
- Accounts:
- Search query (optional):
- Exclude replies and reposts: yes
- Per-account cap: 50
- Search cap: 50

### GitHub releases

- Repos:
- Skip drafts: yes
- Skip prereleases: yes
- Per-repo cap: 20

### Hacker News

- Points floor: 50

### RSS

- Feeds:
- Per-feed cap: 20

## Code filters

Facts only. Each one is a number or a flag in the engine.

- Max age: 7 days
- Already seen: drop (seen-item memory is on)
- Engagement floor: the Hacker News points floor above. No floor on X, GitHub, or RSS.
- Empty title or url: drop

## The keep/drop questions

<!-- One thing per question. Written from the examples below, in my words.
     The cheap model answers both. Code drops when either answer is no. -->

1. Relevant to my AI/dev work (yes/no):
2. Reports something new and concrete (yes/no):

A failed model call is not a drop.

### Keep examples

<!-- Three things I would have wanted, with a link if I have one. -->

-
-
-

### Drop examples

<!-- Three things I never want to see again. -->

-
-
-

## The digest

- Where I read it:
- Length cap: 5
- Format: grouped items, one or two sentences each, every link kept
- Delivery file: `digests/YYYY-MM-DD.md`
- Full run: daily, early morning, local time

## Models

- Keep/drop: the cheapest model that returns structured JSON reliably
- Digest: one call per run

## Store

- SQLite path: a gitignored file in the repo
- Seen-item memory: on

## Safety

- `agent_args_override`:

<!-- Leave this empty. Empty, missing, or whitespace means off: agent commands stay at
     the tool default and permission checks stay on. The build must not fill this in.
     I edit it myself if I want extra agent arguments appended. It is arguments only,
     not a switch for model safety. Model safety stays at the provider default. -->
