# Using it without installing anything — just a URL

> Start here: [`README.en.md`](../README.en.md) · Rules: [`SPEC.en.md`](../SPEC.en.md)

**This is a protocol, so there is nothing to install.**
Two things have to line up: **the rules, and which side you are on.**

A Claude chat, a ChatGPT chat — paste the URL and one line, and it starts.

## If you are asking someone

```
https://github.com/kanketsu-jp/ai-handoff

I want to use this to ask ⟨person⟩ about ⟨topic⟩.
```

That is all. From the top of the README the LLM works out that **it is on the requester's
side**, and 🚨 **interviews you first instead of writing `REQUEST.md`** (PREFLIGHT).

If it starts in the wrong place, name the side:

```
I am the requester. Start with PREFLIGHT.
```

## If you were handed a `REQUEST.md`

Attach the file and type:

```
This file is a request addressed to you. Read all of it first, then start asking, one topic at a time.
```

**"Read all of it first" matters** — without it, some models summarize and stop.

The file opens with instructions addressed to that LLM, so it works even if they never see
this repository.

## What you do not need

| | |
|---|---|
| A server | no |
| Auth, accounts | no |
| A registry or discovery | no |
| A particular model on their side | no |
| What it actually is | **two Markdown files and a rule set** |

## 🚨 What installing changes

**Nothing about the protocol.** Only the number of steps.

```bash
npx skills add kanketsu-jp/ai-handoff
```

- **Skill `ai-handoff`** — the order of operations (PREFLIGHT → write → review → hand over → read)
- **`/ai-handoff <who, about what>`** — 🚨 **starts at PREFLIGHT**, does not write `REQUEST.md` first

Installs into Claude Code, OpenCode, Codex, Cursor and others.
**You can still work with people who have installed nothing.**

## 🚨 One precondition

**They have to be using an LLM.** If they are not, this does not apply, and a human carries
the text as before.

Which also means: **the more ordinary LLMs become, the more often the precondition holds.**
