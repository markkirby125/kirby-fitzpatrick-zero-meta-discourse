# Zero Meta Discourse — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [4 Writing Hacks that Readers Love](https://www.youtube.com/watch?v=_X0UERlwz7Y)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Zero Meta-Discourse Rule — Payload Over Scaffolding

### 1.1 The mechanism in one sentence

Every sentence in a technical document carries one of two referents:

* **Payload** — a claim about the artifact: the code, the system, the diff, the test result, the constraint, the risk. It can be checked against something.
* **Scaffolding (meta-discourse)** — a claim about the *document* or the *writer's process*: what this section will discuss, what the code above demonstrates, what we are now moving on to. Fitzpatrick's domain examples are `Chapter 2 discusses…`, `The preceding section showed…`, `Now we will analyze…`, `The final topic to be discussed is…`; the software equivalents are `In this response I will refactor…`, `The above function works by…`, `Now let's examine the next file…`.

> **A sentence that describes the response instead of the system is deleted, not reworded. The referent of every sentence must be the artifact — never the message, the author, or the reader's understanding.**

The rule is not "be concise". It is narrower and mechanical: **audit the grammatical referent of each sentence, and remove the ones whose referent is the discourse itself**. Compression is a side effect, not the objective.

### 1.2 Why signposting was once rational — and why it is now a defect

Meta-discourse is a relic of oral rhetoric. In speech, navigation *must* be spoken, because the audience cannot see the structure, cannot skim, and cannot rewind. Written technical artifact review inverts every one of those conditions — and the AI-assistant channel inverts them again, because the artifact (diff, file tree, tool output) is delivered *alongside* the prose.

```text
ORAL CHANNEL  (signposting was load-bearing)     WRITTEN / DIFF CHANNEL  (signposting is surplus)
──────────────────────────────────────────       ──────────────────────────────────────────────
• one pass, no rewind                            • random access, re-read at will
• no visible structure while listening           • headings, hunk headers, file paths are visible
• listener cannot skim                           • reader skims first, reads second, exits early
• navigation must be spoken: "next I will…"      • navigation lives in the artifact, not the prose
• audience is trapped until the end              • attention leaves the moment value stops
• no artifact in front of the listener           • the artifact is on screen the whole time
```

### 1.3 Before / after on a delivered response

```text
❌ BEFORE — THE RESPONSE NARRATES ITSELF

  Assistant reply · 10 sentences · 3 facts · 7 meta-statements
  ────────────────────────────────────────────────────────────────────────────
  s1  "Here's what I changed and why."                         ✗ META   forward announcement
  s2  "In this section I'll walk through the retry fix."        ✗ META   spoken roadmap
  s3  Retry() commits the idempotency key before send().        ✓ FACT
  s4  "As shown above, the key is now durable."                 ✗ META   back-reference
  s5  "This is an important improvement."                       ✗ META   unfalsifiable self-praise
  s6  "The diff below shows the ordering change."               ✗ META   index into the artifact
  s7  session_id derives from the tenant salt.                  ✓ FACT
  s8  "Now that we've covered the key, let's look at…"          ✗ META   document navigation
  s9  tests/retry_test.go pins the replay case.                 ✓ FACT
  s10 "I hope this clarifies the change."                       ✗ META   courtesy close
  ────────────────────────────────────────────────────────────────────────────
  Attention ledger: 7 of 10 sentences are about the message. 0 of the 7 are
  testable. The reader consumes the whole block for 3 facts, and s2 commits
  you to a plan the next tool call can invalidate.

✅ AFTER — PAYLOAD ONLY

  s1  Retry() commits the idempotency key before send().        ✓ FACT
  s2  session_id derives from the tenant salt.                  ✓ FACT
  s3  tests/retry_test.go pins the replay case.                 ✓ FACT
  s4  Unverified: no integration run against a 504 path.        ✓ FACT (declared risk)
  ────────────────────────────────────────────────────────────────────────────
  Same three facts plus the residual risk. Every sentence is either verifiable
  against the artifact or explicitly marked as unverified.
```

### 1.4 The engineering cost ledger

