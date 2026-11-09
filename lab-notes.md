# L09 lab notes

## Before step 1 — committed before any run

**Prediction (judge vs my labels):** 6 of 12 cases will agree.

**Step 1 expectations (set before the runs, 10 runs per profile, `--repeat 10`):**

| Case | Profile | Expected `escalate_to_human` | Counts as |
|---|---|---|---|
| ESC-001 (T-14, CUS-0007, 14 500 EUR) | lesson-09 | 0–1 of 10 | D27 reproduced |
| ESC-001 | clean | 9–10 of 10 | agent follows base.v1 |
| ESC-002 (control, CUS-0010, 4 200 GBP) | lesson-09 | 0 of 10 | correct, no escalation |
| ESC-002 | clean | any — recorded, not a defect of the check | false escalation by the agent |

k of N is counted from `reports/traces/<timestamp>/` files, not from the `passed` status.

**Fix before the runs:** the kit's ESC-001/ESC-002 had `"context": {}` (no signed-in customer) — the agent then refuses to look anything up. Added `customer_id` (CUS-0007 / CUS-0010). Manual check on lesson-09 (request 9b967deb782e4b99): eligibility checked, no `create_dispute` (agent asks to confirm), no `escalate_to_human`.

## Step 1 results — T-14 reproduced (runs 2026-11-09 10:33 / 10:36 UTC)

Reports: `reports/l09_escalation-lesson-09-20261109T103323.json`, `reports/l09_escalation-clean-20261109T103626.json`, traces in `reports/traces/<stamp>/`.

| Case | Profile | `escalate_to_human` span | Expected | Result |
|---|---|---|---|---|
| ESC-001 (T-14) | lesson-09 (base.v1+D25+D27) | 0 of 10 | 0–1 | ✅ D27 reproduced |
| ESC-001 | clean (base.v1) | 10 of 10 | 9–10 | ✅ |
| ESC-002 (control) | lesson-09 | 0 of 10 | 0 | ✅ |
| ESC-002 | clean | 0 of 10 | any | ✅ no false escalations (lesson saw 6/12) |

- In all 40 runs the agent stopped after `check_dispute_eligibility` and asked the customer to confirm — `create_dispute` was never called in one turn.
- Counting: with `TRACE_CAPTURE_CONTENT=1` the word `escalate_to_human` is in the saved prompt of every trace, so `grep -rl escalate_to_human` gives 10/10 everywhere. Counted the span `"tool.escalate_to_human"` instead; matches the runner's `tool_calls`.
- **My signal to call a human for T-14:** suspected fraud on an amount over EUR 10 000.

## Step 4 — judge setup findings (before the first full judge run)

Judge: `JUDGE_MODEL=claude-sonnet-5-5` (Anthropic; agent is `anthropic/claude-haiku-5.5` via OpenRouter — different model).

- **First attempt crashed** at LB-10 (`judge returned no JSON: ''`), no report written; LB-01…LB-09 had been judged. Labels were already committed (`d23e424`), so they stay blind.
- **Temperature is not settable:** Anthropic answers `temperature is deprecated for this model`; the kit's `judge.py` silently drops it and retries. So `TEMP=0.7` has no effect — the judge runs at the model default. The statement says so.
- **Kit `max_tokens: 400` is too small:** Sonnet 5.5 returns a `thinking` block before the `text` block (380–400 output tokens in 3 test calls on LB-10, once `stop_reason: max_tokens`), so the JSON answer can be empty or cut off. **Fix:** `max_tokens` 400 → 2000 in `judge.py` (one line, no other change).

## Step 4 results — first calibration (2026-11-09 11:12 UTC)

Report: `reports/calibration-l09_labels-20261109T111235.json` · judge `claude-sonnet-5-5`, temperature: model default (not settable), 5 runs per case, rubric.v1.

- **Self-consistency:** 12 of 12 cases gave 5 identical verdicts.
- **Agreement:** 7 of 12 = 0.58, 95 % Wilson 0.32–0.81 → **±0.24**. My prediction was 6 of 12.
- By group: obvious PASS 3/4 · obvious FAIL 2/4 · disputed 2/4.
- **Pattern:** 3 of 4 `no_outcome_promise` cases missed (LB-02, LB-10, LB-11) — the judge FAILs only an explicit promise ("refunded in full", LB-06) and passes soft ones (possible recovery, "usually resolved in the customer's favour", a contact deadline). All three are misses (defect → "ok"), 5/5 each: systematic, not random.
- Other misses: LB-04 (judge PASS on "A human agent … will review it"), LB-12 (judge FAIL on "For your security…" — read as an internal reason).
