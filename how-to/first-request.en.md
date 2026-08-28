# Your first request — from PREFLIGHT to handing it over

> Start here: [`README.en.md`](../README.en.md) · Rules: [`SPEC.en.md`](../SPEC.en.md)

**Worked example** (deliberately not software):
someone redesigning an internal approval process needs to hear from **the people who file
those requests every day**.

```
What the designer needs … under what conditions it routes to whom (enough to write branches)
What the field has     … the lived practice: "when it's like this, we ask the section head"
🚨 The field cannot answer "what are the conditions". **They can answer in the language of the work**
```

## 1. 🚨 Do not start by writing `REQUEST.md`

**First you — the requester — get interviewed by your own LLM.**
The checklist is [`templates/PREFLIGHT.md`](../templates/PREFLIGHT.md).

🚨 **It is not read out loud.** Sort first:

| Bucket | What to do |
|---|---|
| The LLM can already answer it | Do not ask. **State the answer; let the requester correct it** |
| Does not apply to this request | 🚨 **Do not ask. Just say it was skipped** |
| Only the requester knows | Ask. One at a time |

Example: no deadline on this piece of work → do not ask "when do you need it by".
One line — "I will assume there is no deadline" — is enough.

### The two that carry the weight

```
① What is already settled       → the range that must NOT be re-asked
② What does the recipient have  → minutes, past messages, half-written drafts
```

🚨 **Skip those two and you produce a file that asks someone to re-answer their own meeting.**
"The questions came out abstract" is the symptom; this is the cause.

## 2. Writing it

Template: [`templates/REQUEST.md`](../templates/REQUEST.md). Non-negotiables:

- **Body in English, conversation in `lang:`** (the reader is an LLM; other languages cost 1.5–2× the tokens)
- **Person information goes in `people`, short. Not prose in the body** (the file stays with them)
- 🚨 **Say how deep each topic has to go.** Without it they stop where answering is comfortable
- 🚨 **Show what they already said as a draft and ask them to confirm / correct / complete.**
  Ask from blank and you get things that are interesting to know and never used
- 🚨 **You supply the JSON schema.** Their LLM does not invent one

### 🚨 Every constraint needs one more line

State a cost, a limit, a signal you cannot capture, and **they will quietly ask for less** —
and you never learn what they were going to ask for.

```
❌ "We can't capture X, so decide using Y"
✅ "Y is what works right now. But if you want to split on X, say so —
    whether we can build it is our problem"
```

### 🚨 Never have them told what something is NOT

Corrections in the file address **their LLM**. Said to the person, they make them wonder
**why they are being contradicted** — about your wording history, not their work.

```
❌ "This is not extra homework. It is the task assigned to you at…"
✅ "This is the task assigned to you at the meeting — these two lines"
```

## 3. Review before it leaves

🚨 **`REQUEST.md` never comes back.** It enters their context and stays in their conversation.
Checklist: [`SPEC.en.md`](../SPEC.en.md) §8. Above all:

```
third parties' names and situations / your own deliberation, intent, figures /
other projects / secrets (keys, tokens, internal names)
✅ have someone other than you read it once (another LLM counts)
```

## 4. Handing it over

**Attach the file. Do not paste it into the message body** — that moves the copying to them.
One line goes with it:

```
This file is a request addressed to you. Read all of it first, then start asking, one topic at a time.
```

"Read all of it first" is what stops it summarizing and stopping.

## 5. Common failures

| Symptom | Cause | Fix |
|---|---|---|
| They said the questions were abstract | You did not use what they already said | §1 ②. Show drafts to confirm |
| Only shallow answers came back | No depth stated | §2. Say what becomes undecidable without it |
| They asked for less than they wanted | You stated a constraint and stopped | §2. Add "tell us what you want anyway" |
| "Why am I being contradicted?" | A correction was spoken to the person | §2. Use the positive form |
| Nothing comes back until it is complete | You did not say partial is fine | Ask for the load-bearing topics first |
