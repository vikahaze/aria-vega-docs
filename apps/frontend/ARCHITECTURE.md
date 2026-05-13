# Architecture — Frontend Control Panel

The `@lp-system/frontend` package is a React Next.js application designed to provide liquidity providers with a real-time monitor and control plane over their active on-chain positions and strategy orchestrators.

---

## 1. Core Framework & Styling

- **Next.js (v15+)**: Standardizing on the Next.js App Router for server-side structure and static route generation.
- **Client Component Routing**: The entire control panel is served as a client component (`"use client";`) on the root path `/`. This is because the dashboard depends heavily on browser-specific state, real-time client-side polling, and modular DOM subcomponents.
- **Tailwind CSS (v4)**: Global styles, keyframe animations (like CRT CRT scanlines and infinite text marquees), and a responsive wireframe grid are configured using Tailwind CSS.

---

## 2. API Communication & Synchronization

The frontend connects to the `market-maker` engine REST API using a configurable environment variable `NEXT_PUBLIC_API_URL` (which defaults to `http://localhost:8441` in development).

### Synchronization Loop

1. **Mount**: Upon mounting, the component initiates parallel fetches to the following Express endpoints:
   - `GET /health` — Check engine liveness and fetch the server-side timestamp epoch.
   - `GET /positions` — Fetch active on-chain CLMM positions.
   - `GET /assignments` — Fetch registered strategy assignments.
   - `GET /strategies` — Fetch introspected trading strategy definitions.
   - `GET /steps` — Fetch atomic stateless pipeline steps.
2. **Dynamic Polling**: An interval runs every **5 seconds (5000ms)** to silently refresh position, health, and assignment states on the client without UI stuttering.
3. **Data Enrichment**:
   - Strategy IDs are enriched with human-friendly names and risk indicators (Low, Medium, High).
   - Stateless step IDs are paired with descriptions describing their individual validations/computations.

---

## 3. Interactive Operations

The dashboard permits write-operations directly modifying the engine state:

- **Create Assignments**: When a user selects a strategy and clicks "Set Assignment", the frontend first requests to `DELETE /assignments/:id` for any existing assignment on that position (to avoid conflicts), followed by a `POST /assignments` request registering the new strategy and triggering its orchestrator.
- **Delete Assignments**: Deleting an assignment triggers `DELETE /assignments/:id`, which cleans up the persistence files and deregisters the orchestrator on the engine.
- **Evaluate Ad-Hoc**: For active strategy assignments, users can click "Evaluate Ad-Hoc" to run manual strategy ticks using `POST /strategies/:id/evaluate` with the position and pool details. Results are streamed in real time to the live CRT Events Log console on the screen.
