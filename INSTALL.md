# Giving this to your agent

`wz` is the whole interface. Any agent that can run a shell command can use it, so the only thing
to install is the knowledge of when to reach for it.

Do step 1 and 2 of SOP.md first, then pick whichever of these matches your setup.

## Claude Code

Drop `SKILL.md` in as a skill and it loads itself when a keyword-research task comes up:

```sh
mkdir -p .claude/skills/writerzen
cp SKILL.md .claude/skills/writerzen/SKILL.md
```

## Codex, or anything that reads AGENTS.md

Add one line, and keep `SKILL.md` where it points:

```md
- Keyword research runs through the `wz` command. Read `docs/writerzen/SKILL.md` before using it.
```

## An always-on agent with its own instructions file

Paste this into it. It is deliberately short: the detail belongs in SKILL.md, which the agent can
read when it needs it, not in a system prompt it carries forever.

```md
## Keyword research

`wz` gives you WriterZen's Keyword Explorer, Topic Discovery and Keyword Planner:
`wz kw SEED`, `wz topic SEED`, `wz plan create --name N --file keywords.txt`,
`wz plan read ID`, `wz plan matrix ID`. Run `wz session check` first; if it fails,
say so and stop — only the account owner can refresh the session.

Read SKILL.md before the first use. It lists which parts of WriterZen's own analysis
are wrong outside English — its topic ranking, its relevance score, allintitle and KGR —
and those must never be quoted as findings.
```

## Checking it landed

Ask the agent to run `wz limits` and report what the account has left. If it comes back with the
numbers, it can reach the tool. If it comes back describing what the command would do, it has not
actually run anything — point it at SKILL.md again.
