# The cheap-model keep/drop pass

Every new item that survived the code filters gets one cheap-model call. The call answers
the questions in `PROFILE.md`. Code turns the answers into keep or drop. The digest model
never sees a drop, and it never sees an item the store already remembers as decided.

Use the cheapest model the user can call that returns structured JSON reliably. Do not put
a larger model on this pass. Do not add a second decision service.

Invoke it the way `references/safety.md` requires: provider safety checks on, no skip flag,
no safety-disable field.

## Write the questions from their examples

Start from the focus line and the keep/drop examples in `PROFILE.md`. The questions must
separate those examples. Read them against each example before running anything.

**One thing per question.** "Is this a new and relevant release?" is two questions, and the
answer is a blur. The profile has two:

- **Relevant to this person's AI/dev work:** yes or no. An unfamiliar product or model name
  is not a reason to answer no, or every new launch gets buried.
- **Reports something new and concrete:** yes or no. A release, a feature, a break, a
  measured result, a technique, a documented incident. Opinions, questions, jokes, memes,
  reactions, and rumors are no, even when they name a real product.

Taste ("toy", "would I enjoy this", "is it good") is not a question on this pass. It can
influence which ten the digest keeps. It is never a reason to drop here.

**Keep numbers out of the questions.** Points, dates, and counts are code. The model is
unreliable at arithmetic.

**Send only the fields the decision needs:** title, source, the first few hundred characters
of the body, and the URL's domain.

## What the model returns

One item per request. Ask both questions in that request. Require structured output:

```json
{"relevant": true, "new": true}
```

Booleans, not probabilities. A cheap chat model is not a calibrated scorer, and a fake
probability becomes a threshold nobody can defend.

## Route in code

```
drop    relevant is no, OR new is no
keep    relevant is yes, AND new is yes
```

There is no taste route and no score cutoff.

If the call fails, times out, or returns JSON that does not parse, the item gets no
decision and is not dropped. It stays eligible for the digest only when there are fewer
keeps than the digest cap. The keep/drop pass must never be a single point of failure, and
it must never drop on an error.

Store the decision on the row. The next run does not call the model for that id again.

## Run it in parallel, inside a cap

Run items concurrently with a small cap (8). Sequential calls over a few hundred items take
many minutes. One item per request: several items in one prompt make them interact.

Retry a 429 or a 5xx a small fixed number of times with backoff. Then treat the item as no
decision, as above. Honor the profile's per-run cap on keep/drop calls. This pass runs on
the daily schedule, not on a frequent scan.

## Prove it on their examples

Before the first digest:

1. Score every keep and drop example in `PROFILE.md`. Print each with both answers and the route.
2. Take a real sample from today's new rows (up to 50), label it with the user, and score that too.
3. Report how many dropped, how many kept, and every item they wanted that was dropped.

Fix misses by rewording the question first. A miss that is really a policy disagreement
("it dropped a consumer app launch because the question is about tools I use") is their
decision: offer to widen the question. Do not "fix" it by weakening the route in code.

## Measure it on a throwaway database

Run the same stored ids through the digest twice, once with every survivor marked keep and
once with the pass on. Report how many rows the digest model read, and which rows it would
have included that the pass dropped. Expect the digest model to vary between the two runs
on its own; do not count those rows against the pass.

The pass saves on reading. It does not save on the cost of fetching X.
