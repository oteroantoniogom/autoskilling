# Architecture research

## Turn the problem into structural hypotheses

Describe the input axes, prediction target, available information at prediction time, missingness, required invariances, context length, and deployment constraints. Distinguish an input's observation time from its arrival time. Identify which dependencies a model must represent: local patterns, long-range recall, interactions between entities, or persistent state.

Map each proposed module to a specific need. Example: intermittent sources may need an availability mask and time-since-observation encoding; attention alone does not establish correct missing-data handling. Start with a simple working baseline and a task-appropriate non-neural baseline when useful.

Keep a module inventory with each component's role, input/output contract, parameters, trainable or frozen status, state, cost, hypothesis, and observed evidence. Supported roles can include encoders, token mixers, feature mixers, memory, fusion, and prediction heads. Avoid assuming every role needs a separate module.

## Decompose a Transformer

Treat these as separable design decisions rather than an indivisible block:

- Input projection or embedding and positional or elapsed-time encoding.
- Query, key, and value projections, head grouping, score scaling, attention normalization, masking, and output projection.
- Feed-forward expansion, activation or gating, and contraction.
- Normalization placement, residual paths, gates, branch composition, and parameter sharing.
- Layer schedule, memory or recurrence, pooling, and task heads.

Record which dimensions refer to attention heads, feature channels, or separate prediction heads. Three prediction heads with different widths do not necessarily imply three attention heads. Define each head's target, loss weighting, and merge rule. Joint-loss changes are training confounds unless explicitly included in the experiment.

Compare trainable and frozen modules under a stated protocol. For random reservoir components, record construction, seed, scale, connectivity, and which readout or projections are trained. A frozen random Q/K experiment tests a particular inductive bias and compute tradeoff; it does not establish that attention generally requires no learning.

For recurrence or memory, define state initialization, update, reset, truncation, and inference behavior. Reset state across independent examples and data splits. If cross-example state is intentional, specify its causal ordering and reproduce it in evaluation.

## Establish what each piece contributes

Write a hypothesis before implementation. Include a control, expected behavior, measurable success criterion, and likely confounds. Compare substitution and removal ablations where they answer different questions. When a combined architecture wins, remove components individually and test interactions that could explain the gain. Keep equal opportunity for baseline tuning.

Do not confuse increasing capacity with discovering a useful mechanism. Where affordable, compare a simpler model with a similar parameter or compute budget. Report both resource-matched and deployment-budget comparisons when they support different conclusions.

Record findings as supported, contradicted, or unresolved under the tested conditions. An unsuccessful short run is not proof that a primitive cannot work. Export promising modules with their constraints into the versioned NAS space, and keep uncertain alternatives only when exploration budget permits.

## Domain knowledge

Feature engineering, structural priors, and loss constraints are different interventions. Record them separately. A physics-informed neural network incorporates explicit physical relationships, such as conservation laws or differential-equation residuals, into training or construction. For any imposed constraint, state its units, assumptions, applicable domain, and how its contribution will be measured.
