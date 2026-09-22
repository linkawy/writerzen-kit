# SOP — running WriterZen from a server

For the person who owns the WriterZen account. Your agent reads SKILL.md; this file is the part
only you can do.

Roughly fifteen minutes the first time, two minutes each time the session expires.

## What you need before you open this

Five things. None of them can be worked around, so check them now rather than halfway through.

- [ ] **A WriterZen account whose plan includes the tools you want.** There is no trial mode here
      and no free tier to fall back on. Step 3 below shows you how to read your own allowances.
- [ ] **A browser you can sign into that account with**, to copy two cookies out of. Step 2.
- [ ] **A machine with Python 3.7 or newer** that can reach `app.writerzen.net` over HTTPS. No
      packages to install — `wz` uses only the standard library.
- [ ] **A writable directory** for the raw JSON every command saves. The default is
      `~/.writerzen/out`.
- [ ] **The market you work in**: a country name and a two-letter language code, e.g. `Germany` and
      `de`. You will set these once in step 1 and never type them again.

And one thing to decide before you run anything: **searching is free, clustering is not.** `wz kw`
and `wz topic` are unlimited and consume nothing countable. `wz plan create` spends one keyword
credit per keyword out of a monthly allowance. Everything else in this kit only reads.

---

## 1. Put the tool somewhere the agent can reach

`wz` is a single Python 3 file with no dependencies. Copy it next to your agent — the same machine
or the same container — and make it executable.

```sh
install -m 755 wz /usr/local/bin/wz     # or anywhere on PATH
wz --help
```

It will complain that there is no session yet. That is step 2.

Four environment variables control everything:

| Variable | Default | What it is |
|---|---|---|
| `WZ_SESSION` | `~/.writerzen/session.txt` | your cookie file |
| `WZ_OUT` | `~/.writerzen/out` | where raw JSON lands |
| `WZ_LOC` | `United States` | the market `--loc` defaults to |
| `WZ_LANG` | `en` | the language `--lang` defaults to |

Set the last two in the agent's environment for the market you actually work in, and nobody has to
remember to type them:

```sh
export WZ_LOC="Germany" WZ_LANG=de
```

---

## 2. Hand it a session

This is the only manual step, and the only one that ever needs repeating.

1. Sign in to WriterZen in your browser as usual.
2. Open DevTools (F12) → **Application** → **Storage** → **Cookies** → `https://app.writerzen.net`.
3. Copy the values of exactly two cookies: **`writerzen_session`** and **`XSRF-TOKEN`**.
4. Write them into the session file, one per line, `name=value`:

```sh
mkdir -p ~/.writerzen && chmod 700 ~/.writerzen
cat > ~/.writerzen/session.txt <<'EOF'
writerzen_session=PASTE_THE_VALUE_HERE
XSRF-TOKEN=PASTE_THE_VALUE_HERE
EOF
chmod 600 ~/.writerzen/session.txt
wz session check
```

`session check` printing your remaining keyword credits means you are done.

### Three things that will waste your afternoon if nobody tells you

- **A cookie-export extension is not enough.** `writerzen_session` is httpOnly, so extensions do not
  export it. It has to come from the DevTools panel above. `wz` refuses a file without it rather
  than letting you discover this through a wall of 500s.
- **Do not try to build a session from the `remember_web` cookie.** It works: pages open, you look
  signed in, and then every single API call answers 500. Only a real `writerzen_session` works.
- **Both cookies must come from the same browser session.** Sign out, and both die together.

### Handling the session

Treat the file the way you would treat a password, because that is what it is — it signs in as you,
with no second factor.

- `chmod 600`, and never commit it.
- Never paste cookie values into a chat window, a ticket, or a log. Anything pasted into a chat has
  left your machine.
- Never share the file with another person. Each person uses their own subscription: sharing a
  session means sharing the account, which breaks WriterZen's terms and hands someone your login.
  Share this kit freely; share the session with nobody.

### When it expires

Sessions die on sign-out, on a password change, and on their own after a while. The symptom is
always the same: `wz` stops with `session has expired`. Redo step 2. Nothing else is wrong, and an
agent cannot fix it for you.

