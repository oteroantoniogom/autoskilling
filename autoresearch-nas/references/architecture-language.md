# Architecture representation and mutations

## Represent executable candidates

Use a small versioned JSON descriptor or the project's equivalent configuration. Do not invent a general programming language when a typed module graph is sufficient. The descriptor specifies construction; actual training still requires implemented primitives and a builder in the user's project.

Define a registry of allowed operators. Each registry entry declares input/output ports, tensor axes, parameter ranges, shape inference, state semantics, trainable parameter policy, and construction code. A tensor with axes `[batch, time, entity, channel]` is not interchangeable with `[batch, entity, time, channel]` simply because its dimensions match.

Represent a candidate with:

- A schema version, registry version, named external inputs, and named task outputs.
- Nodes with stable IDs, operator types, operator parameters, and explicit parameter-sharing groups.
- Directed port-to-port edges with any projection, reshape, concatenation, or reduction represented explicitly.
- State ports and delayed edges for recurrence, including initialization and reset rules.
- Input availability, masking, causality, and output constraints.

A minimal descriptor might look like this. Operator names refer to project-implemented registry entries; this is an illustrative format, not a bundled training implementation.

```json
{
  "schema_version": 1,
  "registry_version": "1",
  "inputs": {"x": {"axes": ["batch", "time", "channel"], "width": 16}},
  "nodes": [
    {"id": "encode", "op": "linear", "params": {"in_width": 16, "out_width": 32}},
    {"id": "mix", "op": "causal_conv1d", "params": {"width": 32, "kernel_size": 3}},
    {"id": "head", "op": "linear", "params": {"in_width": 32, "out_width": 1}}
  ],
  "edges": [
    ["input.x", "encode.x"],
    ["encode.y", "mix.x"],
    ["mix.y", "head.x"]
  ],
  "outputs": {"prediction": "head.y"},
  "constraints": {"causal": true}
}
```

Topologically order the same-step graph. Reject cycles unless implemented through explicit delayed state transitions. Reject unknown operators, incompatible ports, unavailable inputs, unbound outputs, illegal sharing, and unsupported device/dtype combinations before training. Count projection and state-storage costs. Run dynamic checks after static validation because shape correctness does not prove causality or numerical stability.

Serialize deterministically and hash the descriptor together with registry and builder revisions. Keep training recipe, seed, evaluation contract, and fidelity in the trial identity. Only reuse a prior measurement when all relevant conditions match. Graph isomorphism detection is optional; a simple hash does not detect every equivalent network.

## Define mutation operators

Every mutation returns a valid child or a structured rejection reason. Record parent ID, operator name, arguments, mutation RNG seed, descriptor diff, and any initialization or training changes.

| Mutation | Preconditions and resulting change |
| --- | --- |
| Insert or remove | Preserve external contracts; reconnect compatible ports and retain required outputs |
| Substitute | Replace an operator with one that accepts and produces the required axes, shapes, and state semantics |
| Widen or narrow | Propagate dimensions through consumers and sharing groups; include changed projection costs |
| Add or remove a path | Preserve causal reachability; define addition, concatenation, or gating and its shape rules |
| Split or merge modules | Define branch inputs and fusion; reject disconnected required information paths |
| Share or unshare weights | Require compatible parameter shapes and roles; preserve intentional aliasing in checkpoints |
| Add memory or recurrence | Introduce delayed state with explicit update, reset, and inference semantics |
| Freeze or unfreeze | Keep the graph but change learning behavior; label this as a trainability intervention |

Permit simplification as well as growth. Increasing complexity is not itself progress. For architecture-preserving weight inheritance, verify the preservation claim numerically; arbitrary insertion or widening does not automatically preserve the parent function.

Crossover is optional. Implement it only for subgraphs with compatible boundary contracts and explicit sharing/state ownership. Single-parent mutations are enough for an initial engine.

## Select a search policy

For a few expensive trials, use agent-guided proposals from recorded hypotheses. For a compact discrete space, random sampling provides a useful simple reference. Evolution is appropriate when valid local mutations are implemented and enough trials fit the budget.

For a basic evolutionary policy, initialize a small feasible population including the baseline. Select parents from a declared quality/resource rule, sample valid mutations with bounded rejection attempts, evaluate children at the same screening fidelity, and update the population while retaining elites and some structural diversity. Record population size, selection rule, mutation probabilities, replacement rule, and seed. Cap retries for invalid mutations rather than looping indefinitely in an infeasible space.

A candidate is Pareto-dominated if another feasible candidate is no worse on every declared objective and strictly better on at least one. Use only complete comparable metrics. Apply any complexity penalty or tie-break rule before observing results, and report the unpenalized metrics too.

Changing the grammar or search policy mid-run creates a new version. Preserve lineage and comparison evidence. Do not claim that an adaptively changed policy beat a fixed reference unless the reference received a comparable total budget.

## Validate the engine before scaling

Test deterministic descriptor reconstruction, mutation preconditions, state resets, serialization, and checkpoint reconstruction. Verify that the same descriptor and implementation build the same graph and parameter-sharing pattern. Instrument mutation outcomes to distinguish invalid proposals from poor but valid architectures.

Choose small synthetic tasks that discriminate between mechanisms in the proposed space:

- A local sequence target to test whether convolution or a short receptive field can suffice.
- Delayed recall with varying delay to test access to history and state reset behavior.
- A permutation-invariant set target to test pooling and entity-axis semantics.
- Intermittently observed inputs with explicit masks to test availability handling without future leakage.

Use separate synthetic training and evaluation seeds. Include a known adequate architecture and an intentionally restricted control whose missing mechanism matters for the chosen task. Ensure useful candidates are reachable through the allowed operators; verify reachability by enumeration for a tiny space or a constructed mutation path for a larger one. Run the search on a bounded case and check whether it improves on the restricted control, not whether it reproduces one exact architecture.

Charge synthetic validation to the agreed budget. If training validation cannot run, deliver structural checks and mark engine effectiveness untested. Synthetic success establishes limited search capability; performance on the target task requires separate evidence.
