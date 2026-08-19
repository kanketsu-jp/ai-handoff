# AI Handoff — Specification

[日本語](SPEC.md)

## 0. Premise — the two LLMs share no memory

Today, this is how two people work together:

```
me → ask my LLM → write a message → send
                                  → they receive it → ask their LLM → write a reply → send → …
```

**There is an LLM at each end, and a human carrying text between them.** This is a transitional shape.

But the current shape has something **worth keeping**. Each LLM holds things that
**exist only in its own environment**.

```
🟢 Never crosses the boundary (stays in each environment)
   ・secrets (tokens, keys, customer data)
   ・company-specific knowledge, past threads, internal rules
   ・the Skills / MCP servers / tools that person installed
```

So this is not "connect the two via an API". **You hand over one file.**
**Only the file crosses.** The environments never mix.

What the current shape gets **wrong** is this:

```
🚨 No shared context … their LLM knows nothing about your system
🚨 More round trips … humans translate "what does this mean" at both ends
🚨 Copy-paste in the middle … a human re-pastes chat text into an LLM. That part is not AI-native
```

**This protocol solves those three things only.**
Put the context in the file, cut the round trips to one, remove the copy-paste.

---

## 1. Artifacts

The protocol has exactly two artifacts.

| File | Written by | Read by |
|---|---|---|
| `REQUEST.md` | the requester (with their LLM) | **their LLM** |
| `BRIEF.md` | their LLM | **the requester's LLM** |

**Both must be readable by a human, but neither is optimized for one.**
They are optimized for an LLM being able to act on them directly.

---

## 2. Language — the documents are English, the conversation is not

**Write the body of `REQUEST.md` and `BRIEF.md` in English.**

The reader is an LLM, not a person. Writing the same content in Japanese costs
roughly **1.5–2x the tokens**. That cost repeats on every round trip, so the body is fixed to English.

**The conversation is a separate matter.** If the person on the other end speaks Japanese,
the LLM must speak Japanese to them. Putting that instruction somewhere in the body
**gets skipped**, so it goes in **front matter**.

### 2-1. Front matter

The very first lines of `REQUEST.md`:

```yaml
---
protocol: ai-handoff/1
lang: ja
from: kazuma-horiike (KANKETSU Inc.)
people:
  - id: r-tanaka
    to: primary
    role: decision-maker
    tech: non-eng
    knows: html,css,llm-assisted-pages
    gap: next.js,headless-cms,deploy
    terms: explain-1line-on-first-use
    decides: content,copy,photos
    defers: pricing->manager
---
```

| Key | Meaning |
|---|---|
| `protocol` | Protocol and version. Fixed to `ai-handoff/1` |
| `lang` | 🚨 **The language to speak to the human in** (BCP 47: `ja` / `en` / `ko` …) |
| `from` | Who the request is from |
| `people` | 🚨 **Recipients.** Always a list, even for one person (§2-3) |

**`lang` is the conversation language, not the document language.**
The document is always English.

### 2-2. 🚨 `people` — who it is for, and how to talk to them

**Anything about a person goes here in a short machine-readable form, never as prose in the body.**

Two reasons:

```
① Prose like "this person is a non-engineer who..." will be read by the person it describes
   🚨 The file stays on their machine. It reads as an assessment, whatever you meant
② Prose is long. The same information fits in about a third of the tokens
```

| Key | What goes in it | Example |
|---|---|---|
| `id` | Short handle (used in `BRIEF.md` to attribute quotes) | `r-tanaka` |
| `to` | `primary` (ask this person) / `cc` (present, or an escalation target) | `primary` |
| `role` | Their role on this project | `decision-maker` |
| `tech` | Technical baseline | `non-eng` / `eng` / `designer` |
| `knows` | Areas needing no explanation (comma-separated) | `html,css` |
| `gap` | Areas outside their experience. **Explain these carefully** | `next.js,headless-cms` |
| `terms` | How to handle terminology | `explain-1line-on-first-use` |
| `decides` | What they can settle alone | `content,copy,photos` |
| `defers` | What they route elsewhere | `pricing->manager` |

**Only include fields that actually change the LLM's behaviour.**
Anything that does not change behaviour is just a character assessment. Leave it out.

### 2-3. 🚨 Never read `people` out loud

