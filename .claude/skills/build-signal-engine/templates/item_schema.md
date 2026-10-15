# The item shape

Every source maps to this before it is stored. Nothing downstream knows which source an
item came from except through these fields.

| Field | Type | Notes |
|---|---|---|
| `id` | string | **Stable.** `x:<post id>`, `gh:<owner>/<repo>:<release id>`, `hn:<story id>`, or `rss:` plus a hash of the guid (or the link when there is no guid). Same item, same id, every run. |
| `source` | string | `x`, `github_releases`, `hn`, or `rss` |
| `title` | string | One line. X: the first line of the post. GitHub: repo, tag, and release name. |
| `url` | string | Where a human reads it. Hacker News uses the article URL when there is one. |
| `url_key` | string or list | Lowercased URL, no trailing slash, tracking parameters removed. X also records outbound URLs. A later item with the same key is a duplicate. |
| `body` | string | Text, capped (a few thousand characters). Not a full thread. |
| `published_at` | datetime or null | From the source. A null date is not an age drop. |
| `engagement` | number or null | X: likes plus reposts. HN: points. Null on GitHub and RSS unless the feed provides one. Used only by code filters. |
| `metadata` | object | Handle, repo, feed URL, Hacker News discussion URL, and the raw source id. |

Stored alongside, by the pipeline. These fields are the seen-item memory.

| Field | Set by |
|---|---|
| `first_seen_at` | the insert |
| `filtered_by` | `duplicate`, `age`, `empty`, or null. Drafts, skipped prereleases, and under-floor stories are not stored. |
| `decision` | `keep`, `drop`, or null when the cheap-model call failed |
| `decision_answers` | the two booleans, when the call succeeded |
| `in_digest` | the date of the digest that included it, or null |

An id that is stored is seen. The daily run does not re-fetch it into the model and does
not put it in a second digest.
