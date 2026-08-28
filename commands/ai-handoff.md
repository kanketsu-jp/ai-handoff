---
description: Start an ai-handoff — interview me first (PREFLIGHT), then write a REQUEST.md addressed to the other person's LLM.
argument-hint: <who you need to hear from, and roughly what about>
allowed-tools: Read Write Edit Bash
---

Run an ai-handoff for: **$ARGUMENTS**

Follow the `ai-handoff` skill. Spec: https://github.com/kanketsu-jp/ai-handoff

🚨 **Start with PREFLIGHT (`templates/PREFLIGHT.md`). Do not write `REQUEST.md` yet.**

Work through the checklist **yourself first** and sort every item into:

- **I can already answer this** → say your answer, let me correct it. Do not ask
- **Does not apply here** → 🚨 skip it, and tell me in one line that you skipped it and why
- **Only I can answer this** → ask me, one at a time

Then ask only that last group, one question at a time, short — the question plus the one or
two lines that bear on it. Offer depth once rather than front-loading it.

🚨 The two you almost certainly cannot answer for me:

- what is **already settled** (must not be re-asked)
- what the recipient **already has** (minutes, past messages, drafts)

When we are done, write `REQUEST.md`, then show it to me before anything is sent.