**It is input to the LLM, not material for the conversation.**

```
❌ "Kazuma asked me to explain technical terms in one line for you."
✅ Just add the one-line explanation.
```

Reading it back to the person **undoes the reason it was written short** — it becomes prose again.
Use it only to pick vocabulary and depth.

🚨 **Writing it short does not make it hidden.** The file is on their machine; they can open it.
**Writing something that is fine to read** is the actual rule; brevity is how you get there.

### 2-4. When there is more than one recipient

If `people` has two or more entries, **the LLM establishes which one its owner is first.**

```
・owner is primary … proceed
・owner is cc …… put items needing the primary's confirmation in a separate section of BRIEF.md
・owner is neither … 🚨 stop and ask (the file may have reached the wrong person)
```

In `BRIEF.md`, **attribute every quote with the speaker's `id`.**

### 2-5. What is English and what is not

| | Language |
|---|---|
| Body of `REQUEST.md` | **English** |
| Body, headings and JSON keys of `BRIEF.md` | **English** |
| 🚨 **Quotes of what the person said** | **Their own language, verbatim** (do not summarize or translate) |
| 🚨 Proper nouns (store names, people, URLs, product names) | **As written** (translating breaks matching) |
| Talking with the person | The language in `lang` |

If you translate a quote, the requester can no longer check whether the person actually said that.
**A quote is evidence. Do not process it.**

---

## 3. 🚨 The receiving LLM decides what its own environment can do

**Even within "Claude", capability differs by environment.**
Running in a terminal, in a desktop app, or on a phone changes whether you can write a file at all,
and whether you can connect to external tools.

So **`REQUEST.md` never prescribes a mechanism. It states the goal; the receiving LLM picks the means.**

### 3-1. Three capabilities to test

| Capability | When available | 🚨 Fallback when not |
|---|---|---|
| **Writing a file** | Save `BRIEF.md` to disk | Offer it as a download → if that fails too, **print the whole thing in one code block in chat** |
| **A batch-answer UI** | An HTML form or a selection UI | **Ask one question at a time**, or emit a JSON skeleton to fill in |
| **External tools (MCP)** | Connect directly to an admin panel | **Do not connect. Finish with the interview alone** (the requester does the data entry) |

🚨 **Decide by actually trying.** Claiming a capability and then failing is the worst outcome.

### 3-2. Rough guide by environment

| Environment | Files | Batch answers | Installing MCP |
|---|---|---|---|
| **Claude Code** (terminal / IDE) | Writes to disk directly | Write an HTML file and have them open it | Edit `~/.claude.json` |
| **Claude desktop / web** | Offer a download | Use an Artifact | Settings → Connectors → custom connector |
| **Claude mobile** | 🚨 Print in chat | 🚨 One at a time | 🚨 Not possible |
| **Another vendor's LLM** | Depends | Depends | 🚨 Usually not possible |

🚨 **This table is a hint, not evidence.** These products change.
**Do not trust the table. Try it.**

### 3-3. Whatever you could not do, write it in `BRIEF.md`

**Do not silently substitute another mechanism.**
The requester cannot tell why the result came back in an unexpected shape, and asks again.
**Cutting round trips is the point of this protocol**, so a silent fallback violates it.

Put the environment, what failed, why, and what you did instead at the top of `BRIEF.md`:

```markdown
## 0. Environment

- Runtime: Claude mobile app
- 🚨 Could not write a file (no filesystem access) -> this brief is pasted into chat as a single block
- 🚨 Could not connect to the admin panel (MCP unavailable on mobile) -> no data was entered; see §3
```

---

## 4. Rules for `REQUEST.md`

### 4-1. Declare in the first line that the reader is an LLM

Right after the front matter, address **their LLM**, not the person.

```markdown
> **This file is a request addressed to you (the LLM), not to your owner.**
> Interview your owner, then produce `BRIEF.md`.
> 🚨 Read the whole file first. Then start a conversation, one topic at a time.
```

### 4-2. Do not let it ask everything at once

**This is the core of the protocol.** Forwarding a questionnaire through an LLM improves nothing.

```
✅ One topic at a time, in conversation
✅ Batch only when the person asks to answer in bulk
🚨 Do not prescribe the mechanism (§3 — environments differ)
```

### 4-3. Be generous with context

