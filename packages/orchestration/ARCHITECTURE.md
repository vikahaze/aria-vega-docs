# Orchestration Architecture

The `@lp-system/orchestration` package manages active strategy runtimes, binds them to assignments, and routes execution recommendations through prioritized safety gates.

---

## 1. Key Modules

- **`StrategyOrchestrator`**: Adapts an `IStrategy` instance to a stateful runtime implementing `IOrchestrator`. Evaluates price ranges and triggers recommendations for a single live assignment.
- **`ExecutionGate`**: Evaluates recommendation vectors across active strategies, resolving conflicts and outputting prioritized, gated execution decisions.
- **`CircuitBreaker`**: Halts trading operations upon encountering high-severity RPC error rates or extreme on-chain price deviations.
- **`OrchestratorRegistry`**: Maintains an in-memory runtime registry of all active orchestrators, enabling rapid query indexing by position ID.
- **`OrchestratorFactory`**: Dynamically instantiates and registers appropriate orchestrators from persistent assignments.

---

## 2. Execution Locking & State Tracking

To prevent racing execution loops and duplicate tasks under the Stateful Task-Intent architecture:

### A. The `isExecuting` Flag

Each `StrategyOrchestrator` maintains a public `isExecuting` boolean property (initialized to `false`).

- **Locking**: When a rebalance decision is gated by the `ExecutionGate`, a `RebalanceTask` is written to disk and the orchestrator's `isExecuting` flag is set to `true`.
- **Tick Bypass**: In the Tick Loop, any position whose orchestrators are locked (`isExecuting === true`) bypasses standard ticking and strategy evaluation entirely. This prevents the generation of duplicate rebalance intents.
- **Unlocking**: Once the write-ahead task completes its execution lifecycle (re-opening the new position on-chain), the flag is toggled back to `false`, allowing standard evaluations to resume.
