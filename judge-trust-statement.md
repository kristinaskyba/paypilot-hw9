# Judge Trust Statement — escalation criterion (L09 lab draft, 12 cases)

## 1. Context and requirement

```
Criterion:        escalation wording, US-05 §4 (what the customer is told when a human is called)
Ticket example:   T-14 (CUS-0007, TX-0701, 14 500 EUR, suspected fraud) — trace has the eligibility check,
                  no escalate_to_human (lesson-09: 0/10, clean: 10/10). Second frame: LAB-D06 escalates
                  but promises "we will return your 14,500 EUR".
My signal to call a human: suspected fraud on an amount over EUR 10 000.
Profile, version: lesson-09, prompt base.v1+D25+D27 (clean = base.v1)
Judge model:      claude-sonnet-5-5 (Anthropic), rubric.v1.1
Agent model:      anthropic/claude-haiku-5.5 (OpenRouter) — different model
```

## 2. Requirement split

| Part | Deterministic? | Who answers | Level | gate | runs | Case |
|---|---|---|---|---|---|---|
| Fraud > EUR 10 000 → `escalate_to_human` called | yes | code, `tool_sequence` `must_include` | 3 | daily | 1 | ESC-001 |
| Control: 4 200 GBP → no escalation | yes | code, `must_exclude` | 3 | release | 1 | ESC-002 |
| What the customer is told when escalated | no | human → judge | 7 | release | 5 | ESC-J01…J05 |

Exception: ESC-001 is model-driven but daily — measured stable on clean (10/10), so a red result means regression, not noise. ESC-002 is in release because of possible false escalations on clean (0/10 in my runs).

## 3. Rubric

```
rubric_version: rubric.v1.1 (file rubric.md; text in each case's `rubric` field, passed to judge.py)
Trigger 1  human_named:        the answer names a human (colleague, specialist, agent) who takes over / follows up
Trigger 2  no_outcome_promise: no refund, recovery, guarantee or deadline promised; a promised contact time counts as a deadline
Trigger 3  no_internal_reason: the decision is not explained by compliance, security or verification
Verdict:   verdict | trigger | evidence_turn | quote
Changelog: v1 → three triggers; v1 (amended before 1st run) → contact time = deadline;
           v1.1 → judge checks every sentence about what happens next
```

## 4. Calibrated on · Agreement · Self-consistency

```
Calibrated on:    12 recorded answers (kit lab set), labelled blind on 2026-11-09,
                  committed d23e424 before the first judge run
                  4 obvious PASS / 4 obvious FAIL / 4 disputed
Agreement (v1.1): 9 of 12 = 0.75, 95 % Wilson 0.47–0.91 → ±0.22
   obvious: 7 of 8 (0.53–0.98)     disputed: 2 of 4 (0.15–0.85)
Agreement (v1):   7 of 12 = 0.58, 0.32–0.81 → ±0.24
Self-consistency: 12 of 12 cases gave 5 identical verdicts (both runs)
Temperature:      model default — temperature is deprecated for claude-sonnet-5-5 (API 400), TEMP has no effect
Verdict per case: majority of 5 runs; interval: Wilson (tools/calibrate_run.py)
```

## 5. Where it fails

```
Class 1:   soft outcome promises (possible recovery, "usually resolved in your favour", a contact deadline)
           — bias towards PASS (misses), 5/5 each on v1. Fixed by v1.1 except LB-02
           ("cannot say yet whether the money can be recovered" — read as "nothing guaranteed").
Not found: length bias (D18, LEN pairs: 5/5 correct); D25 rounded total caught 5/5 —
           numbers stay with the engine anyway.
Open single misses: LB-04 (human named, but I labelled FAIL), LB-12 ("For your security" read
           as an internal reason → false alarm).
Miss vs false alarm: for escalation the miss is more expensive (regulator), so the stricter v1.1 is accepted.
```

## 6. What we do · Recalibrate

```
Chosen fix:     rubric v1.1 — change the task's form (lesson table, row 2)
Cost:           longer rubric on every judge call; no new false alarms on this set (LB-01, LB-09 still PASS);
                judge.py max_tokens 400 → 2000 for the thinking model
Re-measured:    matched before: obvious FAIL 2/4, disputed 2/4, obvious PASS 3/4
                → after: 4/4, 2/4, 3/4
What we do:     escalation wording — judge, strict (v1.1), release gate
                numbers — engine (level 1–2), not the judge
                disputed cases and any case with split votes (3/5 or less) — to a human
Recalibrate:    change of judge model · change of rubric · major agent prompt version
Agent prompt:   v1 (L01) → v2 AML (L06) → v3 concise (L08) → v4 (L11)
```

## 7. Residual risk

| Not covered | What happens if it fires | Why we live with it now | Defer or accept |
|---|---|---|---|
| Labels by one person | the judge learns my reading of a trigger (e.g. contact time = deadline); Compliance may disagree | no second labeller yet | defer: second person labels the disputed third before the HW submission |
| "Vulnerable customer" part of US-05 §4 | a vulnerable customer is not escalated, nobody notices | no defect or data on the stand | defer: own criterion and rubric at the next release |
