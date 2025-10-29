# Integration Models and Replay-Capable Design (PLM Context)

When we talk about system integration, the conversation often centers around queues, events, acknowledgements, retries, Saga patterns, and orchestration. These approaches are well-known and widely used. But they are not the only integration models available.

Replayability and recoverability are often the real foundation of long-term reliability.

## 1) Version-Control-Based Integration (Local Repository Model)
- Shared business data is treated like a **source repository**.
- Each system maintains a **local working copy**.
- Incoming changes are applied as **versioned commits**.
- When propagating updates, **diffs** are computed and only the delta is sent.

**Benefits:**
- Replay and time-travel
- Audit and traceability
- Deterministic state reconstruction
- Conflict analysis and controlled rollout

The mental model is closer to **Git** than to messaging middleware.

---

## 2) Content-Addressed / Immutable Change Logs
- Each update references the **hash of the previous state**.
- Does *not* require blockchain networks or tokens.

**Provides:**
- Provenance / traceability
- Tamper detection
- Deterministic replay
- Multi-system state convergence

If the system state can be derived from the change log, then **any downstream system can be rebuilt or reconciled**.

---

## 3) Broadcast + Upsert + Replay (Authoritative Source Propagation)
- One system acts as the **source of truth**.
- Changes are **broadcast** as stable contracts.
- Consumers apply **idempotent upsert** using natural keys + version.
- If failure occurs, the same payload can be **replayed** safely.

This model aligns well with PLM lifecycle semantics:
- Item + Revision define identity
- ECO/CO define units of BOM structural change
- CAD metadata separated from file delivery
- Effectivity expressed explicitly
- Lifecycle states serve as synchronization checkpoints

---

## 4) Why "Replay" Matters More Than "Event Notification"
Many public explanations of event-driven architecture emphasize **publish/subscribe** and asynchronous messaging. However, in enterprise environments, the critical requirement is **recoverability**.

Event-driven becomes robust only when it supports:

| Capability | Often Mentioned Publicly | Required in Real Systems |
|-----------|--------------------------|--------------------------|
| Publish events | ✅ | ✅ |
| Subscribe to events | ✅ | ✅ |
| **Replay past updates** | ❌ | ✅ |
| **Idempotent state application** | ❌ | ✅ |
| **Out-of-order tolerance** | ❌ | ✅ |
| **Monotonic version/sequence guard** | ❌ | ✅ |

Notification alone is not enough.
Replay + deterministic convergence is what makes integration **repairable**.

---

## 5) Choosing the Appropriate Model
| Scenario | Suitable Model |
|---------|----------------|
| Propagating authoritative state | Broadcast + Upsert + Replay |
| Cross-domain workflow state coordination | Event-driven choreography / Saga |
| High auditability and reproducible state | Version-controlled or content-addressed |

The key is **operational reality**:
- Number of systems
- Release agility
- Ownership boundaries
- Maintenance capacity
- Traceability requirements

---

## 6) Core Principle
**Not everything must be event-driven or orchestrated.**
But **everything must be replayable**.

A dependable integration system is one that can be:
- Re-applied
- Reconstructed
- Recovered

Regardless of middleware or architectural style.

---

End of document.

