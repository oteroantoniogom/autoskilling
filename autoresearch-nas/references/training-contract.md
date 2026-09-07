# Training and evaluation contract

## Required project roles

Map the user's local code to these roles. The filenames below are examples, not required dependencies.

| Role | Responsibility | Search permissions |
| --- | --- | --- |
| Data preparation | Produce versioned training, validation, and optional final-test splits; fit preprocessing on training data | Freeze before ranking candidates |
| Model builder | Construct a network from a recorded architecture description | Editable |
| Trainer | Initialize weights and optimizer, consume training data, enforce budget, save checkpoints | Adapt for new modules while preserving the comparison contract |
| Evaluator | Compute the declared metric over fixed examples | Protected during a comparison group |
| Trial runner | Start an owned process, enforce an outer timeout, save logs and exit status | May change without changing the meaning of a score |
| Session ledger | Store hypotheses, revisions, metrics, decisions, and resource use | Append results after every attempt |

Use the project's existing commands and dependencies. If no repeatable training entry point exists, implement one in the authorized project with arguments for architecture configuration, seed, training budget, and output directory. Record the exact command in every trial. Do not clone another training repository as a prerequisite to this skill.

Before training, verify the task's data, baseline model, objective, and target device are available. Infer these from the local project when possible. If any is missing, prepare the parts that are known and ask for the missing task input. A skill can define the workflow but cannot supply an unspecified dataset or hardware.

## Model and optimizer interface

The model must accept the same input layout and produce the same task outputs across candidates. For language modeling, input token IDs usually have shape `[batch, sequence]`, and logits have shape `[batch, sequence, vocabulary]`. Preserve the target shift, padding mask, and causal prediction rule. Other domains use the project's equivalent output contract.

Each structural mutation must update model construction, forward execution, initialization, optimizer grouping, checkpoint save/load, and resource accounting where affected. Inspect parameter identity rather than names alone when weights are shared. Confirm every intended trainable parameter belongs to exactly one optimizer group. Check nonzero gradients over enough steps to account for deliberately zero-initialized branches.

A checkpoint should contain the architecture descriptor, model state, preprocessing identity, and code revision. Include optimizer state, scheduler state, data position, random-generator states, and completed budget when exact training resumption is required. Do not load incompatible weights silently into a changed graph.

## Establish the baseline

Run the unmodified architecture through the same validity checks, training budget, and evaluator used for candidates. Save raw output and configuration. Measure startup, compilation, training, and evaluation time separately. A baseline failure is a setup problem; resolve it before interpreting candidate scores.

For fixed-time training, use a monotonic clock. Synchronize accelerator work at timing boundaries. End after the first completed step that reaches the budget and record actual elapsed time and overshoot. Declare a tolerance for step-boundary overshoot before comparison; a model with very long steps must not receive substantial extra compute unnoticed. Choose whether warm-up updates count toward training and apply the same rule to every candidate.

An outer process deadline includes startup and evaluation. It must fit the remaining session budget. Failed runs, retries, and data preparation still consume session resources even if excluded from the training timer.

For fixed-token or fixed-example comparisons, hold the processed data budget constant, including accumulation and distributed-worker accounting. Fixed steps alone do not equalize training exposure when batch size changes.

## Protected evaluation

Run evaluation with training-only randomness disabled and gradients off. Use the same examples, order, masking, reduction, and preprocessing for every candidate. Aggregate sums and counts rather than averaging unequal batch means. Verify all required outputs and metrics are finite.

For a language-model bits-per-byte metric, sum the negative log probabilities in natural-log units for the evaluated target tokens. Divide by `ln(2)` and by the number of original data bytes represented by those targets. Explicitly define how special tokens and boundaries contribute to scoring and byte counts. Reuse a working evaluator rather than replacing it during search. A lower score is better. A changed tokenizer or validation corpus requires a new comparison group even when the metric is byte-normalized.

For classification, segmentation, regression, or another task, retain the existing metric and its direction. Keep architecture-specific auxiliary training losses out of the primary evaluation metric unless the contract explicitly includes them.

## Trial lifecycle

```text
verify remaining budget and reserve confirmation time
select a parent and write a structural hypothesis
save candidate code revision and architecture descriptor
run construction, forward, backward, and task-validity checks
if checks fail:
    record invalid attempt with null score
else:
    launch training with an owned process and deadline
    save raw logs, exit status, checkpoint, and resource measurements
    if training and protected evaluation complete successfully:
        record finite metric with seed and fidelity
        compare only with compatible baseline and candidate records
        retain, promote, or archive with a written reason
    else:
        record explicit failure status with null score
append evidence and atomically update session state
```

Training success requires a successful process exit and complete valid evaluation evidence. A score printed before a later crash is not a completed trial. Do not infer success merely from a checkpoint file.

Keep rejected revisions reachable or preserve their patches. Retain commands and environment details needed to reproduce each result. The final report must distinguish architecture improvements from hardware, data, recipe, or measurement changes.
