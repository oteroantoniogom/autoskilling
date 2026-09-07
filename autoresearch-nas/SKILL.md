---
name: autoresearch-nas
description: Investigate neural architectures and automate architecture search with explicit module graphs, valid mutations, controlled ablations, and reproducible evaluation. Use for architecture research, neural architecture search or NAS, evolutionary network design, and autoresearch focused on network structure.
---

# Architecture research and neural architecture search

Search network structure through a propose, edit, train, measure, retain loop. Produce reproducible architecture candidates, measured comparisons, and a reviewable winning implementation. Define an explicit search space, cover structural alternatives, validate candidate models, and confirm promising results. All workflow instructions are included in this skill and its bundled references; no external repository, website, or companion skill is required.

## Choose the working mode

Architecture research develops and tests hypotheses about network structure. Neural architecture search automates exploration within a defined space of structures. Treat "architectural search" as architecture search, not a separate technique.

- For questions about what structure or information flow a problem needs, use research mode. Read [architecture research](references/architecture-research.md), run focused comparisons when authorized, and produce an evidence-backed inventory of useful modules and unresolved hypotheses.
- For requests to automate exploration, use search mode. Read [architecture representation and mutations](references/architecture-language.md), define valid candidates and a search policy, and run the budgeted trial loop.
- For combined work, investigate uncertain primitives first, then freeze a versioned search space and automate it. Reopen research when results expose a missing primitive or invalid assumption. Version the space when it changes and preserve earlier evidence.

Scale the work to the request. A focused ablation does not require building a search engine. A request to design the engine calls for an executable representation, mutation implementation, and validation in the user's project, not merely a list of candidate architectures. Creating those components does not authorize a large training run.

## Establish the search

Read the user's local project instructions, model, training entry point, and evaluation code. Identify the actual device and available kernels before choosing candidates. Map the project to [the training and evaluation contract](references/training-contract.md). That reference also explains the minimum setup when the project lacks a repeatable training command. Use existing files where possible.

Honor the user's chosen objective, model family, hardware, and budget. Infer routine choices from the project and state assumptions briefly. A request to create a skill or design a search does not launch training. When asked to run a search, proceed through authorized setup and experiments without asking whether to continue after each trial. If no run budget is available, finish inspection and prepare the search contract, then ask for the compute or time limit before launching sustained training. Do not provision paid compute from an unspecified budget.

Write a session contract before ranking candidates. Record:

- Dataset and split fingerprints, preprocessing, tokenizer, input shapes, evaluation command and code hash, metric direction, and loss definition.
- Editable model files, protected evaluation files, allowed dependencies, architecture families, and hardware constraints.
- The comparison basis: fixed training wall time, fixed training tokens or steps, or estimated compute. Specify what the timer includes. Keep the basis fixed within each comparison group.
- Trial and session limits, memory limit, per-process timeout, seeds, confirmation allocation, and a minimum useful improvement or explicit exploratory selection rule.

Use the repository's existing session format when suitable. Otherwise keep `contract.json`, `state.json`, an append-only `trials.jsonl`, and trial artifacts in `.nas/<session>/`. Save full candidate commits or patches, commands, logs, and architecture descriptions. See [search design and trial records](references/search-protocol.md) when setting up or resuming a run.

## Search the architecture

Describe the baseline as a computation graph or module hierarchy. For automatic assembly, record typed module ports, tensor axes, state semantics, and mutation preconditions using the bundled architecture representation. Validate new mutation operators on small synthetic tasks before scaling an engine; reuse prior validation when its assumptions still hold. Separate structural choices from the training recipe. Search operators, connectivity, block order, sharing, and conditional computation as well as depth and width. A run of only learning-rate, batch-size, or optimizer changes does not satisfy an architecture-search request.

Create a short queue of hypotheses using the applicable families in [search design](references/search-protocol.md). Each candidate needs a parent, a concrete structural change, a predicted benefit, a likely failure mode, and the measurement that would support it. Prefer one interpretable structural change per early trial. Combine supported changes later and run ablations on the combination. For a small budget, test fewer meaningful structural alternatives instead of a large numeric grid.

