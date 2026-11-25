# Evaluation: Method and Data

How every edit from [reimplementation.md](reimplementation.md) and [change-data.md](change-data.md) is evaluated. Terms in *italics* are the paper's.

## Principle

This follows the paper's *evaluation fixed in advance* (App. D.3). Before any evaluation answer is generated, write down the question, the problems, the seeds, the test and its threshold, together with the analysis code. Compute no interim result.

## Answer settings

| Setting | Value |
|---|---|
| Sampling | Temperature 0.6, top-k 20, top-p 0.95, thinking on |
| Budget | 16,384 new tokens |
| Seeds | Fixed by problem identifier and answer index, the same for base and edited model, never used in any session |
| Grading rule | No credit if the thinking is still open at the budget. Only the answer after the end of the thinking counts |
| Base model answers | Generated once per benchmark and shared by every comparison |

## Data

### Math (Phase 1, and the no-harm check in Phase 2)

| Benchmark | Problems | Answers each | Grader | Role |
|---|---|---|---|---|
| Paper's held-out split, items 500–999 minus repeats | 490 | 8 | Math-Verify, last `\boxed{}` | Primary test |
| MATH-500 | 500 | 4 | Math-Verify | Clean extra check, removed from the practice set |
| AIME 2025 | 30 | 32 | Math-Verify | Clean extra check, removed from the practice set |

### Coding (Phase 2)

| Benchmark | Problems | Answers each | Grader | Role |
|---|---|---|---|---|
| Held-out coding split from the practice pool | 490 | 8 | Hidden unit tests in a sandbox | Primary test |
| LiveCodeBench, problems released after the base model | all in window | 4 | Hidden unit tests, pass@1 | Clean external check, protected by release date |
| HumanEval+ and MBPP+ (EvalPlus) | 164 + 378 | 4 | EvalPlus tests | Easier external check |

For code, the graded program is the last fenced code block after the thinking. A failed or timed-out sandbox run counts as wrong.

### Physics (Phase 3, potential)

| Benchmark | Problems | Answers each | Grader |
|---|---|---|---|
| GPQA-Diamond, physics subset | about 86 | 16 | Exact letter match, options shuffled with a fixed seed per problem |
| OlympiadBench, text-only English physics | a few hundred | 8 | Numeric match with the benchmark's tolerance |

### Which edits run on which data

| Edits from | Math | Coding | Physics |
|---|---|---|---|
| Phase 1 (math sessions and published edits) | Yes | Yes | If Phase 3 runs |
| Phase 2 (coding sessions) | Yes, as the no-harm check | Yes | If Phase 3 runs |
| Phase 3 (new datasets) | Yes | Yes | Yes |

The result builds into a matrix of practice domain against evaluation domain.

## Statistics

- **Primary statistic.** Per problem, the share of answers the edited model solves minus the share the base model solves (the paper's d_i). Test the mean over problems with a one-sided sign-flip permutation test, 100,000 flips. An edit is significantly better if p < 0.05.
- **Interval.** Report the change with its 95% interval over problems.
- **Noise floor.** Run the base model against itself with two seed sets on every benchmark. The paper notes generation is not exactly reproducible, so this measures how much change no edit produces.
- **Power.** From the noise-floor run, compute the minimum detectable effect for each benchmark before evaluating any edit. If a benchmark cannot detect a 3-point change, raise its answers per problem first.
- **Exclusions.** None after the fact.

## Controls

| Control | What it shows |
|---|---|
| Base vs base with new seeds | How much change non-reproducible generation alone produces |
| Random edit: same tensors, same norm of change as each significant edit, five draws | Whether the model's own choice of edit matters |
| Re-derived edits E-S2 and E-FP | Whether the gain needs the model's own reasoning, or only its recipe |

## What to report for every edit and benchmark

- accuracy of base and edited model,
- the change with its 95% interval and p,
- counts of answers that turn wrong-to-right and right-to-wrong,
- the share of answers that run out of tokens, as a description only.

## Hypotheses

| ID | Hypothesis | Supported if |
|---|---|---|
| **H1** | Phase 1 reproduces the paper: the model's own edit makes it more accurate on new math problems | The verification criteria below hold |
| **H2** | A session on a coding practice set finds an edit that makes the model more accurate on new coding problems | At least one submitted edit has p < 0.05 on the 490 held-out coding problems |
| **H3** | Coding edits do no harm on math | The lower 95% bound of the math change is above −2 points |
| **H4** | Deferred | Not tested in these runs¹ |

## Phase 1 verification criteria

Phase 1 passes if all of these hold.

| Check | Paper's result | Criterion |
|---|---|---|
| Base model on the 490 problems | 57.2% | Within the noise floor of the base-vs-base run |
| E-S8 | +6.6 points, 95% interval 5.0 to 8.1, p < 10⁻⁵ | Our 95% interval overlaps the paper's, and p < 0.05 |
| New sessions | 3 of 8 submit, 2 of the 3 significant | At least one submitted edit significant at p < 0.05. The withdraw count is reported, not required to match |
| Every submitted edit | The model in use only gets better | No submitted edit is significantly worse than the base model |

---

¹ **H4, for later runs.** The paper's own diagnosis is *answers that run out of tokens*, so an edit may act as a learned stopping rule rather than better reasoning. A later run can test this. Re-evaluate the base model and the best edits at 32,768 tokens as well as 16,384, and check whether the gain shrinks. Report accuracy on finished answers separately. Add a decoding-time control, such as a repetition penalty or thinking-budget forcing, tuned on practice problems to cut answers that run out of tokens by the same share as the edit. H4 is supported if the gain shrinks at the larger budget, accuracy on finished answers does not rise, and the decoding-time control matches the edit.
