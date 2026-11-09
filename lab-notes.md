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
