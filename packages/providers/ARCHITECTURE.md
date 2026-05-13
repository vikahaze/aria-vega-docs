# Providers Architecture

The `providers` package contains components that connect to external data feeds and blockchain nodes.

## Components

- **`MeteoraApiProvider`**: Connects to the Meteora DLMM Datapi endpoint to query open positions, pool layouts, and historical market metrics. Lives under the `meteora/` directory.
- **`SolanaRpcProvider`**: Abstract connection point using the standard `@solana/web3.js` `Connection` client. Serves as the base RPC provider class in `rpc/provider.base.ts`.
- **`HeliusRpcProvider`**: Specialized RPC connection subclass for Helius endpoints, extending `SolanaRpcProvider`. Lives in `rpc/provider.helius.ts`.
- **`RpcPool`**: Combines multiple RPC nodes into a round-robin load balancer. Handles automated rate-limiting failures (HTTP 429) via progressive Fibonacci backoffs. Lives in `rpc/rpc-pool.ts`.
