# Using WriterZen through `wz` — a working guide

This is the short version of how to actually get something out of the three tools. It assumes `wz`
is installed and signed in; if `wz session check` fails, that is SOP.md's job, not this file's.

One idea underneath everything here: **WriterZen gives you excellent raw material and mediocre
conclusions.** The volumes, the ranking pages and the SERP-overlap clustering are real. The topic
labels, the relevance score and the "golden keyword" columns are not, especially outside English.
So the shape of every task below is: let the tool gather, and do the judging yourself.

---

## The loop

Six steps. Two of them are free, one costs credits, and one is you.

```
 1. check      wz session check && wz limits
 2. widen      wz kw "seed"                       free
 3. narrow     you, with your own judgment        free
 4. cluster    wz plan create --file list.txt     1 credit per keyword
 5. verify     wz plan matrix ID                  free
 6. enrich     wz topic "cluster head"            free
```

### 1. Check

```sh
wz session check
wz limits
```

Thirty seconds, and it saves you from discovering halfway through a plan that the account is signed
out. Confirm the market too — `--loc` and `--lang` default to whatever the operator set, which may
not be yours.

### 2. Widen

One seed at a time, two or three seeds for a topic area.

```sh
wz kw "standing desk" --top 40
wz kw "standing desk" --loc "Germany" --lang de
```

You get every idea WriterZen has for that seed with volume, competition and CPC. A seed usually
returns over a thousand ideas, so `--top` only limits what prints; the full set is saved as JSON.

Read `volume` and `competition`. Ignore `cpc (raw)` as money — it is in a unit ~23,000× the dollars
the planner reports, useful only for ranking within this one table.

### 3. Narrow

**This step is yours and the tool cannot do it.** Out of a thousand ideas, most are irrelevant,
off-market, or another company's brand. Cut them down to the twenty or fifty you would genuinely
write for, by whatever rule fits your business.

If you have a vocabulary of terms your customers actually use, apply it here. If not, read the list
and delete. It takes ten minutes and it is the difference between clustering something meaningful
and spending credits on noise.

```sh
# whatever produces one keyword per line
$EDITOR keywords.txt
```

### 4. Cluster

```sh
wz plan create --name "Desk buying guides" --file keywords.txt
```

One credit per keyword, so a 50-keyword list costs 50. Ten keywords finish in well under a minute;
a few hundred take a while. Start small the first time.

What comes back is groups of keywords that share search results — which is to say, keywords Google
thinks are the same question. **Each group is one page you should write, not one keyword.**

### 5. Verify before you trust it

```sh
wz plan matrix 179404
```

This is the step people skip, and it is the most useful one. It prints how many of the top ten
results each pair of keywords actually shares:

```
                              1   2   3   4   5   6
  1. ecommerce platform        9   5   1   1   2   2  A
  2. best ecommerce platform   5   8   1   2   2   2  A
  3. shopify pricing           1   1   8   1   1   2  -
  4. online store builder      1   2   1  10   1   1  -
  5. headless ecommerce        2   2   1   1   8   1  -
  6. shopify vs woocommerce    2   2   2   1   1   7  -
```

Read it like this: keywords 1 and 2 share 5 of their ten results, so they are one page. Everything
else shares 1–2 with everything else, so those are four separate pages, not a cluster that failed.
That is a real answer, not a shrug.

The warning sign is a keyword scoring about the same against two different clusters — it sits on a
boundary, and which group it landed in was close to a coin toss. Decide that one yourself.

### 6. Enrich

For each cluster, take its highest-volume keyword and ask what already ranks:

```sh
wz topic "ecommerce platform" --top 30
```

You get the real headlines of the pages that rank, their URLs, and the phrases around them. That is
your brief: what the existing top results promise, in their own words. Ignore the topic labels and
the relevance score that WriterZen prints on top of this; they do not survive contact with a
non-English language, and they are not needed even in English.

---

## A short worked example

Six keywords about ecommerce platforms, United States, English.

```sh
wz limits                                        # keyword_value: 49,735 left
printf '%s\n' "ecommerce platform" "best ecommerce platform" "headless ecommerce" \
  "shopify pricing" "shopify vs woocommerce" "online store builder" > k.txt
wz plan create --name "Kit example" --file k.txt  # 6 credits
```

Result: **one cluster of two, and four keywords standing alone.**

| Cluster | Keyword | Volume | Competition |
|---|---|---|---|
| ecommerce platform | ecommerce platform | 14,800 | 10 |
| ecommerce platform | best ecommerce platform | 1,000 | 43 |
| — | shopify pricing | 9,900 | 42 |
| — | online store builder | 1,600 | 17 |
| — | headless ecommerce | 590 | 36 |
| — | shopify vs woocommerce | 590 | 28 |

`wz plan matrix` (above) confirms it: the two in the cluster share half their results, and nothing
else shares more than two. So this is a five-page plan, not a six-keyword page — and `shopify
pricing`, at 9,900 searches with almost no overlap with anything, is its own page and probably the
first one to write.

Total spend: six credits and about four minutes.

---

## Decision rules worth knowing

**`--level` is the grouping threshold** — how many shared results two keywords need to land
together. It defaults to 3. Raise it to 5 for tight, small clusters when you have hundreds of
keywords; drop it to 2 when everything is coming back unclustered and you want broader groups.

**Batch size.** Ten to fifty keywords is comfortable. Beyond a few hundred it gets slow and the
credit cost stops being trivial. Split by topic area rather than sending everything at once.

**Keywords with zero volume still cluster correctly**, because the grouping reads search results,
not numbers. Do not delete them in step 3 just because another tool reported `n/a` — two such
keywords in our test landed in exactly the right clusters.

**Every command saves its full raw JSON** in `$WZ_OUT`. Nothing you paid for is lost to a summary,
so you can re-read a plan months later without spending again.

---

## What never to report as a finding

Four things WriterZen prints that look like insight and are not:

- **`allintitle` and KGR** — 0 and ~1.0 for every non-English keyword tested. Reads as "every
  keyword is a golden opportunity". It is a broken field, not an opportunity.
- **Topic Discovery's topic ranking** — it splits a phrase into fragments and reports each
  fragment's own volume. In one run the "top topics" included a bare preposition and a business
  from an unrelated industry.
- **Its relevance score** — 40 of 100 topics in that run scored an identical 87.
- **The AI name given to a cluster** — two clearly different clusters were handed the same label.
  Use the cluster's highest-volume keyword as its name instead.

And one that is merely a trap rather than broken: **CPC is not the same unit in both tools.** Never
put Explorer's `cpc (raw)` and the Planner's dollars in one table.
