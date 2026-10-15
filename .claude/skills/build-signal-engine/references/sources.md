# Sources, seen-item memory, code filters

Four sources, in this order, each mapped to `templates/item_schema.md`. Build X end to end
into the store before adding the next. Prove each with a real run: how many items came
back, and a second run that adds roughly zero.

Caps, the age limit, the Hacker News points floor, and the draft and prerelease switches
are the numbers in `PROFILE.md`. Do not invent a second set.

Use each source's official interface. Do not scrape the site, do not use an unofficial
client, and do not read private or protected content.

## X (first)

The primary source. Fetch it before the other three so a story that also shows up as a
release or a feed item is remembered as the X post.

- **Method:** the official X API, with the user's own credential in the gitignored env
  file. For each handle in `PROFILE.md`, that account's recent original posts. If they set
  a search query, one recent-search call for that query.
- **Leave out, unless the profile says otherwise:** replies, reposts, quote-only
  duplicates of a post already stored. Never DMs, bookmarks, likes, or protected posts.
- **Ids:** `x:` plus the post id.
- **URL:** `https://x.com/<handle>/status/<id>`. Also collect outbound URLs from the post
  for seen-item memory (below).
- **Fields:** text as the body (capped), `created_at` as `published_at`, public like count
  plus repost count as `engagement`. Title is the first line of the text, or the handle
  plus that line when the text has no title.
- **Caps:** the per-account cap and the search cap from the profile, whichever is smaller
  on a given call. Stop the run on a credit or rate-limit error. Do not retry in a loop.
- **Known failures:** recent search only covers a short window, so an old post will not
  come back through search; account timelines are how a named handle is read. A busy
  account's replies will dominate the cap if the exclude filter is wrong. Check one real
  payload and confirm replies and reposts are absent before Stage 2.
- **If the profile says X is off:** do not invent accounts. Stage 1's proof is that the
  store exists and X fetched nothing. Then build the other three in the same order.

## GitHub releases

- **Method:** `GET /repos/{owner}/{repo}/releases` on the GitHub REST API. Not the releases
  HTML page. A token in the env file raises the rate limit; without one, stay inside the
  unauthenticated limit and say so.
- **Ids:** `gh:` plus `owner/repo` plus `:` plus the numeric release id.
- **Fields:** title is repo, tag, and release name. Body is the release notes, capped.
  `published_at` is the API's `published_at`. URL is `html_url`.
- **Skip drafts.** Skip prereleases when the profile says so (the default). Do not store
  those skips: publishing a draft, or clearing the prerelease flag, reuses the same release
  id, and a stored skip would hide the real release. The next fetch sees the latest page
  of releases and can admit it.
- **Cap:** the per-repo cap from the profile. A repo that returns an empty list is a valid
  run, not a failure.
- **Known failure:** pulling full release history on the first run floods the store. The
  cap plus the age filter is what prevents that.

## Hacker News

- **Method:** the official Firebase API or the official Algolia API. Stories only, not
  comments. Apply the points floor in the query when the API supports it, so under-floor
  stories are never downloaded into the store.
- **Ids:** `hn:` plus the story id.
- **URL:** the outbound article URL when the story has one, with the discussion URL in
  `metadata`. Text posts use the discussion URL.
- **Engagement:** points.
- **Do not store stories under the points floor.** Points rise after publication. A story
  that is at 10 this morning can clear the floor tomorrow and should still be eligible
  while it is inside the age cap. Storing the skip would mark it seen.
- **Known failure:** treating the front page as the query. The front page is not an AI/dev
  filter. Relevance is the keep/drop pass, not a keyword list. Keywords bury a new tool
  name the list does not contain.

## RSS

- **Method:** each feed URL in the profile, fetched over HTTP. Send stored `ETag` and
  `Last-Modified` when the previous run saved them.
- **Ids:** `rss:` plus a hash of the item's guid, or of its link when there is no guid.
- **Fields:** title, link, the entry summary or content capped as the body, the entry's
  published date.
- **Cap:** the per-feed cap from the profile. A dead feed is reported and skipped. It does
  not fail the run.
- **Known failure:** a feed that republishes the same article under a new guid every day.
  The URL key below is what catches that.

## Seen-item memory

One SQLite table of items, primary key `id`. One row per source with `last_fetched` and the
HTTP validators for RSS. The live database is the memory. Tests use a throwaway file.

Also store `url_key` values for the item URL and, for X, each outbound URL: lowercase, no
trailing slash, tracking parameters (`utm_*` and the same kind of click id) removed. X is
fetched first, so the X post is the row that later copies match.

| Outcome | Stored? | Later runs |
|---|---|---|
| Inserted, decision still null | yes | not fetched again; the next run decides it |
| Keep or drop decision | yes, decision saved | left alone |
| Included in a digest | yes, `in_digest` set | left alone |
| Same `url_key` as a row already stored | yes, `filtered_by=duplicate` | left alone |
| Older than the max age | yes, `filtered_by=age` | left alone |
| Under the engagement floor | no | eligible again if it clears the floor inside the age cap |
| GitHub draft, or a skipped prerelease | no | eligible if that release is later published as a full release |
| Empty title or empty URL | yes, `filtered_by=empty` | left alone |

X is fetched first, so an X post normally wins the `url_key`. If that fetch fails, a later
source may store the story first. That is still one story. Do not delete the row to make
X win tomorrow.

Check ids and URL keys before any paid call and before any model call. A second run of a
finished morning adds nothing and calls no model.

The daily run does not re-decide history. Changing the question does not resurrect old
rows. A manual redecide may clear `decision` only on rows that were kept or dropped, have
`in_digest` null, and `filtered_by` null. It does not delete ids.

## Code filters

Filters run on rows just inserted, before the model, and they drop by fact. Report how many
each one dropped.

- **Already seen,** including a matching `url_key`.
- **Max age,** from `published_at`. A missing date is not an age drop; the keep/drop pass
  judges that item.
- **Engagement floor,** only where the profile sets one. The default floor is the Hacker
  News points floor, applied at fetch as described above. X, GitHub, and RSS have no floor
  unless the profile adds a number.
- **GitHub drafts, and prereleases when the profile skips them.** Applied at fetch, not
  stored.
- **Empty title or empty URL.**

Language, vibe, and "too basic" are not filters. They belong in the keep/drop questions.

Pull less before filtering harder. A filter does not refund an X row that was already billed.
Prefer the account list over an open search, and keep the caps small.

## Proof for Stages 1-2

For each source, one line: how it is pulled, items returned, items stored, and the count on
an immediate second run (0 or close to it). For X, add whether the run stopped inside its
cap. Stage 1 is done only when X has stable ids and the re-run adds nothing.
