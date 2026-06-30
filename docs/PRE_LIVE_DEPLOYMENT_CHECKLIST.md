# Pre‑Live Deployment Checklist

This go/no‑go checklist must be cleared before any production deployment.

- **Signature Verification:** The Update Receiver and Router must reject incoming webhooks that lack a valid X-Telegram-Bot-Api-Secret-Token.
- **Navigation Boundaries:** All compiled menu matrices must include a functional back button.
- **KV Atomic Testing:** Parallel write attempts must trigger proper Optimistic Concurrency Control (OCC) retries without causing data corruption.
- **ExecutionPlan Conformance:** Every Layer 1 engine must emit only ExecutionPlan objects, with zero direct KV or Telegram API calls.
- **Outbox Atomicity:** Every OutboxIntent must be committed in the same transaction as its corresponding ModelUpdate.
- **Runtime Purity:** The pipeline.ts and kv.ts files must contain zero feature-specific branches.
