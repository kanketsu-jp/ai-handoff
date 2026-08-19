# AI Handoff — Specification

[日本語](SPEC.md) ／ **English**

## 0. Premise — the two LLMs share no memory

Today, a request between two people looks like this.

```
me → ask my LLM → write a message → send
                              → they receive → ask their LLM → write a message → send → …
```

**There is an LLM at each end, and a human carrying text in the middle.** This is a transitional shape.

It has one property worth keeping. Each LLM holds **things that exist only in its own environment**.

```
Never crosses the boundary
   · secrets (tokens, keys, customer data)
   · company knowledge, past threads, internal rules
   · the Skills / MCP servers / tools that person installed
```

That is why this is **one file**, not a direct API link. **Only the file crosses.** Environments never mix.

And here is the part worth removing.

```
Context is not shared … their LLM knows nothing about your system
Round trips multiply … humans translate "what did they mean" at both ends
Copy-paste in between … a human moves text from a chat app into an LLM by hand
```

**This protocol solves those three and nothing else.** It puts context in the file explicitly,
cuts the round trips to one, and removes the copying.

---

## 1. Artifacts

There are only two.

| File | Written by | Read by |
|---|---|---|
| `REQUEST.md` | the requester | **their LLM** |
| `BRIEF.md` | their LLM | **the requester's LLM** |

**Both must be readable by humans, but neither is optimized for humans.**
They are optimized so an LLM can act on them directly.

---

## 2. Rules for `REQUEST.md`

### 2-1. Declare in the first line that the reader is an LLM

```markdown
> This file is a request addressed to you, the LLM.
> Interview your user, then write out `BRIEF.md`.
> Read all of it first, then start asking one item at a time.
```

### 2-2. Do not let it ask everything at once

**This is the substance of the protocol.** Handing a questionnaire through an LLM improves nothing.

```
Default: one item at a time, conversationally
Batch only when the user asks to
State in REQUEST.md when batching is allowed
```

**There are three ways to ask, and the LLM chooses.**

| Method | When |
|---|---|
| Plain conversation | Default. One question at a time |
| A selection UI (AskUserQuestion in Claude) | Options exist, and there are only a few questions |
| **An HTML form** | Many items. Build answer fields, have them copy → paste back to the LLM |

The LLM generates the HTML on the spot. Include a copy button, then interpret what is pasted back.

### 2-3. Be generous with context

Their LLM knows nothing about your system. **Questions written without it will miss.**

```
What you are building (URLs, page structure, dynamic routes)
What is currently placeholder data and what is real
How easy each change is (what is trivial, what touches the design)
Vocabulary — explain any term they may not know, in one line
```

### 2-4. `REQUEST.md` never comes back

Once handed over it cannot be recalled. **Do not write third-party personal data, internal
decisions, or other clients into it.** Have someone else — another LLM or a person — read it first.

### 2-5. Keep secrets out of the body

Deliver tokens and keys through a separate channel. In `REQUEST.md`, write only *"you will receive
this separately"* — and have their LLM walk them through the setup. **The value itself never
reaches the LLM.**

---

## 3. Rules for `BRIEF.md`

### 3-1. Never guess. Write `unconfirmed`

**This is the most important rule in the protocol.**

```markdown
Bad:  "They are probably assuming updates about twice a week."
Good: "Update frequency: unconfirmed (asked; they have not decided)."
```

Returning with blanks is fine. **A filled-in guess is worse than a blank.**

### 3-2. Return structured data as JSON code blocks

Prose forces the requester to re-read and re-structure it. That does not reduce round trips.

````markdown
```json
{ "items": [ { "name": "", "category": "A | B | C", "note": "unconfirmed" } ] }
```
````

**The schema comes from `REQUEST.md`.** Their LLM does not invent it.

### 3-3. Record who decided

```markdown
| Item | Answer | Decided by | Status |
|---|---|---|---|
| … | … | the user | settled |
| … | — | **needs confirmation (their manager)** | pending |
```

### 3-4. Partial returns are fine

```markdown
## Progress
- Answered: 3 / 6
- Not started: […]
- This BRIEF is **partial**
```

### 3-5. Keep the original words

Summarizing removes the requester's ability to judge. **Quote what they said with `>`.**

---

## 4. What you do not have to follow

- **Exact formatting.** If the items are there and nothing is guessed, it works
- **Finishing in one pass.** Round trips are allowed
- **Filling everything in.** See §3-4

---

## 5. Anti-patterns

- Listing 20 questions in `REQUEST.md` and saying "answer these" (that is a questionnaire with extra steps)
- Telling their LLM to batch the questions (the opposite of §2-2)
- Guessing in `BRIEF.md` (§3-1)
- Returning structurable data as prose (§3-2)
- Putting secrets or third-party information in `REQUEST.md` (§2-4 / §2-5)
- Omitting who decided (§3-3 — the requester ends up asking again)
