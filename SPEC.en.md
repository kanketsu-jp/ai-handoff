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

## 0.5 🚨 The two ends need different words (this is the point)

**The person asking and the person answering usually do not share a vocabulary.**

```
What the requester needs … something they can build from
                           (how data is shaped, what branches key on, what to compute next)
What the answerer has  … the business judgement. Not the terminology — nor should they need it
```

Traditionally a **human translated twice** to close that gap: simplify the need into a
questionnaire, then translate the answers back up. **Both translations lose something.**

**This protocol hands the translating to their LLM.**

| | Conversation | Deliverable |
|---|---|---|
| The person answering | **entirely in their own words** | — |
| The person asking | — | **at the level they can build from** |

🚨 **One conversation, two registers.** §4-9 and §5-8 are the rules that make it happen.

## 1. Artifacts

The protocol has exactly two artifacts.

| File | Written by | Read by |
|---|---|---|
| `REQUEST.md` | the requester (with their LLM) | **their LLM** |
| `BRIEF.md` | their LLM | **the requester's LLM** |

🚨 That counts only what **crosses the boundary**. One more lives on your side --
[`templates/PREFLIGHT.md`](./templates/PREFLIGHT.md) (**what to ask the requester before you
write**, §4-0) -- and it is never handed over.

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

### 4-0. 🚨 Interview the requester before you write

**Before writing a line of `REQUEST.md`, question the requester.**
Skip it and you produce a file with **the requester's own assumptions missing from it** --
and you discover which ones only after the answers come back. This is the single largest
source of extra round trips.

The questionnaire is [`templates/PREFLIGHT.md`](./templates/PREFLIGHT.md).
**It never crosses the boundary** (§1 counts only what does). Use it locally, fold the answers
into `REQUEST.md`, discard it.

Eight questions.

| # | What to ask | What breaks without it |
|---|---|---|
| 1 | Who receives it, and **what they can settle alone vs. pass on** | You ask someone to decide what they cannot decide |
| 2 | 🚨 **What is already settled** (must not be re-asked) | You ask them to re-answer their own meeting |
| 3 | 🚨 **What they already have** (minutes, past messages, drafts) | You make them invent from nothing; questions come out abstract |
| 4 | **When it is needed**, and whether partial is acceptable | They aim for a completeness that arrives too late |
| 5 | 🚨 Is this **new work, or work they already owe** | It reads as an extra chore piled on top |
| 6 | **What happens to the answer** | They cannot judge how precise to be |
| 7 | **Who sends it and how** | A finished file stalls on delivery |
| 8 | Anything unrecoverable once sent (§4-6 / §4-7) | It goes somewhere you cannot take it back from |

🚨 **2 and 3 carry most of the value.** A `REQUEST.md` written without them asks a person to
work out, from scratch, things they have already said.

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

#### 🚨 And keep each question short

**Do not deliver three paragraphs of background and then ask.** They came to answer, not to
read a briefing.

```
❌ context, context, context → finally, a question
✅ one question + the one or two lines that bear on it → "say the word if you want detail"
```

**Context goes in the file, thick; it does not go into the conversation** (→ §4-3).
The LLM holds all of it and surfaces only what this question needs.

Have it offer depth **once, at the start**:

```markdown
> I can explain the reasoning behind any of this --
> **just say "tell me more" and I will go deeper.**
```

Without that line, a short question reads as a shallow topic, and people answer by guessing.
With it, only the people who need depth spend time on it.

### 4-3. Be generous with context — 🚨 in the FILE, not in the conversation

Their LLM knows nothing about your system. **Questions written without that context
produce an off-target interview.**

```
file … thick. An LLM reads it, once
🚨 conversation … thin. The LLM holds the context and spends one or two lines of it (§4-2)
```

**Confuse the two and the person is made to read your preamble.**

```
✅ What you are building (URL, page structure, dynamic routes)
✅ What is currently placeholder and what is real
✅ How easy each change is (what is trivial, what touches the design)
✅ Terms (explain every abbreviation in one line)
```

### 4-4. 🚨 Do not only ask for answers. Accept material

**Asking "is X correct?" makes the person recall, look up, and summarize.
Most of that work disappears if they can just hand something over.**

So every question carries this:

```markdown
🚨 **Also accept material instead of an answer.**
Say: "if you have a file, a link, a screenshot or a document about this,
just hand it over -- I will read it and reconcile it against what we have."
Read whatever they give you, extract the answer, and report the difference.
```

| ❌ Answers only | ✅ Material too |
|---|---|
| They recall and write it (slow, error-prone) | **They hand over what already exists** (seconds) |
| "I think it was 105 days?" | The employment rules PDF -> **a certain figure** |
| They stall deciding whether to answer or send | **Either is fine, so they do not stall** |

