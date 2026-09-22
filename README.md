# writerzen-kit

WriterZen's keyword research from a shell, so an agent can use it.

> Unofficial, and not affiliated with or endorsed by WriterZen. It drives your own account through
> the same JSON endpoints their own web app uses, on your own subscription. Nothing here bypasses
> payment, authentication or rate limits, and none of it will survive a determined redesign of their
> front end — `SOP.md` tells you how to find the new paths when that happens.

WriterZen has no public API. Its own front end talks to an ordinary JSON one, and `wz` — a single
Python file, no dependencies — speaks to that one the same way, carrying a session you export from
your own browser.

```sh
wz session check
wz kw "standing desk" --loc "Germany" --lang de
wz topic "project management software"
wz plan create --name "Desk buying guides" --file keywords.txt
wz plan matrix 179403
```

Read them in this order:

| File | For |
|---|---|
| `SOP.md` | Setup. Install, hand it a session, and fix it when WriterZen changes. Once |
| `GUIDE.md` | The actual work: the six-step loop from a seed keyword to a page plan, with a worked example. Every time |
| `SKILL.md` | Your agent. What each tool gives, and which parts of WriterZen's analysis to ignore |
| `INSTALL.md` | Wiring `SKILL.md` into Claude Code, Codex, or your own agent |
| `examples/` | Real output, so you know what to expect before spending a credit |
| `docs/ar/` | The same guide as an Arabic web page, for readers who want it that way |

`GUIDE.md` stands on its own — it is the one to send to whoever will do the research.

## What this is careful about

WriterZen's raw data is good and some of the analysis on top of it is not — its topic ranking, its
relevance score, and `allintitle`/KGR all fail outside English, in ways that look like findings
rather than like errors. In one measured run its "top topics" included a bare preposition and a
business from an unrelated industry, each carrying a confident relevance score. `wz` drops those
columns by default and `SKILL.md` says why, so an agent does not quote one back at you.

It also prints something WriterZen's own interface does not: `wz plan matrix` shows how many search
results each pair of keywords actually shares, which is the evidence behind a cluster and the only
way to catch a keyword sitting on a boundary.

## Before you start

You need your own WriterZen subscription. The session file signs in as you, with no second factor —
`chmod 600`, never commit it, never paste it into a chat, and never share it with another person.
Share this kit; share the session with nobody.

Everything here reads except `wz plan create`, which makes a project in your own workspace and
spends one keyword credit per keyword.
