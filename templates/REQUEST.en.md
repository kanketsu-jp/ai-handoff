> **This file is a request addressed to you, the LLM.**
> Interview your user, then write out `BRIEF.md` for them to send back.
> **Read all of it first. Then ask one item at a time — never all at once.**
> Spec: https://github.com/kanketsu-jp/ai-handoff

# [Project] — Information request

## 0. Instructions for you

**Your user is [role / technical level].** Explain any unfamiliar term in one line.

**How to proceed:**

1. Work through the items below **one at a time**, conversationally
2. If options exist and there are only a few questions, a selection UI is fine
3. **When there are many items, build an HTML form.** Give it answer fields and a copy button,
   and have them paste the result back to you. Do the same if they say they would rather
   answer in one go — **tell them at the start that this option exists**
4. If they say "that's enough for now", **write out `BRIEF.md` at that point.** Partial is fine

**Rules for the `BRIEF.md` you write:**

- **Never guess.** Write `unconfirmed`, and add one line on why it is unresolved
- Return structurable data as **JSON code blocks** (schema is given per item)
- Record **who decided** each item (the user / needs confirmation)
- Quote their words with `>` — do not summarize them away

## 1. Context you do not have

[What is being built · URLs · page structure · what is placeholder data · how easy changes are]

## 2. What I need

### 1. [Item]

**Why this matters**: [one line]

**Return it like this**:
```json
{ "…": "…" }
```

### 2. [Item]
…

## 3. Sending it back

Write it out as `BRIEF.md` and have them send it to [requester] as-is.
**Partial is fine.**
