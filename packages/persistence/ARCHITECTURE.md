# Persistence Architecture

The `persistence` package implements file-system based implementations of the `IStore` and `IPositionStore` contracts using atomic JSON reading and writing.

## Adapters

- **`JsonFileStore`**: Adapts `IStore` to store assignments and execution histories. Supports optional wallet and environment namespacing to produce dynamic filenames like `{wallet_short}_{env}_assignments.json` and `{wallet_short}_{env}_executions.json` (falling back to `assignments.json` and `executions.json` if namespacing is not configured).
- **`JsonPositionStore`**: Adapts `IPositionStore` to keep track of known, currently open positions. Supports optional wallet and environment namespacing to produce dynamic filenames like `{wallet_short}_{env}_positions.json` (falling back to `known_positions.json` if namespacing is not configured).

## Safety & Concurrency Guidelines

1. **File-Level Asynchronous Mutex**: All read-modify-write and read/write file operations are serialized through `AsyncFileMutex` (`fileMutex` singleton). This prevents multiple processes or orchestrators from executing overlapping file accesses, entirely eliminating asynchronous "lost update" race conditions on shared database files.
2. **Atomic File Write**: Ensure files are written securely to prevent partial state corruption on sudden daemon failures.
3. **Directory Bootstrapping**: Automatically initializes the target database directories on startup if they do not exist.
4. **Instance Namespacing Isolation**: Multiple bot instances running concurrently with different wallets/environments should specify namespacing options in the constructors to avoid state collisions.