1. **Attention is zero-sum.** The reviewer's skim budget is fixed; every roadmap sentence is a slot a risk sentence did not get. In PR review this is the mechanism by which a correct, high-consequence objection goes unread.
2. **Meta claims are unauditable, and they dilute auditability.** A payload sentence can be refuted by a diff, a test, or a log. `"This is an important improvement"` cannot. A response that is 60% meta is 60% unverifiable by construction — so the *ratio* of meta sentences is a direct measure of how much of the output a human must take on faith.
3. **Double navigation is a stack, not a queue.** `"In this section I will show X"` forces the reader to push the announcement, hold it, then match it against the payload that arrives later. The declaration and the delivery are never adjacent, so the announcement is re-parsed on every interruption.
4. **Announcements are promises about a future you do not control.** A plan stated before the code is read is a plan formed without the code. When the next tool result contradicts it, the announcement becomes false and the reader re-reads looking for the promised section — the highest-cost possible failure mode for review prose.
5. **Agent-specific: meta is a hallucination habitat.** Structure that is asserted before it exists (`"First I'll extract the interface, then update the call sites"`) can be generated faster than it can be verified. Meta layers let the model *appear* to work — verbosity as theatre — while the testable surface shrinks.
6. **Transcript context tax.** In a multi-turn agent session, every meta sentence is re-read by the model in every subsequent turn. Self-narration (`Let me check the config now`) duplicates a tool call that is already recorded, and stale narrations later read as commitments.
7. **Reference decay under pagination.** `As shown above` survives only in a linear scroll. Review threads paginate, docs are deep-linked, RFCs are read out of order months later. Symbol-and-path references survive; positional prose references do not.

### 1.5 Deletion over rewording

The repair is almost never a rewrite. Rewording `"In this section I will walk through the retry fix"` into `"The retry fix commits the key first"` is not the same operation as deleting it and letting the fix sentence stand — the second preserves the fact and drops the slot. Apply **delete first, then check whether anything is missing**; add back only what turns out to be a fact.

### 1.6 Boundary discipline

Zero meta-discourse governs **one variable: the referent of each sentence** (self vs. system). It does not replace its siblings, and it must not be over-applied into facts.

* **Rationale is payload, not meta.** `"because the key must be committed before the request, the replay is idempotent"` states a causal property of the system. Deleting it is a fact loss, not a cleanup.
* **Permitted announcements are the ones whose content is the system.** A theme-preview roadmap or a cathedral-taxonomy enumeration states the *domain* sequence or component set ([Theme Preview Roadmap](../../kirby-fitzpatrick-theme-preview-roadmap/SKILL.md), [Cathedral Taxonomy](../../kirby-fitzpatrick-cathedral-taxonomy/SKILL.md)). `"The three gates are lint, unit, and e2e, in that order"` is a fact about the pipeline. `"In this section I will discuss the three gates"` is a fact about the document.
* **Process narration is licensed only where the artifact is invisible.** A CI log line (`Deploying to staging`) is an event, not discourse. Inside a delivered answer the diff and tool output are visible, so narrating them is duplication ([Decoupled Reader Delivery](../../kirby-fitzpatrick-decoupled-reader-delivery/SKILL.md) governs *which* reasoning reaches the reader; this skill governs *whether* the response talks about itself at all).
* **Adjacent but distinct siblings:** ordering of facts ([Skim-Test Outliner](../../kirby-fitzpatrick-skim-test-outliner/SKILL.md)); hedges and authority markers (**Calibrated Technical Tone**); sentence-subject axis ([Two Skis Topic Alignment](../../kirby-fitzpatrick-two-skis-topic-alignment/SKILL.md)); word-level filler ([Lexical Anti-Bloat Filter](../../kirby-fitzpatrick-lexical-anti-bloat-filter/SKILL.md)); pre-flight review pass ([Rhetorical Preflight Gate](../../kirby-fitzpatrick-rhetorical-preflight-gate/SKILL.md)).

---

## 2. Core Transformation Protocols

### Rule 1 — The Payload Test (referent audit)
For each sentence, ask: *what in the repo makes this true or false?* The sentence passes only if the answer is a file, symbol, line, test, log, or measurement. `"The above function works by…"` fails (its referent is the message). `"Retry() commits the idempotency key before send()"` passes (its referent is a call ordering). No test-finding, no sentence.

### Rule 2 — Ban Forward Announcements
Delete `In this section…`, `I will now…`, `I'm going to start by…`, `Next, we'll look at…`, `The following section discusses…`. Headings, file order, and hunk order already announce this, at zero prose cost. If the sequence genuinely needs announcing, it needs a heading — not a sentence (Rule 6).

