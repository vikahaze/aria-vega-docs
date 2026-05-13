# Core Architecture

The `core` package comprises pure TypeScript data types and contract interfaces defining the boundaries between modules in the LP strategy system.

## Key Design Principles

1. **Zero Runtime Dependencies**: Avoids importing any external runtime packages or dependencies.
2. **Stateless Logic Contracts**: Interfaces like `IStep` or `IStrategy` accept state and configuration as arguments, emitting results deterministically.
