# The profile interview

The diet is fixed: one person, AI and dev news, X then GitHub releases then Hacker News
then RSS, seen-item memory, code filters, a cheap-model keep/drop pass, a five-item daily
digest. The interview fills in what only this person knows. It does not reopen the diet.

**Ask about things that already happened. Never ask them to design the filter.** They do
not write the decision questions or the pipeline. They name accounts, repos, feeds, and
examples. The agent turns the examples into the questions, then plays the profile back.

## Rules for every question

Use the host's question tool when the session has one. Each question carries a free-text
answer, so nothing is lost when the drafts are wrong.

- Two to four options, one marked `(Recommended)` and first, its reason in one line.
- Draft options from what they already said. Before the first question, the only material
  is their opening message.
- Questions go one at a time, in order.
- A question that wants a list is a multi-select of drafted candidates plus free text.
  What they deselect is as useful as what they keep.
- Reflect each answer back as the concrete thing it became, in one line, then move on.

After the last question: "One message of defaults, then I start building."

---

## Questions

### 1. "Which X accounts should this read, and is there one search you actually check?"

**What it becomes:** the X section of `PROFILE.md`. X is built first and fetched first.

Draft handles from tools and people they already named. Recommend a short list of accounts
they already read over a broad search. Search is optional and capped; an empty search is
the right default when they live in a few timelines.

Replies and reposts stay excluded unless they say they want those. Reposts are usually the
second copy of something the other sources will also carry.

### 2. "Which GitHub repos' releases would change your week?"

**What it becomes:** the repo list. Draft repos from the tools they use. Find the
`owner/repo` yourself when they name a project. Do not ask them for API URLs.

Releases only. Not issues, not commits, not trending.

### 3. "Which feeds do you already subscribe to for this?"

**What it becomes:** the RSS list. Draft up to four feeds from what they said they read.
Free text is where the real list arrives. An empty list is allowed: the RSS stage stays in
the pipeline and fetches nothing until a URL is added. Do not invent a long default pack.
Do not subscribe to Hacker News here; HN is its own source.

### 4. "From the last two weeks, which three items did you want, and which three do you never want again?"

**What it becomes:** the keep and drop examples, and the wording of the two keep/drop
questions. This is the most valuable answer in the interview.

Ask for their own examples in the free-text answer. Offer two or three plausible drafts
from earlier answers as options they can accept. Only if they cannot name any, pull a
sample after Stage 2 and have them sort it then. Never mark that fallback as recommended,
and note it in `PROFILE.md`.

**Push back on:** examples that are all one kind. If every keep is a model release, ask for
one that is not, or the question will only ever keep model releases. If an example is a
judgment ("too basic", "toy"), it belongs in the question, not in a code filter.

### 5. "Where do you read something in the morning, and what do you do with an item you care about?"

**What it becomes:** delivery. "A markdown note" means `digests/YYYY-MM-DD.md` plus a copy
into that notes folder. "Email" or "a chat message" is a second destination. The file is
always written first.

---

## Defaults, in one message

Start from `templates/PROFILE.md`. Put their answers in. Then one message, every remaining
setting as already filled, and ask what to change:

| Setting | Default |
|---|---|
| Reader | one person |
| Focus | AI and software-development news they would act on this week |
| Source order | X, GitHub releases, Hacker News, RSS |
| X | named accounts, no search unless they gave one, replies and reposts excluded, 50 posts per account, 50 for search |
| GitHub | releases API, 20 per repo, drafts skipped, prereleases skipped |
| Hacker News | stories with at least 50 points; the keep/drop pass judges AI/dev relevance |
| RSS | the feeds they named, 20 items per feed |
| Max age | 7 days |
| Seen-item memory | on |
| Keep/drop model | the cheapest model they can call that returns structured JSON reliably |
| Digest model | one call per run |
| Digest | 5 items, every link kept |
| Delivery | `digests/YYYY-MM-DD.md`, then their destination from question 5 |
| Schedule | daily, early morning, local time |
| Stack | Python, unless the repo already has a language; SQLite, gitignored |
| `agent_args_override` | empty (off) |

Do not raise the digest cap. Do not add a source. Do not put anything in
`agent_args_override`.

## One breath, if they ask

- **Seen-item memory:** the same item always gets the same id, and once it has been stored
  as decided, duplicated, or digested, later runs leave it alone. Without it, every morning
  is a full re-read.
- **Keep/drop pass:** a cheap model answers two yes/no questions on each new item. Code
  turns the answers into keep or drop. It writes no digest.
- **`agent_args_override`:** extra arguments for an agent process. Empty means permission
  and safety checks stay on. The details are in `references/safety.md`.

## Closing

Write `PROFILE.md`, read it back as a proposal in one short message, and ask one question:
"Build it like this?" with options to build, change something, or explain a part. Then
Stage 1.

**Narrow, do not widen, when:**

- They ask for everything in tech. The digest is AI/dev, five items. Ask once what they
  would do differently if they knew on the day, and write that line into the profile.
- They name Reddit, YouTube, Discord, papers, a news search, or a second person. Those are
  outside this skill. Say so.
- They have no X accounts and no X search. Ask once. If they still do not want X, record
  X as off and build the other three in the same order. Do not replace it with a source
  this skill does not have.
