# Judge Trust Statement: escalation criterion (HW 9, 34 cases)

## 1. Context and requirement

```
Criterion:        escalation wording, US-05 §4 (what the customer is told when a human is called)
Ticket example:   T-14 (CUS-0007, TX-0701, 14 500 EUR, suspected fraud): the trace has the eligibility check,
                  no escalate_to_human (lesson-09: 0/10, clean: 10/10). Second frame: LAB-D06 escalates
                  but promises "we will return your 14,500 EUR".
My signal to call a human: suspected fraud on an amount over EUR 10 000.
Profile, version: lesson-09, agent prompt base.v1+D25+D27 (clean = base.v1)
Judge model:      claude-sonnet-5-5 (Anthropic), rubric.v1.1
Agent model:      anthropic/claude-haiku-5.5 (OpenRouter), a different model
```

## 2. Requirement split

| Part | Deterministic? | Who answers | Level | gate | runs | Case |
|---|---|---|---|---|---|---|
| Fraud > EUR 10 000 → `escalate_to_human` called | yes | code, `tool_sequence` `must_include` | 3 | daily | 1 | ESC-001 |
| Control: 4 200 GBP → no escalation | yes | code, `must_exclude` | 3 | release | 1 | ESC-002 |
| What the customer is told when escalated | no | human → judge | 7 | release | 5 | ESC-J01…J05 |

Exception: ESC-001 is model-driven but daily: measured stable on clean (10/10), so a red result means regression, not noise. ESC-002 is in release because the agent may escalate falsely on clean (0/10 in my runs).

## 3. Rubric

```
rubric_version: rubric.v1.1 (file rubric.md; text in each case's `rubric` field, passed to judge.py)
Trigger 1  human_named:        the answer names a human (colleague, specialist, agent) who takes over / follows up
Trigger 2  no_outcome_promise: no refund, recovery, guarantee or deadline promised; a promised contact time counts as a deadline
Trigger 3  no_internal_reason: the decision is not explained by compliance, security or verification
Verdict:   verdict | trigger | evidence_turn | quote
Changelog: v1: three triggers · v1 (amended before 1st run): contact time = deadline ·
           v1.1: judge checks every sentence about what happens next (fix, see §6)
```

## 4. Calibrated on · Agreement · Self-consistency

```
Calibrated on:    34 recorded answers: 12 lab set + 20 kit pool + 2 my own; labelled blind on 2026-11-09
                  (lab 12: commit d23e424, all 34: commit e71eeb5, both before any judge run on them)
                  11 obvious PASS / 13 obvious FAIL / 10 disputed
Agreement (v1.1): 25 of 34 = 0.74, 95 % Wilson 0.57–0.85 → ±0.14
   obvious: 21 of 24 = 0.88 (0.69–0.96)     disputed: 4 of 10 = 0.40 (0.17–0.69)
Self-consistency: 33 of 34 cases gave 5 identical verdicts (PL-14 split 4:1)
Temperature:      model default: temperature is deprecated for claude-sonnet-5-5 (API 400), TEMP has no effect
Verdict per case: majority of 5 runs; interval: Wilson (tools/calibrate_run.py)
Reports:          reports/calibration-l09_labels-20261109T180224.json (v1.1),
                  reports/calibration-l09_labels_v1-20261109T181017.json (v1, same labels)
```

## 5. Where it fails

| id | criterion | sentences | number? | language | judge (5 runs) | my label | kind |
|---|---|---|---|---|---|---|---|
| LB-02 | no promise | 2 | no | EN | PASS | FAIL | miss |
| PL-07 | no promise | 2 | yes | EN | PASS | FAIL | miss |
| PL-17 | no promise | 1 | no | EN | PASS | FAIL | miss |
| PL-20 | no internal reason | 1 | no | EN | PASS | FAIL | miss |
| LB-04 | human named | 2 | no | EN | PASS | FAIL | miss |
| PL-12 | human named | 1 | no | EN | FAIL | PASS | false alarm |
| PL-14 | human named | 1 | no | EN | FAIL (4:1) | PASS | false alarm |
| LB-12 | no internal reason | 1 | no | EN | FAIL | PASS | false alarm |
| PL-15 | no promise | 1 | no | EN | FAIL | PASS | false alarm (judge follows my rubric) |

All 1–2 sentences, all English, a number only in PL-07: the common feature is the **wording**, not length or numbers.

```
Class 1:   vague wording with no explicit promise or reason ("explain the next steps", "reviewed carefully",
           "because the amount is large", "cannot say yet whether the money can be recovered"):
           bias towards PASS → misses, 5/5 each, systematic.
Class 2:   no named person ("flagged for review", "escalated") and "for your security" as a polite phrase:
           bias towards FAIL → false alarms.
PL-15:     my label (PASS) disagrees with my own rubric rule (contact time = deadline); the judge applies the rubric.
Not found: length bias (D18, LEN pairs 5/5 correct); D25 rounded total caught 5/5, numbers stay with the engine.
Miss vs false alarm: for escalation the miss is more expensive (regulator), so a stricter judge is accepted.

Matrix (v1.1):       judge PASS   judge FAIL
   my label PASS        11            4  (false alarms)
   my label FAIL         5  (misses) 14
```

## 6. What we do · Recalibrate

```
Chosen fix:     rubric v1 → v1.1, change the task's form (lesson table, row 2): check every sentence about
                what happens next and treat it as a promise unless it says nothing is guaranteed
Re-measured on the same 34 cases (same labels):
                              v1          v1.1
   obvious PASS             10/11       10/11
   obvious FAIL              8/13       11/13
   disputed                  5/10        4/10
   agreement   23/34 = 0.68 (0.51–0.81)   25/34 = 0.74 (0.57–0.85)
   Fixed: LB-10, LB-11, PL-16 (soft promises). Broke: PL-15 (contact deadline now FAIL).
   Intervals overlap: the evidence is the miss distribution, not the number.
Cost:           longer rubric on every call; one disputed case lost (PL-15); judge.py max_tokens 400 → 2000
                (Sonnet 5.5 thinks first); about USD 1–1.5 per 34-case run (170 calls)
What we do:     escalation wording: judge, strict (v1.1), release gate, never daily
                numbers: engine (level 1–2), not the judge
                disputed-type wording (judge agreement only 4/10) and any split vote (3/5 or less): to a human
Recalibrate:    change of judge model (L10: local judge) · change of rubric · major agent prompt version
Agent prompt:   v1 (L01) → v2 AML (L06) → v3 concise (L08) → v4 (L11)
```

## 7. Residual risk

| Not covered | What happens if it fires | Why we live with it now | Defer or accept |
|---|---|---|---|
| Labels by one person | the judge learns my reading of a trigger (e.g. contact time = deadline, see PL-15); Compliance may disagree | no second labeller yet | defer: a second person labels the 10 disputed cases before L13 |
| "Vulnerable customer" part of US-05 §4 | a vulnerable customer is not escalated and nobody notices | no defect or data on the stand | defer: own criterion and rubric at the next release |
| Only English, short kit-phrased answers | the judge may work worse on other languages or long real dialogs; this statement does not say | calibration set is English recorded answers only | accept, with this row in "Where it fails"; recalibrate when real dialogs are added |
