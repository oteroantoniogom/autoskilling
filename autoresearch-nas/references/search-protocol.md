# Search design and trial records

## Choose a tractable space

Start with the model's working operators and installed kernels. Name both the mutation and the constraints that keep it valid. The following are candidate families, not a requirement to try every family.

| Family | Candidate mutations | Checks and costs |
| --- | --- | --- |
| Attention | MHA to GQA or MQA, local/global schedules, attention placement | Head divisibility, causal windows, positional encoding, cache layout, kernel support |
| Token mixing | Causal convolution, recurrent or state-space mixer, attention/mixer hybrids | No future leakage, state resets, receptive field, incremental inference when required |
| Feed-forward block | Gated MLP, expansion ratio, activation, low-rank factors | Hidden dimensions, initialization, actual parameter count and throughput |
| Connectivity | Sequential versus parallel branches, residual gates, skip paths, normalization placement | Residual shape alignment, gradient flow, stable depth scaling |
| Sharing | Tied embeddings, cross-layer weights, repeated blocks | Optimizer deduplication, checkpoint aliasing, unique parameters versus repeated compute |
| Macroarchitecture | Depth/width tradeoff, heterogeneous stages, block schedules | Matched resource comparisons, projection costs, input/output contract |
| Conditional computation | Sparse experts, routing, expert sharing | Total and active parameters, routing collapse, auxiliary loss accounting, dispatch overhead |
| Vision or other domains | Convolution kernels, dilation, separable blocks, multiscale paths | Output resolution, task-specific invariances, receptive fields, deployment operators |

Example hypothesis: replace the baseline's sequential attention and MLP residual updates with parallel branches that consume the same normalized input. Hold width, training recipe, and evaluation fixed. Test whether training throughput improves enough to improve validation quality at fixed training time. This changes the computation graph and has a directly measurable tradeoff.

Treat depth and width as architecture parameters, but do not let them displace operator and connectivity experiments when the user wants broad NAS. Hardware feasibility can narrow the space; record why a family was excluded.

Prefer direct training of discrete candidates for a small code-based search. Introduce a weight-sharing supernet or differentiable search only when the task and budget justify it. Shared-weight rankings are proxies and require standalone retraining. Include supernet training and search-controller costs in total search cost.

## Comparisons and promotion

Choose one primary objective and hard constraints first. For multiple objectives, use the Pareto set or a scalarization declared before seeing outcomes. Missing metrics cannot qualify a candidate as feasible or Pareto-optimal.

Use stages such as shape checks, short screening, and final-budget confirmation. Choose their actual durations from the user's budget; a 300-second trial is an example, not a universal optimum. Compare candidates within a fidelity. Promotion rules should allow some diverse candidates through to detect architectures with slower early learning.

Keep dataset, evaluation examples, tokenizer, loss aggregation, sequence length, precision policy, device type, and measurement protocol stable. If changing one is part of the requested research, version the contract and rerun a suitable baseline. Faster hardware or a smaller validation sample is not an architecture improvement.

Fresh initialization is the default. If exploring inherited weights or architecture morphisms, label these runs and compare against an equally eligible baseline. Final confirmation uses standalone training unless inherited training is itself the deployment objective.

Measure latency after a fixed warm-up with accelerator synchronization and stated batch/input sizes. Record compile latency separately. For autoregressive deployment, distinguish prefill from decoding and include cache memory. Measure actual peak device memory. Treat analytical FLOPs as estimates; update the estimator after changing operators, sharing, or sparse routing. Do not reuse a dense transformer estimator for a different graph.

## Persistent records

Use JSON with explicit units and null for unavailable values. The following fields define a practical record; adapt naming to an existing experiment tracker rather than creating a duplicate ledger.

- Identity: session ID, trial ID, parent ID, timestamps, code commit, patch or source digest, architecture descriptor and digest.
- Contract: evaluation fingerprint, comparison group, hardware/software versions, precision, dataset/split IDs, training recipe, seed, fidelity and budget basis.
- Intent: structural mutation, hypothesis, predicted tradeoff, reason for selection, implementation fixes, tuning confounds.
- Outcome: execution status, metric name/direction/value, steps, tokens or examples, training seconds, compile seconds, evaluation seconds, total seconds, unique/active parameters, peak memory, relevant latency and throughput, FLOPs estimate with method.
- Evidence: command, exit code, raw log path, model artifact path, validation checks, promotion/selection decision and reason.

Separate execution status such as `completed`, `oom`, `invalid`, `timeout`, or `interrupted` from selection such as `incumbent`, `promote`, `archive`, or `reject`. Keep null metrics for unfinished trials. A repaired implementation gets its own attempt ID and code revision.

`state.json` records the incumbent, Pareto trial IDs, pending queue, completed IDs, spent and remaining budgets, and running trial ownership information. Write state atomically and keep the trial ledger append-only. A resumed run must identify stale processes using job ownership and start time, not a PID alone. Preserve completed logs even if a branch moves. Git commits referenced by trials must remain reachable through retained refs or be preserved as patches.

If reusing `.auto` or another tracker, add these NAS fields to that session rather than assuming a specific external skill is installed.
