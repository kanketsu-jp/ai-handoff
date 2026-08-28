---
name: ai-handoff
description: Get information out of someone without sending them a questionnaire. Write a REQUEST.md addressed to THEIR LLM, have it interview them, and get back a BRIEF.md you can act on directly. Use when you need requirements, facts, or decisions from a client, a colleague, or another team — especially when questionnaires have gone unanswered before. Spec: https://github.com/kanketsu-jp/ai-handoff
argument-hint: [new|review|brief] <who you need to hear from>
allowed-tools: Read Write Edit Bash
---

# ai-handoff

**Do not send a person a list of questions. Send their LLM a way of asking.**

Spec: https://github.com/kanketsu-jp/ai-handoff — `SPEC.md` is the source of truth.
This skill is the operating procedure.

## When it fits

```
✅ They use an LLM (Claude, ChatGPT, …)
✅ You need three or more things from them
✅ Questionnaires to this person have gone unanswered before
🚨 Skip it: one question / they do not use an LLM / a phone call is faster
```

## The order — do not start at step 2

```
🚨 ① PREFLIGHT   interview the REQUESTER   templates/PREFLIGHT.md   ← never skip
   ② write       REQUEST.md                templates/REQUEST.md
   ③ review      it never comes back       SPEC 4-6 / 4-7
   ④ hand over   the file, plus one line telling them what to type
   ⑤ read        BRIEF.md; only the unconfirmed items go into the next round
```

### 🚨 ① PREFLIGHT is where the round trips are won or lost

The two questions that matter most, because you can almost never answer them yourself:

- **What is already settled** — what must NOT be re-asked
- **What do they already have** — minutes, past messages, half-formed drafts

Skip those and you produce a file that asks someone to re-answer their own meeting, from
scratch. Everything else in the checklist is secondary.

🚨 **The checklist is not a script.** Sort each item into "I can answer this" / "does not
apply here" / "only they know", ask only the third group, and **say out loud what you skipped**.

### ② Writing

- Body in English, `lang:` for the conversation (SPEC 2)
- 🚨 **Rules, not attributes.** Ask what they want to happen — "〈this kind of person〉 gets X,
  then Y after N days" — and derive the data you need from that, in front of them. Ask about
  the form first and you collect answers nothing uses
- Show drafts from what they already said and ask for confirm / correct / complete. Blank
  questions are what make an interview feel abstract
- 🚨 **Never have them told what something is NOT** (SPEC 4-8). Corrections in the file are for
  their LLM; said aloud they make the person wonder why they are being contradicted
- Context goes in the file, thick. The conversation gets the question plus one or two lines,
  and one offer of depth on request (SPEC 4-2 / 4-3)

### ③ Review before handing over

**It never comes back.** It enters their context and stays in their conversation.

```
🚨 Look for: third parties' names and situations / your own notes about people /
   other projects / credentials, identifiers, internal hostnames and paths
✅ Have someone who is not you read it — another LLM counts
```

### ④ Handing over

Attach the file. Do not paste it into the message body — that just moves the copying to them.
Tell them the one line to type:

> このファイルは私宛の依頼です。全部読んでから、書いてあるとおりに1つずつ質問を始めてください。

### ⑤ Reading the brief

- 🚨 `unconfirmed` is a real answer. Do not fill it in yourself
- A note with no value is **unanswered**, not a soft yes
- Check the environment section: what they could not do, and what they did instead
