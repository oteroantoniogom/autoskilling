# Research-agent operation

## Follow through within the task

The research agent proposes and implements networks. The trained network is the search subject. Keep the user's configured agent model; model selection and reasoning settings for the agent are separate from the neural architecture being searched.

Carry out authorized inspection, setup, trials, analysis, and finalization within the user's budget. Resolve routine implementation choices from local context. Ask only when a missing input materially determines the task or required resources. Do not ask whether to continue after each completed trial.

User corrections apply to pending work. Preserve completed results and record any change to the search contract. If a side question arrives during a run, answer it briefly and continue unless the user cancels or replaces the task. A request to stop should prevent new launches and safely stop owned work, preserving available evidence.

## Long-running tools

Use the host's existing process and tool interfaces. After launching training, save a trial ID, process or job identity, start time, command, log path, and deadline. Wait in bounded intervals that allow progress reports and steering. Report observations and remaining work without inventing interim scores.

If the host supports asynchronous tools, independent analysis may continue while training runs. Associate the completed tool result with its original call and trial. Do not compare a pending run as though it were finished. If asynchronous execution is unavailable, use ordinary process polling; it is sufficient for this workflow.

The application or process runner executes commands, handles termination, returns results, and accounts for resources. Instructions alone do not implement these mechanisms. Reuse the installed host rather than introducing a new API integration as a prerequisite.

## Optional delegation

Delegate only when available and authorized. Suitable independent tasks include analyzing a candidate family or reviewing a proposed architecture's validity. Give each worker a concrete scope and the same evaluation contract. Keep candidate edits in separate worktrees, and let the coordinating agent own the shared session ledger.

Serialize GPU benchmarks on a shared device. Concurrent training changes memory availability and timing. Separate devices can run independent trials only when their hardware and measurement conditions belong to compatible comparison groups. Count all workers against the total compute budget.

## Verification and reporting

Prioritize checks that can expose an invalid experiment: future-token leakage, broken masking, missing optimizer parameters, shared-weight duplication, unstable losses, and evaluator changes. Repeat checks when the code or assumptions change. Do not add repeated tests that merely restate configuration values.

Record concise hypotheses and observed evidence rather than speculative explanations presented as facts. Say when a trial is pending, failed, provisional, or confirmed. Report individual confirmation runs and resource tradeoffs. If the budget prevents confirmation, deliver the current evidence and the exact remaining work.

Persist the contract, queue, revisions, results, and process ownership before context loss. On resume, verify saved state against the local files and running jobs. The skill's bundled instructions and the user's project are sufficient; no external documentation lookup or companion skill is needed to operate the search.