Their LLM knows nothing about your system. **Questions written without that context
produce an off-target interview.**

```
✅ What you are building (URL, page structure, dynamic routes)
✅ What is currently placeholder and what is real
✅ How easy each change is (what is trivial, what touches the design)
✅ Terms (explain every abbreviation in one line)
```

### 4-4. 🚨 Do not characterize the person

Write **what needs explaining**, never **what kind of person they are**.

```
❌ "Your owner is a non-engineer who understands some HTML."
✅ "Your owner decides this project but does not implement it.
    Explain any technical term in one line the first time it appears."
```

🚨 **`REQUEST.md` stays on their machine.**
A characterization will be read exactly as written, whatever you meant by it.

### 4-5. 🚨 `REQUEST.md` never comes back

Once handed over, you cannot retract it. **No third-party personal data, no internal decisions,
no other clients.** Have someone other than yourself read it first — another LLM counts.

### 4-6. Keep secrets out of the body

Deliver tokens and keys **through a separate channel**. In `REQUEST.md`, say only
"you will receive this separately". Have their LLM **walk the person through the setup**
rather than handling the value itself.

🚨 **Match the token's privileges to the recipient's.**
Hand over an admin token and every "only touch this part" in `REQUEST.md` becomes decorative.

---

## 5. Rules for `BRIEF.md`

### 5-1. 🚨 Never guess. Write `unconfirmed`

**The single most important rule in this protocol.**

```markdown
❌ "They probably want updates about twice a week."
✅ "update_frequency: unconfirmed (asked; owner has not decided)"
```

Returning with blanks is fine. **A filled-in guess is worse than a blank.**

### 5-2. Return structured data as JSON code blocks

Prose forces the requester to re-structure it, which is the round trip you were removing.

````markdown
```json
{
  "stores": [
    { "name": "◯◯店", "brand": "be", "address": "unconfirmed", "open_date": "2026-04" }
  ]
}
```
````

**`REQUEST.md` supplies the schema.** The receiving LLM does not invent one.
🚨 **Keys in English; proper nouns in their original form** (§2-5).

### 5-3. Record who decided

```markdown
| Item | Answer | Decided by | Status |
|---|---|---|---|
| … | … | owner | settled |
| … | — | **needs confirmation (their manager)** | pending |
```

The requester no longer has to ask "is this actually settled?" every time.

### 5-4. Partial returns are fine

```markdown
## Progress
- Answered: 3 / 6 topics
- 🚨 Not started: sample articles, design feedback
- This brief is **partial**. The rest will follow in a later brief.
```

### 5-5. Keep the original words

Summarizing removes the requester's ability to judge. **Quote what they said with `>`.**
🚨 **Do not translate quotes** (§2-5).

### 5-6. 🚨 State the environment and what failed

Per §3-3, open with `## 0. Environment`. **Write it even when everything worked** —
otherwise "did not write it" and "everything worked" look identical.

---

## 6. What you do not have to follow

- **Exact formatting.** If the items are there and nothing is guessed, it works
- **Finishing in one pass.** Round trips are allowed
- **Filling everything in.** Partial returns are fine (§5-4)

---

## 7. Anti-patterns

- ❌ Listing 20 questions in `REQUEST.md` and saying "answer these" (that is just a questionnaire, forwarded)
- ❌ Telling their LLM to ask everything at once (the opposite of §4-2)
- ❌ 🚨 **Prescribing a mechanism** ("build an HTML form", "write a file") — some environments cannot (§3)
- ❌ 🚨 **Silently substituting a fallback** (§3-3 — it creates the round trip you were removing)
- ❌ 🚨 **Writing the body in a non-English language** (§2 — the reader is an LLM; 1.5–2x the tokens)
- ❌ 🚨 **Translating quotes or proper nouns** (§2-2 — it destroys evidence and breaks matching)
- ❌ 🚨 **Describing the person's technical level or character** (§4-4 — the file stays with them)
- ❌ Guessing in `BRIEF.md` (§5-1)
- ❌ Returning structured data as prose (§5-2)
- ❌ Putting secrets or third parties in `REQUEST.md` (§4-5 / §4-6)
- ❌ Handing over an over-privileged token (§4-6)
- ❌ Omitting who decided (§5-3 — it forces a confirmation round trip)
