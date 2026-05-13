# Configuration Package Architecture

The `@lp-system/config` package houses central workspace environment loader, path resolution, and system constants.

## Key Exports

- `PROJECT_ROOT_PATH`: The absolute path to the monorepo workspace root, resolved dynamically.
- `DATA_DIR_PATH`: Absolute path to the central `./data` store folder.
- `LOCAL_DB_LOGS_PATH`: Absolute path to the `./data/logs` folder.

## Core Design

By loading `.env` and resolving directories dynamically, we avoid hardcoding or guessing paths in downstream services (like logging, persistence, or trading bots).

```typescript
import { PROJECT_ROOT_PATH, DATA_DIR_PATH } from '@lp-system/config';
```
