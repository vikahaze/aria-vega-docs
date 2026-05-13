# Executor Architecture

The `@lp-system/executor` package is a stateless execution layer responsible for transaction construction, simulation, signed-dispatching, on-chain finality confirmation, and low-level token balance polling.

---

## 1. Architectural Role and Boundaries

While some design notes historically grouped the stateful "rebalance lifecycle" under the executor, the production codebase enforces a strict separation of concerns between stateful coordination and stateless execution:

```
+--------------------------------------------------------+
|          Engine Layer (apps/engine/src/lifecycle.ts)   |
|  - Execution Monitor (processTasks)                    |
|  - Manages Task Store (saveTask, status transitions)   |
|  - Builds Synthetic Positions                          |
|  - Evaluates JIT Strategies                            |
+---------------------------+----------------------------+
                            |
           Applies Atomic   |  Calls Helpers
           Legs (Close/Open)|  (pollBalances)
                            v
+--------------------------------------------------------+
|       Executor Layer (packages/executor/src/)          |
|  - SolanaExecutor (Stateless Transaction Dispatcher)   |
|  - Simulated Compute Budget limits                     |
|  - Rebroadcasting "Spam Loops"                         |
|  - ATA balance checking                                |
+--------------------------------------------------------+
```

- **Execution Monitor (`processTasks`)**: Drives the multi-phase rebalance state machine, writes progress to `data/tasks.json`, synthesizes positions in memory, and performs strategy evaluations.
- **Transaction Executor (`SolanaExecutor`)**: Serves as a stateless worker. It receives a specific, authorized `Decision` (like an atomic `close` or `open`) from the monitor, prepares and signs the transactions, and broadcasts them to the network.

---

## 2. Stateless Execution Operations

The `SolanaExecutor` implements the `IExecutor` contract. Its core entry point is the `apply` method, which is called by the Execution Monitor to perform distinct atomic legs on the blockchain.

### A. The Close Leg (`action: 'close'`)

1. **On-Chain Position Lookup**: Queries the active on-chain positions via the provider to find the position's exact bin boundaries (`lowerBinId`, `upperBinId`).
2. **Transaction Building**: Requests the on-chain provider to build the liquid removal transaction payload with `shouldClaimAndClose: true`.
3. **Rent Reclamation**: Bundles instructions to close the position PDA and reclaim the locked SOL rent back to the user's wallet.
4. **Sequential Dispatch**: Signs and submits transaction chunks sequentially using the assured broadcast loop.

### B. The Open Leg (`action: 'open'`)

1. **Dynamic Gas/Rent Capping**: If opening a WSOL position, queries the wallet's SOL balance and dynamically caps the deposit amount to ensure a gas/rent buffer remains in the wallet.
2. **Keypair Generation**: Generates a fresh Solana Keypair to act as the signature authority for the new position PDA.
3. **Instructions Assembly**: Builds the on-chain instructions for adding concentrated liquidity within the target bin ranges.
4. **Dispatched execution**: Bundles the instructions into a signed transaction and broadcasts it with the position's unique keypair.

---

## 3. Core Technical Features & Guardrails

### A. Delivery Assurance ("Spam Loop")

Solana congestion can cause UDP packet loss. To ensure transaction delivery, the executor runs an active **rebroadcast loop**:

- Sends raw serialized transactions to the RPC network with `skipPreflight: true` to avoid local node bottlenecks.
- Actively polls the signature status every 2 seconds.
- Re-transmits the raw payload on each pass until `confirmed` finality is achieved or a 60-second hard execution timeout is reached.

### B. Settlement Verification Helper (`pollBalances`)

The Execution Monitor needs to wait until token balances have fully settled before initiating JIT strategy checks. The executor provides a public `pollBalances` helper method to achieve this:

- Queries the wallet's Associated Token Accounts (ATAs) for both Token X and Token Y (including wrapping/unwrapping native SOL balances).
- Continuously polls with a 2-second interval.
- Blocks and returns only when the current balance exceeds the pre-closure baseline, guaranteeing that subsequent strategy calculations work with actual liquid funds.

---

## 4. Architectural Debt & Legacy Code

> [!WARNING]
> **No Direct `close+open` Execution**
> In `solana-executor.ts`, the `else if (decision.action === 'close+open')` code path throws an error. This path is permanently guarded by a hard assertion. Any `close+open` decision reaching the executor directly indicates a programming error in the caller.
>
> \*In the production tick loop, the Execution Monitor decomposes the `'close+open'` task into an atomic `'close'` execution followed by a stateful `'awaiting_settlement'` phase and a `'pending_open'` execution.
