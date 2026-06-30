---
name: STOS Standard Change
about: All changes to engines/, runtime/, workers/, adapters/, kv.ts, pipeline.ts
labels: architecture-review
---

# STOS — Pull Request

## 📋 Summary

**What does this change do?**
<!-- One sentence + link to issue / doc -->

**Implementation notes:**
<!-- Anything a reviewer must understand -->

**Related documents:**
- Blueprint: `STOS‑V2.3.3‑BLUEPRINT.md`
- Governing ADR: ADR‑001 — Freeze Layer 2 Runtime Architecture
- New ADR attached: ☐ Yes ☐ No → **#____**

**Change type:**
☐ Bug fix
☐ Performance
☐ Observability (logs / metrics / tracing)
☐ Internal refactor — preserves all 7 invariants
☐ New Layer 1 engine / feature
☐ ⚠️ Contract extension — **NEW ADR MANDATORY**
☐ Docs / tests / tooling only

---

## ✅ ARCHITECTURAL COMPLIANCE CHECKLIST
> **Derived 1:1 from STOS V2.3.3 Invariants + ADR‑001**
> **RULE:** EVERY line must be ✅ checked OR ☐ N/A + written reason.
> ❌ Any failing item AND no approved ADR → **BLOCKED FROM MERGE.**

### 🟢 LAYER SEPARATION
> Layer 1 creates plans. Layer 2 executes plans. Layer 2 never interprets intent.

- [ ] **L1→storage:** No file in `src/engines/` imports/uses Deno KV directly
- [ ] **L1→network:** No file in `src/engines/` calls Telegram API / external endpoints directly
- [ ] **Contract:** Every engine handler returns **only** `ExecutionPlan | Promise<ExecutionPlan>`
- [ ] **Workflow discipline:** `state/<entity>/<id>` never read/written directly — only via `plan.nextWorkflow`

### 📄 EXECUTIONPLAN CONTRACT
> Sole cross‑layer boundary · pure data only

- [ ] Contains **only** plain serialisable data — NO functions / closures / class instances / live handles
- [ ] Any new field is generic, documented in blueprint §I.3 — NOT a feature‑specific escape hatch
- [ ] Layer 2 validates **shape only**, never inspects *meaning*

### ⛓️ ATOMICITY · INVARIANTS 1, 2, 5
> Transactional Outbox · double‑gate idempotency

- [ ] Every `OutboxIntent` bundled in **same `ExecutionPlan`** as its `ModelUpdate`
- [ ] **ZERO external I/O** anywhere before Step 9 atomic commit succeeds
- [ ] Idempotency marker written **INSIDE** Step 9 tx — never separate before/after
- [ ] Atomic bundle unchanged: models · audit · aggregates · outbox · idempotency+workflow → **ONE write**

### ❄️ LAYER 2 PURITY · ADR‑001 FREEZE
> Frozen: Receiver · Pipeline · Atomic Persistence · Outbox Worker · Adapter · Router *INTERFACE*

- [ ] `pipeline.ts` — **zero** new `if`/`switch` on entity / state / feature
- [ ] `kv.ts` — generic only; NO domain helpers like `getTicket()`, `countPosts()`
- [ ] If `pipeline.ts` / `kv.ts` touched → either:
  ☐ fits permitted categories (bug/perf/observability/refactor) **OR**
  ☐ approved ADR‑### attached and linked
- [ ] Router *contract* unchanged — adding routes ≠ changing runtime

### 🔴 OCC RETRY BOUNDARY · EXPLICIT
> ONLY Steps 6–9 retry. 1–5 and 10–13 = **EXACTLY ONCE**

- [ ] Inside retry = pure reads + KV commit only — NO API / side effects / non‑idempotent ops
- [ ] Nothing from 1–5 recalculated inside loop; 10–13 never re‑entered
- [ ] Retry loop bounded, terminates cleanly

### 📐 NAMESPACE ISOLATION · INVARIANT 9
> `<entity>/<id>` ≠ `state/<entity>/<id>` — FOREVER

- [ ] New entity → **two separate prefixes**, never combined
- [ ] Workflow changes do **not** require migrating domain data

### ⌨️ UI CONTRACT · INVARIANT 6
> Buttons only · always navigable

- [ ] **NO new slash commands** — `/start` remains the only one
- [ ] Every new screen/menu has a **working back path**
- [ ] Limits honoured: text ≤ 4096 · `callback_data` ≤ 64 B · ≤ 8/row · ≤ 100 total

### 📢👥👤 PRIMITIVE SEPARATION · INVARIANT 8
> Never interchangeable

- [ ] Channel / Group / User → independent queues, limits, permissions
- [ ] Forums = Groups only · ledger/payments = Users only

### 👑 OWNERSHIP · INVARIANT 7
> Exactly ONE · always singular

- [ ] `OWNER_ID` remains single value — no arrays / multi‑owner / co‑owner
- [ ] Role set closed: **OWNER · MEMBER · GUEST** — nothing else

---

## 🚨 SEVERITY OF ANY VIOLATION FOUND
> See `docs/architecture/SEVERITY.md` for definitions
- P0 ☐ · P1 ☐ · P2 ☐ · None ✅

## 🧾 REVIEWER SIGN‑OFF
- [ ] All items ✅ or N/A + reason
- [ ] Invariants preserved OR approved ADR on file
- [ ] Merge permitted

**Reviewed by:** @________
