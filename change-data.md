# Phase 2: Change the Data to Coding

Keep the session from [reimplementation.md](reimplementation.md) fixed and swap the math practice set for coding data. Then review what the model does. Terms in *italics* are the paper's. Evaluation data and criteria are in [eval.md](eval.md).

## Start condition

Phase 1 has passed its verification criteria in [eval.md](eval.md).

## What changes

Only the practice set, its grader, the starting diagnosis fields and one phrase in the task text change.

| Element | Math (Phase 1) | Coding (Phase 2) |
|---|---|---|
| Practice set | 3,000 DeepScaleR problems | 3,000 problems from a pool of verified competitive-programming problems with unit tests, such as the DeepCoder dataset |
| Cleaning | Removed MATH-500, AIME, HMMT, AMC, OlympiadBench, GPQA-Diamond | Remove every coding problem used in [eval.md](eval.md), and every LiveCodeBench problem released after the base model |
| Held-out split | 1,000 from the same pool | 1,000 from the same pool, never seen by any session |
| Reserved practice problems | 300 | 300 |
| Grader | Math-Verify on the last `\boxed{}` | Run the last fenced code block after the thinking against hidden unit tests, in a sandbox with a time limit per test |
| Starting diagnosis | Accuracy, answers that run out of tokens, how many end in repetition, accuracy of finished answers | The same fields, plus counts of compile errors, wrong answers and time-outs |
| Task text | "better at competition mathematics" | "better at competitive programming" |

## What stays the same

Everything else stays as in [reimplementation.md](reimplementation.md). That includes the model, the second protocol, the budgets, the tools, the rules, the out-of-scope edits, the 4,096-token task budget and the SUBMIT or WITHDRAW step.

## Changes to the harness

1. **Grader.** Replace Math-Verify with sandboxed test execution. A failed or timed-out sandbox run counts as wrong.
2. **GENERATE.** The paired test reports pass rates instead of math accuracy, with the same statistic.
3. **Starting diagnosis file.** Add the failure type of each graded answer to the file the model reads.
4. **Time limit.** Running tests adds wall-clock time to each pass. Measure it on the base model first. If the 3.0-hour limit must grow, write down the new limit before any coding session runs.

## Sessions

Run eight sessions on the base weights with turn seeds 0 to 7. Report all eight.

Record the same items as in Phase 1: passes and code runs used, starting diagnosis, reserved check, decision, the edit as code, and the transcript.

## Review

After the evaluation in [eval.md](eval.md), review these before deciding on Phase 3.

- **Outcome.** How many sessions submit, and whether any submitted edit is significantly better on held-out code.
- **No harm.** Whether coding edits leave math accuracy within its noise floor.
- **Diagnosis.** What failure the model names in its starting diagnosis, and whether it differs from the math sessions.
- **The edit.** Which tensors it changes, and whether it resembles the math edits, such as a time-step bias change or a removed direction.
- **Process.** Whether the model reaches the same steps as on math: code that runs, paired tests, hidden-state reads, and acting on what its tests show. These are the columns of the paper's Fig. 6.

## Phase 3 (potential): New datasets

If the review warrants it, repeat this phase with one new practice pool at a time. Change only the practice set, its grader and the domain phrase in the task text. Physics is the first candidate. Its evaluation data is listed in [eval.md](eval.md).

## Risks

- **Token budget.** Code answers may need more than 4,096 tokens. If most answers in the starting diagnosis are cut at the budget, that is the failure the model will work on, as in the paper.
- **Contamination.** Older coding problems may be in the base model's training data, which can shrink the room for gains.
