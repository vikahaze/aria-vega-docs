# Engine Architecture

The `@lp-system/engine` application serves as the composition root, bootstrapping execution loops and exposing an HTTP control plane.

---

## 1. Core Processes

- **Composition Root**: Bootstraps adapters, RPC connections, dynamic persistence stores, orchestrator registries, and executors. Located in `main.ts`.
- **Agnostic Discovery Loop**: Synchronizes on-chain live positions with local JSON cache maps. Runs on engine startup and periodically to maintain live orchestrators. Integrates with the Task Store to ensure positions undergoing rebalancing are not pruned.
- **Stateful Tick Loop**: Processes in-flight `RebalanceTask` intents sequentially via the Execution Monitor, locks executing positions, evaluates standard active positions, and commits rebalance intents to disk.
- **REST Control Plane**: Launches an Express HTTP server allowing active monitoring, control, configuration, and manual position tracking.

---

## 2. HTTP Server REST Endpoints

The HTTP server loads dynamic routers and exposes the following concrete REST endpoints:

### A. Assignment Management

- **`GET /assignments`**: Retrieves all persistent strategy assignments.
- **`POST /assignments`**: Creates and persists a new assignment, then registers and initiates standard tick loop tracking on its orchestrator.
- **`DELETE /assignments/:id`**: Deletes the specified assignment from persistent storage and de-registers its orchestrator from the runtime.

### B. Strategy Ad-Hoc Control

- **`POST /strategies/:id/evaluate`**: Triggers an immediate, ad-hoc strategy evaluation against a target position, bypassing the standard Tick Loop interval.

### C. System Introspection & Status

- **`GET /positions`**: Lists all live, on-chain CLMM positions owned by the configured wallet address.
- **`GET /health`**: Standard liveness probe returning system status (`healthy`) and current epoch timestamp.
- **`GET /docs`**: Interactive Swagger OpenAPI UI documentation.
