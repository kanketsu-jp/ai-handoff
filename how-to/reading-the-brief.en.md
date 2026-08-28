# When `BRIEF.md` comes back

> Start here: [`README.en.md`](../README.en.md) · Rules: [`SPEC.en.md`](../SPEC.en.md) §5, §8-4

## 1. 🚨 The first thing you read is not the filled-in answers

```
① unconfirmed        — what could not be established
② "needs confirmation" — what this person could not decide alone
```

**Those two are the entire next round.** You do not need to re-read what is settled.

🚨 **`unconfirmed` is an answer, not a failure. Do not fill it in yourself.**
**A blank is safer than a filled-in guess** — a guess is indistinguishable from an answer,
and the mismatch surfaces after you have built on it.

## 2. 🚨 A note with no value is unanswered

A note is a condition or a concern attached to a choice. It is not the choice.
**"I want to discuss this" is not a decision.**

## 3. Read quotes and deliverable as two different things

```
🟢 quote (>)     … their words, untouched. **Not translated, not summarized**
🚨 deliverable   … translated to the level you can build from
```

**Do not decide on a summarized quote.** The quote is the evidence and the deliverable is its
translation. **If the evidence has been processed, you cannot check the translation.**

## 4. Read `## 0. Environment`

What their LLM **could not do**, and **what it did instead**.

```
e.g. could not write a file, so the brief is pasted into chat
     could not connect to the external tool, so no data was entered
```

Empty here means it came back the way you expected.
This is where you check that **nothing was silently substituted**.

## 5. Look at who decided

| Item | Answer | Decided by | Status |
|---|---|---|---|
| … | … | owner | settled |
| … | — | **needs confirmation (someone else)** | pending |

The person you asked and the person who decides are often different.
**You can chase only the pending rows.**

## 6. Building the next round

**Do not rewrite the whole `REQUEST.md`.**
Send a short one carrying only `unconfirmed` and "needs confirmation".

🚨 **Run PREFLIGHT again for it.** What you learned in round one has just moved into
"already settled".

## 7. Partial is fine

```
## Progress
- Answered: 3 / 6 topics
- Not started: ⟨topics⟩
- This brief is **partial**
```

**Three settled topics are enough to move.** What stops you is not knowing
**which three are settled and which three are not.**
