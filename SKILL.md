---
name: writerzen
description: Use WriterZen's Keyword Explorer, Topic Discovery and Keyword Planner from a shell through the `wz` command — keyword volumes, the headlines that rank, and SERP-overlap keyword clusters. Also covers which parts of WriterZen's own analysis to ignore, especially in non-English languages.
---

# WriterZen through `wz`

WriterZen is a keyword research suite with no public API. `wz` talks to the same JSON endpoints its
own front end uses, carrying a browser session the account owner exported. You never touch the
session file; you only call `wz`.

## Before the first run

Run these two, in this order, before promising anyone any research:

```sh
wz session check     # is this machine signed in at all
wz limits            # does this account actually include what you are about to use
```

`session check` has to print a line about keyword credits. `limits` has to show a non-zero `left`
for the module you need: `keyword_explorer` for `wz kw`, `topic_research` for `wz topic`, and both
`keyword_value` and `keyword_clustering` for `wz plan create`. A plan that does not include a tool
reports zero, and no amount of retrying changes that.

If either fails, stop and say so. Everything below depends on four things the operator provides,
and you cannot supply any of them yourself:

| What | Why | How you know it is missing |
|---|---|---|
| A WriterZen account with the tools in its plan | There is no free tier to fall back on | `wz limits` shows 0 left |
| A live browser session in the session file | The API has no other way in | `wz session check` says expired |
| Network reach to `app.writerzen.net` | — | connection errors, not HTTP errors |
| Python 3.7 or newer, and a writable `WZ_OUT` | `wz` saves every raw response | it fails on the first call |

**Never ask anyone to paste a cookie or a token into the conversation.** Refreshing the session
means the operator edits a file on their own machine; SOP.md step 2 walks them through it. If the
session is dead, the correct move is to say "the WriterZen session has expired, SOP.md step 2" and
stop — not to look for another route in.

## Two things worth asking the person before you start

1. **Which market and language?** `--loc` and `--lang` default to `$WZ_LOC` / `$WZ_LANG`, which
   somebody set for a market that may not be theirs. Confirm it rather than assume it, and say in
   your answer which market the numbers describe.
2. **May you spend credits?** `wz plan create` costs one keyword credit per keyword and is the only
   command here that costs or writes anything. For a list of any size, quote the number first.

## The three tools, and what each is actually good for

| Command | Tool | Use it for |
|---|---|---|
| `wz kw SEED` | Keyword Explorer | Search volume, CPC and competition for a seed and its ideas |
| `wz topic SEED` | Topic Discovery | The real page titles and URLs that rank, plus related phrases |
| `wz plan create` | Keyword Planner | Grouping a keyword list by how much its search results overlap |

```sh
wz session check
wz limits
wz kw "standing desk"
wz kw "standing desk" --loc "Germany" --lang de --top 40
wz topic "project management software" --top 30
wz plan create --name "Desk buying guides" --file keywords.txt --level 3
wz plan read 179403
wz plan matrix 179403
wz kw "standing desk" --xlsx out.xlsx      # any table can also become a spreadsheet
```

`--loc` takes a country or city name and `--lang` a two-letter code; both are resolved against
WriterZen's own lists at call time, so you never need a numeric code. They default to `$WZ_LOC` and
`$WZ_LANG`, which the operator sets once for the market they work in — check those before assuming
a result is about your own country.

## What to throw away

WriterZen layers its own analysis on top of good raw data. Several of those layers are measurably
broken outside English, and an agent that quotes them will mislead whoever reads the answer.

- **Topic Discovery's topic ranking is unusable in a language it handles poorly.** It splits the
  phrase into fragments and reports each fragment's own search volume as a topic's. In one measured
  run the "top topics" included a bare preposition and a business from an entirely unrelated
  industry. `wz topic` therefore drops the topics and their volumes by default and gives you the
  headlines and phrases underneath, which are excellent.
- **Its relevance score does not discriminate** — 40 of 100 topics in that run scored an identical
  87, the unrelated business among them. Never rank by it.
- **`allintitle` and KGR are dead outside English.** Every non-English keyword tested came back with
  `allintitle = 0` and `kgr ≈ 1.0`, which reads as "every keyword is a golden opportunity".
  `wz plan` omits both columns on purpose. Do not go into the raw JSON to fetch them.
- **The AI name a cluster gets is generic.** Two clearly different clusters in one test were handed
  the same label. Use the cluster's highest-volume keyword as its name instead.
- **Domain Authority and content briefs come back empty** unless the account pays for those add-ons.
- **CPC is not the same unit in both tools.** For three keywords present in both, Keyword Explorer's
  `cpc` came back a consistent ~23,000x the dollar figure Keyword Planner reported. Rank by
  Explorer's number if you like, but never call it money and never put the two in one table.

## What holds up

- **Volumes match a paid data provider.** One seed returned 165,000 from both WriterZen and
  DataForSEO for the same country and language.
- **Clustering is SERP-overlap, so the language never enters into it.** `wz plan matrix` prints the
  shared-URL counts behind the grouping, which WriterZen's own interface does not show. In a
  verified run, keywords inside a cluster shared 4–8 of their top ten results while keywords across
  clusters shared 0–1. That matrix is how you check a cluster instead of trusting it.
- **Keywords with no search volume still cluster correctly**, because the grouping reads results
  rather than numbers. Two keywords another tool reported as `n/a` landed in the right clusters,
  sharing 6–8 results with their cluster head.

## Cost

`wz kw` and `wz topic` are unlimited per day and consume nothing countable. `wz plan create` spends
one keyword credit per keyword from a monthly allowance (`wz limits` shows what is left). It is the
only command here that writes anything or costs anything — everything else reads.

Keep the pace human. This is one person's own subscription, not a data feed.

## When it breaks

WriterZen ships front-end changes without notice, and an endpoint can move. SOP.md's last section
is a short recipe for re-deriving the endpoints from their own JavaScript bundle — it is how every
path in `wz` was found in the first place, and it takes about ten minutes. Read it before deciding
the tool is dead.
