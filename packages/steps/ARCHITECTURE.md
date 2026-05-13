# Steps Architecture

The `steps` package holds granular, reusable step modules implementing the `IStep` interface.

## Components

- **`InitializationCheckStep`**: Confirms that a position is alive and holds valid assets.
- **`TrailingRangeCheckStep`**: Monitors whether the CLMM pool's active bin has moved out of bounds.
- **`RangeCalculatorStep`**: Computes a new, balanced bin range surrounding the current active bin.
- **`AmountCalculatorStep`**: Identifies asset allocations needed to establish the new position.

## Design

- **Context Mutation**: Each step receives a `StepContext` object and resolves with a potentially modified `StepContext`.
- **Pure Functions**: Steps are async, but they avoid writing to any database or local storage; they are entirely stateless.