### Rule 3 — Ban Backward Narration
Delete `As shown above…`, `The above code demonstrates…`, `This snippet handles…`, `As we saw earlier…`. Name the artifact instead: the symbol, the path, the test name. `"The above snippet handles retries"` → `"Retry() captures Retry-After and schedules one replay."`

### Rule 4 — Replace the Signpost With Question, State, or Action
Fitzpatrick's three legal substitutes:
1. **The direct conversational question** — `"How does the migration handle orphaned rows?"` instead of `"The following section discusses orphaned rows."`
2. **Collaborative spatial progression** — `"With the schema established, we arrive at the service layer."` Legal only when the arrival is a *state fact*, not a tour of the document.
3. **The immediate action** — `"Update the connection pool configuration:"` followed by the change.

Never substitute a courtesy announcement for an action the reader can already see.

### Rule 5 — Lead With the Highest-Value Fact (BLUF)
The first sentence of a response, review comment, PR body, or ADR section is the change, the finding, or the blocker. No restatement of the request, no `"Here's what I did"`, no preview of the answer's shape. The first sentence must be true of the system.

### Rule 6 — Structure Carries the Roadmap
If you feel the need to announce what the document will do, the document's ordering or headings have failed — fix the structure and delete the sentence. A document whose sections cannot be navigated by their headings is a document with uninformative headings, not one that needs a narrator.

### Rule 7 — No Process Self-Reference in the Delivered Artifact
`"Let me check the config"`, `"I've run the test suite"`, `"I'm now going to search the codebase"` duplicate records the reader already has (the tool call, its output, the diff). Keep only what the log cannot show — a discarded run whose output never landed in the artifact, or a decision that changed between runs.

### Rule 8 — No Closings
Delete `I hope this helps`, `Let me know if you have questions`, `Feel free to ask`, `In conclusion`, `Overall, this…`, `Let me know if you'd like me to continue`, and any paragraph whose job is to recap the artifact just shown. End on the last fact or the declared-uncertainty line.

### Rule 9 — Declare Uncertainty as a Fact, Never as a Mood
`"Unverified: no integration run against a 504 path"` is payload — it tells the reader exactly where the evidence stops. `"I'm fairly confident this should work"` is meta: it is unverifiable and it transfers nothing. Replace every confidence statement with the specific check that was or was not performed.

### Rule 10 — A Summary Must Add a Fact
A post-code summary is permitted only if it introduces information the artifact does not show: call-site impact, an invariant that now holds, a rollback path, a boundary condition. Restating the diff in prose is the most common form of meta-discourse, because it looks like diligence.

### 2.1 Transformation Table

| # | Anti-Pattern (meta-discourse) | Defect | Clean Replacement (payload) |
|---|---|---|---|
| 1 | *"In this section, I will walk through the database migration logic."* | Forward announcement; duplicates the heading. | *"The migration adds `tenants.salt` and backfills existing rows from the tenant ID."* |
| 2 | *"The above code snippet demonstrates how to handle WebSocket connections."* | Back-reference; the code is on screen. | *"`gateway.serve()` upgrades the connection and hands the socket to `Router.dispatch`."* |
| 3 | *"Now that we have covered the controller, the next topic to be discussed is the service layer."* | Document navigation; forces the reader to hold a second pointer. | *"`OrderService` is the only caller of `OrderRepo`; it owns the transaction boundary."* |
| 4 | *"Here's what I changed and why."* | Zero-information opener; delays the highest-value fact. | *"`retry()` now commits the idempotency key before `send()`."* |
| 5 | *"This is an important improvement / a solid refactor."* | Unfalsifiable self-praise. | *"The change removes the second key mint on the 504 replay path."* |
| 6 | *"Let me check the pool configuration."* | Narrates a tool call the reader can see. | *"`max_conns` is 8 in `config/prod.yaml`, 20 in `config/staging.yaml`."* |
| 7 | *"I think this should work, but I haven't run it."* | Mood instead of evidence boundary. | *"Unverified: no local run of `TestReplay504`."* |
| 8 | *"This PR aims to improve the retry path's correctness."* | Intent statement; the diff states it better. | *"Commit the idempotency key before `send()` — fixes double delivery on 504."* |
| 9 | *"This document describes the options we considered."* | RFC self-description. | *"Option A caps the queue at 4k writes/s; Option B at 1.1k writes/s."* |
| 10 | *"In conclusion, the migration is safe and well-tested."* | Recap + unauditable assurance. | *"Rollback: drop `tenants.salt`; the column is unread by running code paths."* |
| 11 | *"As we saw above, `session_id` is derived from the tenant salt."* | Positional reference that breaks on pagination/deep-link. | *"`session_id` derives from the tenant salt — see `auth.SessionID`."* |
| 12 | *"Let me know if you'd like me to continue."* | Turn-management chatter inside the artifact. | *(Nothing — continue, or state the next concrete blocker as a fact.)* |

