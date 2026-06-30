# STOS — Architecture Violation Severity Guide
**Version:** 1.0 · **Effective:** 2026‑07‑01
**Source:** STOS V2.3.3‑BLUEPRINT.md §I.5 + ADR‑001

> Used in every PR review, incident triage and refactoring prioritisation.
> **Cardinal rule:** If it breaks one of the 7 frozen invariants, it is at least P1.

---

## 🔴 P0 — ARCHITECTURE BREACH · IMMEDIATE HALT
**Definition:** Breaks a fundamental correctness guarantee, invalidates the runtime contract, or makes ADR‑001 no longer hold. **Systemic risk.** Data loss, duplicate delivery, lost state or silent corruption *will* or *can already* happen. **CANNOT MERGE. CANNOT STAY IN PROD.**

**Required action:** Revert / block / fix before anything else. No exceptions.

**Examples:**
- External I/O / Telegram send **before** Step 9 commit
- `ModelUpdate` and `OutboxIntent` written in **two separate transactions**
- Idempotency marker written **outside** the atomic commit → race reopened
- OCC retry now includes Steps 1–5 or 10–13
- Layer 2 now inspects business meaning (`if (ticket.status === …)`) 
- `ExecutionPlan` now carries functions / closures / live handles
- Direct KV write or direct Bot API call from inside `src/engines/`
- Introduces multi‑owner / breaks singular `OWNER_ID`

## 🟠 P1 — CONTRACT BREAK · MAJOR DEBT
**Definition:** Violates an architectural invariant, design rule or frozen boundary, but does not immediately break correctness guarantees. **Erodes the architecture over time**, increases blast radius, makes future changes expensive/risky. **Merge blocked until resolved or ADR approved.**

**Required action:** Fix in this PR OR submit + get approval for a numbered ADR explaining why the invariant must change.

**Examples:**
- New feature‑specific branch added inside `pipeline.ts` / `kv.ts` — no ADR
- New domain entity stores data + FSM in **one single record** → no `state/*` split
- New slash command registered beyond `/start`
- Menu / screen added with **no back navigation**
- Channel/Group/User share a queue / rate limit / permission path silently
- New field added to `ExecutionPlan` as a one‑off escape hatch, not generic
- Router registration interface changed

## 🟡 P2 — CONVENTION / DESIGN DEBT
**Definition:** Does **not** break any invariant or contract, but deviates from patterns, naming, structure or layering intent. Makes the system harder to read, test or onboard to. **Does not block merge**, but must be tracked.

**Required action:** Fix in‑flight OR create a tracked tech‑debt issue with target milestone.

**Examples:**
- Naming doesn’t follow KV layout convention
- Helper could be generic but is implemented inside one engine
- Observability structured differently across modules
- Redundant reads, no functional impact
- Test organisation inconsistent with module boundaries

## 🟢 P3 — DOCS / STYLE / COSMETIC
**Definition:** Zero runtime or structural impact.

---

## 🧭 SEVEN FROZEN INVARIANTS — QUICK REFERENCE
*Anything altering these = **at least P1**, usually **P0***
1. Layer 2 never interprets business intent
2. `ExecutionPlan` = **only** Layer 1 ↔ Layer 2 contract
3. Zero external side effects before commit
4. Domain state + Outbox always atomic
5. OCC retries = **Steps 6–9 ONLY**
6. Idempotency marker written **inside** the transaction
7. `<entity>/<id>` vs `state/<entity>/<id>` permanently isolated
