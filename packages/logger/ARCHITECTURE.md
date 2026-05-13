# Logger Package Architecture

The `@lp-system/logger` package provides a robust, multi-transport structured Winston logging facility for the Solana CLMM automation system monorepo.

## Design Highlights

1. **Separation of Concerns**: Kept outside of `@lp-system/core` to keep core types pure, dependency-free, and isomorphic.
2. **Dynamic Path Resolution**: Leverages `@lp-system/config` to resolve the absolute workspace path and correctly write files to `data/logs/` whether executing from root or individual apps/packages.
3. **Multi-Transport & Service Logs**: Emits to:
   - System standard output (console)
   - A global `all-logger.log`
   - A service-specific log file, e.g., `engine.log`
4. **Enhanced Argument Serialization**: Overrides Winston's standard log methods to support multiple arguments, deep-inspecting and pretty-printing objects, Errors, Maps, and Sets.

## Configuration

- `LOG_LEVEL`: Log level for Console transport (default: `'info'`)
- `LOG_FILE_LEVEL`: Log level for File transports (default: `'debug'`)

## Usage

```typescript
import { getLogger } from '@lp-system/logger';

const logger = getLogger('engine');

logger.info('Engine loaded successfully.');
logger.debug('Debug payload:', { a: 1, b: new Set([1, 2, 3]) });
logger.error('An error occurred:', new Error('Some error message'));
```