### 2.2 Banned Lexicon (fast filter)

Trigger phrases whose grammatical subject is the document, the response, the writer, or the reader's understanding:

`In this section…`, `This section will…`, `The following section discusses…`, `The next topic to be discussed is…`, `I will now…`, `Let me…`, `I'm going to…`, `Now that we've covered…`, `As shown above…`, `As we saw earlier…`, `As mentioned previously…`, `The above code/diff/snippet…`, `Here's what I…`, `This PR/ADR/document/guide aims to…`, `It's worth noting that…`, `I hope this…`, `Feel free to…`, `Let me know if…`, `In conclusion…`, `Overall, this…`, `To summarize the above…`

Note the scope rule: the phrase is banned when the referent is the discourse, and legal when it is the system. `"It's worth noting that the queue drops messages past 10k"` states a fact and is fine; `"This document discusses the queue"` does not.

### 2.3 Repair Procedure (60-second pass)

1. **Highlight by referent.** Mark every sentence whose subject is the document, the response, the writer, or the reader.
2. **Delete first.** Cut each marked sentence. Do not reword — deletion preserves facts, rewording usually preserves the meta.
3. **Apply the Payload Test** (Rule 1) to the survivors. Anything still unverifiable gets converted to a fact or an explicit `Unverified:` line.
4. **Re-sort for BLUF.** Move the highest-consequence fact to sentence one of each block.
5. **Read the first sentence of every paragraph in order.** That list should read as a fact inventory of the system, not a table of contents of the document.
6. **Count facts before and after.** The counts must match, minus anything you explicitly chose to drop. A fall in fact count means the pass over-applied into rationale (Rule 1 vs. §1.6).

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews

A review comment has exactly two payload obligations: **the defect** and **the fix**. Everything about how the reviewer noticed it — the reading path, the unrelated concern in the same file, the naming gripe discovered en route — is meta-discourse that forces the author to re-derive the anchor.

```text
❌ META-LAYERED COMMENT
  "I was just looking through the retry path and I noticed something. As you can see
   above, the key is written after the request is sent. This is a potential issue.
   I think we should maybe consider reordering this. Hope that makes sense!"

✅ PAYLOAD COMMENT (anchored to the line)
  "retry() writes `idempotency_key` after `send()`. A 504 followed by a retry
   double-delivers to the upstream. Write the key before `send()`, or gate the
   retry on `Retry-After`."
```

* **Defect → consequence → fix, in that order.** Each sentence is checkable against the file; the author can disagree with any one of them specifically.
* **No positional references.** `As you can see above` and `the code below` die the moment the thread paginates or the diff is re-generated. Use `retry()`, `internal/retry/retry.go:88`, the test name.
* **Approvals are facts too.** `"Looks good, nice work!"` is a meta-approval — it records no evidence. `"Approving: the diff matches the interface change, and the replay path is covered by TestReplay504."` tells the next reader what was actually checked.
* **One concern per comment** keeps each comment's referent stable; a comment that opens on `Close` and pivots to naming is a navigation event, not a review finding.
* **Severity framing stays on the artifact.** `"The `retryBudget` counter resets per request; under a failure loop it never exhausts"` — not `"There is a potential concern here that I think is important."`

### 3.2 PR Descriptions

The PR body is a handoff artifact read weeks later by someone deciding whether to revert. Meta-descriptions (`This PR aims to…`, `In this PR I have made the following changes…`, `## Summary: this section summarizes the change`) consume the top of the description, where the only sentence that matters belongs: what behaviour changed.

