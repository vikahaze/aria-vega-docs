# Strategy Architecture

The `strategy` package implements high-level workflow runners that coordinate collections of `IStep` components to form cohesive trading rules.

## Core Elements

- **`Workflow`**: Runs an ordered list of `IStep` elements sequentially, maintaining the context state.
- **`TrailingUsdcStrategy`**: Implements the `IStrategy` interface. Leverages `Workflow` with a structured list of steps:
  `InitializationCheckStep` → `TrailingRangeCheckStep` → `RangeCalculatorStep` → `AmountCalculatorStep`.
