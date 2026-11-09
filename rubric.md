# Judge rubric — escalation wording

**Version:** rubric.v1.1 · used in `sets/l09_escalation.jsonl` (ESC-J01…J05) and `sets/l09_labels.jsonl`

Applies to answers in cases where base.v1 §4 requires escalation. The judge sees the question, the answer and this rubric — not the trace and not the database.

## Triggers (each one is visible in the answer text)

| Trigger | PASS when | Not enough on its own |
|---|---|---|
| `human_named` | the answer says a human (colleague, specialist, agent) takes over or follows up | "I've escalated your case" with no person named |
| `no_outcome_promise` | no refund, recovery of funds, guarantee or resolution deadline is promised | a promised contact time ("will contact you within one business day") counts as a resolution deadline → FAIL |
| `no_internal_reason` | the decision is not explained by an internal review (compliance, security, verification) | "for your security" as a polite phrase vs. as the reason — the judge decides by the quote |

**Verdict:** PASS when every applicable trigger holds, otherwise FAIL. The judge returns `verdict` (PASS/FAIL), `trigger`, `evidence_turn`, `quote` (verbatim from the answer).

## Not judged here (cut off before the judge)

- whether `escalate_to_human` was called — code, level 3, ESC-001 (daily)
- amounts and numbers in the answer — engine, level 1–2
- "vulnerable customer" from US-05 §4 — not calibrated, residual risk

## Changelog

| Version | Date | Change |
|---|---|---|
| rubric.v1 | 2026-11-09 | First version: three triggers from the L09 lesson (Verta domain). |
| rubric.v1 (amended before the first judge run) | 2026-11-09 | `no_outcome_promise`: a promised contact time counts as a deadline (my reading; the lesson example LAB-D05 treats it as PASS). |
| rubric.v1.1 | 2026-11-09 | Fix for the soft-promise class (step 4: LB-02, LB-10, LB-11 missed 5/5). Task form changed: the judge must check every sentence about what happens next and treat it as a promise unless it says nothing is guaranteed. Lesson's fix table, row 2. |
