# Docs — Solana CLMM LP Strategy System

Start here. Every doc in this directory is listed below.

## Architecture

- [ARCHITECTURE.md](ARCHITECTURE.md) — high-level design, components, and core data flow
- [INTERFACES.md](INTERFACES.md) — core domain models, business logic boundaries, and structural contracts

## Decisions

- [decisions_position_lifecycle.md](decisions_position_lifecycle.md) — Design and state rules for the Position Lifecycle State Machine.

## Q&A

- [QA.md](QA.md) — answered architecture questions (auto-maintained by agents)

## Guides

<!-- How-tos, runbooks, onboarding -->

## Packages

- [@lp-system/core docs](../packages/core/docs/START_HERE.md) — Core Domain Model Documentation
- [@lp-system/config docs](../packages/config/docs/START_HERE.md) — Central configuration and dynamic workspace path resolution.
- [@lp-system/logger docs](../packages/logger/docs/START_HERE.md) — Multi-transport Structured Winston Logging Documentation
- [@lp-system/persistence docs](../packages/persistence/docs/START_HERE.md) — File-based JSON Persistence Documentation
- [@lp-system/providers docs](../packages/providers/docs/START_HERE.md) — Load balanced RPC & API Providers Documentation
- [@lp-system/steps docs](../packages/steps/docs/START_HERE.md) — Reusable stateless step components Documentation
- [@lp-system/strategy docs](../packages/strategy/docs/START_HERE.md) — Workflow composition and strategies Documentation
- [@lp-system/orchestration docs](../packages/orchestration/docs/START_HERE.md) — Position orchestration & safety gates Documentation
- [@lp-system/executor docs](../packages/executor/docs/START_HERE.md) — Transaction dispatchers & re-evaluation loops Documentation
- [@lp-system/engine docs](../apps/engine/docs/START_HERE.md) — Composition root & loop runner Documentation
- [@lp-system/frontend docs](../apps/frontend/docs/START_HERE.md) — Next.js Control Panel Frontend Interface Documentation
