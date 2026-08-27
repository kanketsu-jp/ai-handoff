# Preflight — what to ask the requester before writing `REQUEST.md`

> 🚨 **This file never crosses the boundary.** It is not one of the two deliverables.
> You (the requester's LLM) use it to interview the requester, fold the answers into
> `REQUEST.md`, and throw it away.
> Spec: https://github.com/kanketsu-jp/ai-handoff (SPEC 4-0)

**Speak the requester's own language here.** Ask one at a time, the same way the recipient's
LLM will. If you skip this, the assumptions that live only in the requester's head stay there,
and you find out which ones were missing only after the answers come back.

---

## 1. Who receives this, and what can they settle alone?

- Name, and their role on this piece of work
- 🚨 What they can decide by themselves, and what they will have to pass to someone else
- Is anyone else in the room (cc), and what do they need to see?

→ becomes the `people` front matter. Getting this wrong means asking someone to decide
something they cannot decide.

## 2. 🚨 What is already settled?

**List what must NOT be re-asked.**

- Decisions already made, with where they were made
- Anything the recipient personally already agreed to

→ becomes a "settled — only flag it if they contradict it" block.
🚨 Skip this and you produce a file that asks someone to re-answer their own meeting.

## 3. 🚨 What does the recipient already have?

- Minutes, past messages, decks, spreadsheets, earlier drafts
- 🚨 Are there **draft answers** in there? Half-formed ones count

→ becomes drafts to confirm rather than blanks to fill.
Blank questions are what make an interview feel abstract. Drafts make it concrete.

## 4. When is it needed, and is partial acceptable?

- The real deadline, and what happens at it
- Which topics are load-bearing, so a short answer can be prioritised
- Is a partial answer useful, or is it all-or-nothing?

→ becomes the closing section. Without it, the recipient aims for a completeness that
arrives too late.

## 5. 🚨 Is this new work for them, or work they already owe?

- If it is already theirs: quote the exact line that assigned it
- If it is new: say what it replaces or adds to

→ decides the opening sentence. The same request reads completely differently as
"here is something extra" versus "here is the thing you already have".

## 6. What happens to the answer?

- Who acts on it, and what gets built or decided
- Anything the answer will be recorded in

→ lets the recipient judge how precise to be.

## 7. How does the file get there?

- Who sends it, through what channel, as an attachment or pasted
- 🚨 Does the requester want to approve the text before it goes?

→ prevents a finished file stalling on delivery.

## 8. 🚨 Is anything in here unrecoverable once sent?

- Third parties' words, internal deliberations, other projects
- Credentials, identifiers, internal hostnames or paths
- 🚨 `REQUEST.md` never comes back (SPEC 4-6). Ask before, not after

---

## Before you write

Two answers do most of the work: **§2 (already settled)** and **§3 (what they already have)**.
🚨 If you cannot answer those two, do not start writing. You will produce a file that asks a
person to invent, from nothing, things they have already said.