- **Never ask them to summarize it.** "Pull out the relevant part" is just more work for them.
  **They hand it over as-is; reading it is the LLM's job**
- 🚨 **Reconcile it against what we already have and report the difference** (§5-7).
  Receiving material and stopping there makes the requester redo the same reading
- Screenshots, photos, pasted chat logs, outdated documents -- take all of it.
  **"It is too messy to send" is the worst outcome**, so say that explicitly

### 4-5. 🚨 Do not characterize the person

Write **what needs explaining**, never **what kind of person they are**.

```
❌ "Your owner is a non-engineer who understands some HTML."
✅ "Your owner decides this project but does not implement it.
    Explain any technical term in one line the first time it appears."
```

🚨 **`REQUEST.md` stays on their machine.**
A characterization will be read exactly as written, whatever you meant by it.

### 4-6. 🚨 `REQUEST.md` never comes back

Once handed over, you cannot retract it. **No third-party personal data, no internal decisions,
no other clients.** Have someone other than yourself read it first — another LLM counts.

### 4-7. Keep secrets out of the body

Deliver tokens and keys **through a separate channel**. In `REQUEST.md`, say only
"you will receive this separately". Have their LLM **walk the person through the setup**
rather than handling the value itself.

🚨 **Match the token's privileges to the recipient's.**
Hand over an admin token and every "only touch this part" in `REQUEST.md` becomes decorative.

---

### 4-8. 🚨 Never have them told what this is NOT

**Corrections inside `REQUEST.md` are addressed to their LLM. They are not addressed to the
person.**

Write "this is not A, it is B" and the LLM will say exactly that, out loud. The person then
wonders **why something is being denied** -- and the denial is a fact about your wording
history, not about their work.

```
❌ "This is not a campaign. What we are building is…"
❌ "This is not extra homework. It is the task assigned to you at…"
✅ "What we are building is ⟨X⟩, and it is permanent"
✅ "This is the task assigned to you at the meeting -- these two lines"
```

**Let the LLM hold the understanding and never voice it.** Write it like this:

```markdown
🚨 **Never tell your owner what this is NOT. State what it IS.**
For you, silently: ⟨the correction⟩. 🚨 Hold that. Do not recite it.
```

Same family as §4-5 (do not characterize the person): **both are information about YOU, not
about them**, and **the file stays in their hands** (§4-6).

### 4-9. 🚨 State how deep it has to go, not just what to ask

**Without it, their LLM stops at the depth that is comfortable to answer** — which is not
necessarily the depth you can build from. Per topic, say **what has to become decidable**.

```markdown
🚨 **How deep this has to go**: ⟨what must be decidable from the answer⟩
Keep asking, in their words, until you have that. Do not stop at the first answer that
sounds complete.
```

Three rules for their LLM:

- 🚨 **Do not assume shallowness.** `tech` and `gap` (§2-2) are **a starting point, not a
  ceiling.** If depth surfaces in some area, follow it
- 🚨 **Do not return a shallow answer with "they did not know."** Ask again — **in their
  words, inside the flow of the conversation.** Do not switch into jargon
- 🚨 **Dig only while something is still undecidable.** Digging past a settled answer is an
  interrogation, not an interview

🚨 **A constraint is not a reason for them to ask for less.** When you state one — cost, a
signal you cannot capture, a limit — **ask for what they actually want anyway**, and say that
working out how is your side's problem. Otherwise the constraint quietly truncates the answer,
and you never learn what they were going to ask for.

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

### 5-7. 🚨 Record what material you received, and the differences you found

When §4-4 gets you material, separate **what arrived**, **what you read from it**, and
**how it differs from what we have**.

```markdown
## Sources received

| What | From | Read? |
|---|---|---|
| 就業規則.pdf | owner (LINE) | yes |
| Indeed job page | URL | yes |
| Screenshot of the pay table | owner | 🚨 partially (blurred) |

## Differences found

| Item | On the site now | In the material | Verdict |
|---|---|---|---|
| 年間休日 | 105 days | **110 days** | 🚨 site is wrong |
| 平均年齢 | 25 | (not mentioned) | unconfirmed |
```

- 🚨 **Never stop at "they sent me a document."** That makes the requester redo the reading
- 🚨 **Say when you could not read something** -- do not mix it in with what you did read
- If the material contradicts what the person said, **do not pick a winner.**
  Write both and ask them

---

### 5-8. 🚨 Write `BRIEF.md` at the requester's level

**"The conversation was non-technical" and "the deliverable should be non-technical" are two
different claims.** The reader is the requester's LLM, and the next stop is the work itself.
**Write it at the level they can build from.**

