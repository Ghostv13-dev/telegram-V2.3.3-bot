# CHANGELOG — STOS v2.3.3

**Status: Frozen Reference Specification**

- **Architecture Freeze:** The Layer 2 runtime architecture is officially frozen under the governing ADR-001 rule: *Layer 1 creates plans. Layer 2 executes plans. Layer 2 never interprets business intent*.
- **ExecutionPlan Contract:** The ExecutionPlan has been introduced as the formal, exclusive boundary between Layer 1 (Internal Engines) and Layer 2 (Runtime Pipeline).
- **Strict OCC Retry Boundaries:** Optimistic Concurrency Control (OCC) retries are now explicitly bound to steps 6–9 (Snapshot read, Route resolution, State/Model materialization, and Atomic transaction). Steps 1–5 and 10–13 execute exactly once and are never re-entered on conflict.
- **Permanent Namespace Isolation:** Domain entities (e.g., users/) and their workflow states (e.g., state/users/) are isolated into separate Deno KV key prefixes.
- **Component Reclassification:** Customer Service state machine ownership and Channel Delivery targeting logic have been moved completely into Layer 1.
- **Double-Gate Idempotency:** Idempotency is now enforced twice: a pre-check at Step 4, and an atomic marker write inside the Step 9 commit.