---

## 3. Check what your plan actually includes

```sh
wz limits
```

Match the rows against what you intend to run. Anything showing `0` in the `left` column is not in
your plan, and no amount of retrying will change that.

| Row | Needed for | Healthy looks like |
|---|---|---|
| `keyword_explorer` | `wz kw` | `unlimited` |
| `topic_research` | `wz topic` | `unlimited` |
| `keyword_value` | `wz plan create` | thousands left; one is spent per keyword |
| `keyword_clustering` | `wz plan create` | thousands left |

## 4. First real run

```sh
wz kw "your seed keyword"
printf 'keyword one\nkeyword two\nkeyword three\n' > /tmp/kws.txt
wz plan create --name "First test" --file /tmp/kws.txt
```

Start with a handful of keywords, not a thousand. `plan create` spends one credit per keyword and
gets slower as the list grows; ten keywords cluster in well under a minute.

Every command leaves the full raw JSON in `WZ_OUT`, so nothing you paid for is ever lost to a
summary.

---

## 5. Reading a plan honestly

`wz plan read` prints the clusters. `wz plan matrix` prints the evidence behind them: for every pair
of keywords, how many of their top ten search results are the same page.

Read the matrix like this. Numbers inside a cluster should be clearly higher than numbers across
clusters. In a verified run, within-cluster overlap was 4–8 of ten and across-cluster overlap was
0–1 — an obvious split. A keyword that scores about the same against two clusters is genuinely on
the boundary, and which cluster it landed in is close to a coin toss. That is worth knowing before
you build a content plan on it.

`--level` sets how many shared results are required to group two keywords. 3 is the default. Raise
it for tighter, smaller clusters; lower it for broader ones.

---

## 6. When WriterZen changes and something breaks

Every endpoint `wz` uses was read out of WriterZen's own JavaScript. When they ship a new front end,
a path can move, and you can find the new one the same way. This is the part of the kit worth
keeping.

```sh
# 1. Fetch the page for the tool, using your session, and find its bundle
curl -sS -b cookies "https://app.writerzen.net/user/keyword-planner" | grep -o 'src="[^"]*\.js[^"]*"'

# 2. Download the bundle and list every endpoint it mentions
curl -sS "https://app.writerzen.net/js/app.js?id=..." -o app.js
grep -o 'keyword-clustering/v1/[a-zA-Z0-9/_-]*' app.js | sort -u

# 3. Read the calling code around a path to learn its parameters
grep -o 'fetchLevelClusters:function(t){.\{0,150\}' app.js
```

Three things that recipe taught us, which no amount of guessing would have:

- **Clusters hang off a level, not off a project.** `clusters?task_id=…` answers 500 forever. The
  real route is `tasks/{id}/levels` for a `level_id`, then `clusters/get-by-level?level_id=…`.
- **Creating a project needs the full location and language objects alongside their own ids.**
  Sending the ids alone returns a bare 500 with no message.
- **The three tools disagree about their own field names.** Keyword Explorer takes the seed as
  `input`, Topic Discovery takes it as `keyword`, and the wrong one is — again — an unexplained 500.

A 500 from this API almost never means the server is broken. It means a field is missing, misnamed,
or carrying the other id system's number. Change one thing at a time and try again.

Note also that the app's own pages live under `/user/…`; the bare paths 404.

---

## 7. Two ideas of "country code"

The API carries two different numbering systems and silently 500s if you cross them.

| | Keyword Explorer | Topic Discovery & Keyword Planner |
|---|---|---|
| Field | `criteria_id` (Google Ads constants) | `id` (WriterZen's own rows) |
| United States | 2840 | 235239 |
| English | "1000" | 2 |

The same place and the same language carry completely different numbers in the two systems, and
every market has its own pair. `wz` looks both up by name through `/seo-core/google-geoid/search`
and `/seo-core/google-languageid/get`, which is why you pass `--loc "Germany" --lang de` and never a
number.

`wz` resolves both for you from `--loc` and `--lang`, so you should never need this table. It is
here for whoever debugs a raw call at two in the morning.
