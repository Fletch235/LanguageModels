# Phase 1: Reimplementation

Rebuild the paper's setting and reproduce its math results before changing anything. Terms in *italics* are the paper's. Evaluation data, statistics and pass criteria are in [eval.md](eval.md). Hardware and environment are in [README.md](README.md).

## What the paper does

The paper gives Qwen3.5-27B its own weights θ and a *session* of turns, with no gradient, no optimizer and no stronger model. In each turn the model writes a short rationale and takes one action.

| Action | What it does |
|---|---|
| CODE | Runs Python on a CPU over its weights. It can read tensors and write new values into them |
| GENERATE | Answers practice problems with the unedited weights, the edited weights, or both on the same seeds. Both together form a *paired test* |
| READ | Runs the weights over texts the model chooses and returns hidden states, MLP activations or token log-probabilities |

The model writes an *edit* Δ, and the goal is

  Acc(θ ⊕ Δ; D_test) > Acc(θ; D_test)          (paper, Eq. 2)

on problems no session saw.

## Settings to reproduce

| Element | Setting |
|---|---|
| Model | Qwen3.5-27B, thinking on |
| Protocol | *Second protocol* (App. E.9) |
| Budgets | 20 forward passes, 200 code runs, 3.0 hours |
| Task budget | 4,096 new tokens per answer inside the session |
| Forward pass size | At most 128 answers in all |
| Side-by-side tests | Up to four saved edits against the unedited model, same problems and seeds |
| Starting diagnosis | 256 practice problems: accuracy, answers that run out of tokens, how many end in repetition, accuracy of finished answers |
| Practice set | 3,000 DeepScaleR problems, cleaned of MATH-500, AIME 2024/2025, HMMT Feb 2025, AMC 2023, OlympiadBench and GPQA-Diamond |
| Reserved practice problems | 300, which the session can neither read nor use |
| Final check | The framework tests the kept edit on 256 reserved problems, two answers each at 4,096 tokens. The model then answers SUBMIT or WITHDRAW |
| Answer settings | Temperature 0.6, top-k 20, top-p 0.95 |
| Grading | Math-Verify on the last `\boxed{}` after the thinking. No credit if the thinking is still open at the budget |
| Out-of-scope edits | Input embedding, final normalisation, output layer, rescaling a whole tensor |
| Rules | No gradients, optimizers or training of any kind, no running the network inside Python, no internet |
| Serving | vLLM, with weights and saved edits in bf16. The model that edits runs in the same engine, reset to its own weights before each of its replies. Pin the vLLM version and record it with every result |
| Code runs | Each stops after 600 s, and its output is cut at 10,000 characters |
| Model replies | Up to 12,000 new tokens each. Past 90,000 tokens of context, the conversation restarts from the task, the model's notes and its log of passes |
| Task text | Paraphrased in App. D.4, including the instruction to test each part of an edit separately |

## Step 1a: Re-apply the published edits, with no session

This checks the evaluation pipeline before the session harness exists. It needs only weight loading and vLLM.

| Edit | Source | How exactly it can be rebuilt |
|---|---|---|
| **E-S8** | Second protocol, session 8: *time-step bias +0.5 in all 48 linear-attention layers* | Exactly, from the paper's text |
| **E-S2** | Second protocol, session 2: layer 63 output matrices, projection on one measured direction subtracted 5× | Re-derived: the direction between finished and unfinished practice answers at layer 63 |
| **E-FP** | First protocol: W ← W − 0.05·uuᵀW in the two output matrices of layer 8 | Re-derived: the direction from 20 finished and 20 unfinished answers at layer 8 |

Re-derived edits follow the model's recipe but are not the model's own edit, so they are labelled as such.

Before applying E-S8, confirm that the paper's *time-step bias* is the Gated DeltaNet time-step bias parameter in the Qwen3.5 checkpoint.

If E-S8 fails its criterion in [eval.md](eval.md), stop here. Check the parameter name, the number format and the vLLM version.

## Step 1b: Rebuild the session harness

Build these parts from Appendices D and E.9.

1. **Python session.** It runs on the CPU, keeps its state between runs and has gradients off. It refuses gradient calls, optimizers, model loading, new processes and network use.
2. **Helpers given to the model.** Tensor names and shapes, load and save of tensors, save, drop and list of edits, the tokenizer, the practice problems, and loaders for pass results. Four small helpers: the output matrices of a layer, a unit vector, the unit difference of two group means, and W − uuᵀW.
3. **Edit store.** Saved edits are whole tensors of the same shape with finite values, stored in bf16. Out-of-scope tensors are refused.
4. **GENERATE.** Runs vLLM with the unedited weights, the edited weights or both on the same seeds. It reports the paired change in accuracy with its standard error (Eq. 3).
5. **READ.** Runs the weights over up to 256 chosen texts, each cut at 8,192 tokens.
6. **Reserved check.** Tests the kept edit on 256 reserved problems before the SUBMIT or WITHDRAW step.
7. **Session loop.** Counts passes and code runs, enforces the time limit, restarts the conversation past 90,000 tokens from the task, the model's notes and its log of passes, and logs every turn.

## Step 1c: Run new sessions

Run eight sessions on the base weights with turn seeds 0 to 7, the paper's seeds. Report all eight, whether they submit or withdraw.

Record for each session:

- the passes and code runs used,
- the starting diagnosis,
- the result of the reserved check,
- the decision, SUBMIT or WITHDRAW,
- the edit as the code the model wrote,
- the full transcript of turns.

Every submitted edit goes to the evaluation in [eval.md](eval.md).

## Done when

Phase 1 is done when the verification criteria in [eval.md](eval.md) hold. Phase 2 does not start until then.