Keep the training recipe fixed during initial structural screening where viable. Record any necessary recipe change as a confound. If architectures need tuning, allocate comparable tuning budgets to the baseline and finalists, and report both the controlled comparison and the tuned system comparison.

Use a small population of promising and structurally distinct candidates when the budget permits. Retain useful nondominated candidates for quality, latency, and memory. Do not discard every stepping stone merely because it fails to beat the incumbent on its first short run. Bound exploration by the session budget; random search over the same architecture space can serve as a reference when affordable.

## Execute each trial

1. Choose a parent using observed results and remaining budget. Check session stop conditions before launching work. Reserve time for confirmation and final reporting.
2. Implement the candidate in an isolated branch or worktree. Preserve user changes. Record an architecture description and code revision; equivalent descriptions alone do not prove code equivalence.
3. Validate model construction, supported shapes and dtypes, finite forward loss, backward execution, and optimizer coverage. Check tied parameters are counted and updated once. For causal tasks, perturb future input tokens and verify earlier logits remain unchanged in evaluation mode. Check masking, padding, and recurrent state resets where relevant.
4. Run a cheap feasibility trial for memory, throughput, and numerical stability. Record it as a separate fidelity. Compilation, evaluation, and warm-up costs count toward total session spend even when excluded from the training timer.
5. Train eligible candidates at the contract's screening budget using the protected evaluator. Save raw logs and actual resource consumption. Mark OOM, NaN, timeout, interruption, and implementation failures explicitly, with a null score. A failed run never receives an artificially favorable numeric score.
6. Compare only compatible trials. Promote promising candidates to the same higher budget as their comparison baseline. Label short-run findings provisional. Revisit some structurally different candidates when early learning speed might mislead selection.
7. Append the trial record and update the incumbent, Pareto set, hypothesis queue, and remaining budget. Keep losing evidence. Restore only agent-owned trial changes, using scoped reverts or the isolated worktree rather than resetting a shared checkout.

Use an outer process timeout in addition to the training timer. Terminate only trial-owned processes. Bound retries, normally one fix and rerun for an implementation error; charge both attempts to the budget. Do not repeatedly retry an unchanged OOM configuration.

## Confirm and finish

Retrain finalists and the baseline from scratch at the intended final fidelity. Use matched data order and seed sets where possible; do not assume identical random seeds yield identical initial conditions across architectures. Prefer several seeds when affordable, and report the individual scores and variability. If only one run fits, say the apparent improvement is unconfirmed. Keep an untouched final split when available; if the project exposes only validation data, report that limitation rather than inventing a held-out result.

Report resource tradeoffs on the target hardware. Fixed-time wins demonstrate performance under that training budget, not superior asymptotic sample efficiency. A proxy model or short context win needs confirmation at the intended scale and context length. Do not claim a general NAS optimum.

Stop at the requested limit, interruption, exhausted feasible search space, or a blocker that prevents useful progress. On resume, reconcile running processes and saved artifacts before relaunching, verify evaluation fingerprints, and start a new comparison group if the contract changed. Persist state before context loss; conversational memory is not the experiment ledger.

In research mode, deliver the module inventory, hypotheses, controlled comparisons, ablations, and the proposed search space, including unresolved questions. In search mode, deliver the selected architecture and its revision, baseline comparison, uncertainty, constraints, experiment ledger, rejected hypotheses, and exact reproduction command. Explain any missing confirmation. Leave the winning changes reviewable in the user's repository. A report of no reliable improvement is a valid result.

## Agent behavior and model guidance

The user's instructions override this skill's defaults within applicable system and developer constraints. Avoid setup confirmation rituals for already authorized work. If this skill causes a pause, quote the applicable instruction and explain the concrete missing input. Keep progress updates focused on observations and the next decision. Run checks that can invalidate the architecture; stop repeating them once they pass unless the code or evidence changes.

Use the user's configured agent model. This skill does not require an API controller or a model switch. For long-running tool calls, user steering, or optional delegation, read [research-agent operation](references/agent-operation.md). It includes the applicable agent behavior directly, without external documentation dependencies.