```
conversation  "I want to send different things depending on whether they know the product"
🚨 brief       express it as conditions, branches, and how the data is held (schema: §5-2)
🟢 quote       "I want to send different things depending on…" ← their words, untouched (§5-5 / §2-5)
```

🚨 **Quotes are the one thing you never translate.** The quote is the evidence; the brief is
the result of translating it. **Carry both.**

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
- ❌ 🚨 **Describing the person's technical level or character** (§4-5 — the file stays with them)
- ❌ Guessing in `BRIEF.md` (§5-1)
- ❌ Returning structured data as prose (§5-2)
- ❌ Putting secrets or third parties in `REQUEST.md` (§4-6 / §4-7)
- ❌ Handing over an over-privileged token (§4-7)
- ❌ Omitting who decided (§5-3 — it forces a confirmation round trip)
- ❌ 🚨 **Asking only for answers, never for material** (§4-4 — you are making them recall)
- ❌ 🚨 **Asking them to summarize the material** (reading is the LLM's job)
- ❌ 🚨 **Stopping at "they sent a document"** (§5-7 — without the diff, nothing was saved)
- ❌ 🚨 **Writing `REQUEST.md` without interviewing the requester first** (§4-0 — especially
  "what is already settled" and "what they already have"; skip those and the file asks a person
  to re-answer their own meeting)
- ❌ 🚨 **Front-loading background before the question** (§4-2 — context in the file, one or two
  lines in the conversation)
- ❌ 🚨 **Never offering depth** (§4-2 — unoffered, a short question reads as a shallow one and
  people guess)
- ❌ 🚨 **Having the person told "this is not X"** (§4-8 — corrections are for the LLM; said
  aloud they make the person wonder why they are being contradicted)
- ❌ 🚨 **Not saying how deep it has to go** (§4-9 — they stop where answering is comfortable)
- ❌ 🚨 **Treating `tech: non-eng` as a ceiling** (§4-9 — a starting point; follow depth when it appears)
- ❌ 🚨 **Returning a shallow answer with "they did not know"** (§4-9 — ask once more, in their words)
- ❌ 🚨 **Letting a stated constraint shrink what they ask for** (§4-9 — ask for what they want;
  how to build it is your problem)
- ❌ 🚨 **Writing `BRIEF.md` non-technically because the conversation was** (§5-8 —
  the reader is the requester's LLM. **Quotes alone stay in their words**)

## 8. Review checklist

**Go through this before handing over.** §7 is what not to do; this is what you did.

### 8-1. Before writing (§4-0)

- [ ] Did you interview the requester — above all, **what is already settled** and **what they already have**?
- [ ] Did you skip the questions that do not apply, and **say that you skipped them**?

### 8-2. Inside `REQUEST.md`

- [ ] Front matter present (`protocol` / `lang` / `from` / `people`)
- [ ] The first line addresses the LLM (§4-1)
- [ ] "Do not ask everything at once" and "partial is fine" are both stated (§4-2 / §5-4)
- [ ] **You supplied the JSON schema** — their LLM does not invent one (§5-2)
- [ ] Environment self-check is there, and **no mechanism is prescribed** (§3)
- [ ] The offer to accept material appears **per topic**, not once (§4-4)
- [ ] 🚨 **Each topic says how deep the answer has to go** (§4-9)
- [ ] 🚨 **Every constraint is paired with "tell us what you want anyway"** (§4-9)
- [ ] Depth is offered once, at the start (§4-2)
- [ ] 🚨 Nothing is phrased so the person gets told what something is NOT (§4-8)
- [ ] Person information lives in `people`, not the body, and **only what changes behaviour** (§2-2 / §4-5)
- [ ] Quotes and proper nouns are marked not-to-be-translated (§2-5)

### 8-3. 🚨 Before it leaves (it never comes back — §4-6)

- [ ] No third parties' names or situations
- [ ] None of your own deliberation, intent, or figures
- [ ] Nothing from another project
- [ ] No secrets — tokens, keys, internal hostnames or paths (§4-7)
- [ ] **Someone other than you read it once** (another LLM counts)

### 8-4. When `BRIEF.md` arrives

- [ ] **Is it written at the level you can build from?** (§5-8 — quotes excepted)
- [ ] Pull out `unconfirmed` and "needs confirmation" first — **that is the entire next round**
- [ ] Use the JSON as-is; do not re-derive structure from it
- [ ] Check for guesses. If you find one, name it explicitly in the next request
- [ ] 🚨 **Check whether quotes were summarized.** A summarized quote is not evidence — do not decide on it
- [ ] Read `## 0. Environment`: what failed, and what they did instead