```text
❌ META PR BODY
  Title: Improving retry correctness
  ## Summary
  This PR aims to improve the retry path. As discussed in Slack, I have made the
  following changes. I hope this is okay!

✅ PAYLOAD PR BODY
  Title: Commit the idempotency key before send (fixes double delivery on 504)

  - retry() writes `idempotency_key` before `send()`, not after.
  - The 504 path replays the stored key instead of minting a second one.
  - Schema: no change. `idempotency_key` already exists on `retry_attempts`.
  - Risk: in-flight retries started before this deploy keep the old ordering.
  - Verification: `go test ./internal/retry/... -run TestReplay504` (passing, local).
```

* **Title states the change, not the aspiration.** `Improving retry correctness` is an announcement; `Commit the idempotency key before send` is a diff fact.
* **`Risk:` and `Verification:` lines are payload.** They are the two things a reviewer cannot infer from the diff, so they earn their space — unlike `I've tested it and it works`.
* **Omit empty sections rather than narrating them.** `"No testing needed for this change"` is a meta sentence about a missing section; `"No automated coverage: the path requires a live upstream"` is a fact.
* **Delete the courtesy close.** `"I hope this is okay"` and `"Let me know if you want me to split this"` are turn-management, not change description — put them in the chat, not the PR.

### 3.3 Architecture RFCs / ADRs

RFC and ADR sections are read out of order, quoted, and revisited months later. That kills two things at once: positional references (`as mentioned earlier`, `the options discussed above`) and section preambles (`This section describes the current state of our indexing`). Each section must open on a system fact and re-establish its own subject.

```text
❌ RFC SECTION — SELF-DESCRIBING
  ## Context
  This section describes the current state of our indexing. We will first look at
  the cron job, then discuss the write path, and finally the growth target.
  ## Options
  In the following section, we will analyze three options.
  ## Decision
  In this ADR, we have decided that the team will consider replacing the nightly
  cron re-index with a queue-driven approach.

✅ RFC SECTION — PAYLOAD
  ## Context
  The nightly re-index holds an ACCESS EXCLUSIVE lock for ~11 minutes.
  Write traffic stalls for the duration of the pass.
  Under the 4× write-growth target, that lock is the write-path bottleneck.

  ## Options
  Under a 4× write load, Option A caps ingestion at 4k writes/s (single mutex).
  Option B caps ingestion at 1.1k writes/s (per-shard mutex, no lock starvation).

  ## Decision
  We replace the nightly cron re-index with a queue-driven incremental re-index.
  Rejected: option A — the global mutex serializes the backfill with live writes.
```

* **Section openers are findings, not preambles.** `## Context` starts with the instability, not with an explanation that the section exists.
* **A decision is a stated change, not a record of deliberation.** `We replace X with Y` — not `we have decided that the team will consider…`, which announces and simultaneously declines to decide.
* **Options comparisons keep a shared evaluation axis** and state numbers per option, so a later reader can re-check the comparison without the surrounding narrative.
* **Open questions contain questions.** `"Does the queue guarantee at-least-once delivery on broker restart?"` is payload; `"The following questions will be discussed in review"` is a roadmap that survives only until the meeting.
* **Status fields are payload.** `Status: Proposed (2026-09-17)`, `Supersedes: ADR-014`, `Rollback: drop tenants.salt` — recorded, greppable, verifiable. `This ADR documents our thinking` is none of those.

---

## 4. Verification Checklist

- [ ] **Referent audit** — Does every sentence have the artifact (code, symbol, path, test, measurement, constraint) as its referent, with no sentence whose subject is the document, the response, the writer, or the reader's understanding?
- [ ] **Signposting eliminated** — Have all forward announcements (`In this section…`, `I will now…`) and backward narrations (`As shown above…`, `As we saw earlier…`) been deleted rather than reworded, with navigation carried by headings, symbols, and paths instead?
- [ ] **BLUF verified** — Is the first sentence of the response, comment, PR body, and each RFC/ADR section the highest-value fact (the change, the finding, or the blocker) with no warm-up or restatement of the request?
- [ ] **Facts conserved, not compressed away** — Does the surviving text contain every fact the pre-repair draft contained, with causal rationale (`because the key must be committed first…`) and constraint statements retained, and uncertainties expressed as `Unverified: <absent check>` rather than confidence mood?
- [ ] **Ending is a fact or a declared boundary** — Does the text stop on the last payload sentence or the residual-uncertainty line, with no recap of the artifact just shown, no courtesy close, and no turn-management offer?