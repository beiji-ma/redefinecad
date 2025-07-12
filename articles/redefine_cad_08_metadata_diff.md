# RedefineCAD-08: Metadata Diff and Semantic Drift

> How do we track changes in design structure without relying on geometry? This chapter introduces structural diffing and the philosophy of semantic drift detection.

---

## 🎯 Purpose

Traditional CAD systems struggle with versioning beyond file diffs or geometric deltas. RedefineCAD proposes a diff model that operates over **semantic structure**:

- What **objects** were added, removed, or renamed?
- Which **relationships** or **constraints** changed?
- Are we witnessing an evolution or a contradiction (aka **semantic drift**) in the design intent?

This chapter offers a foundational method for:

- Capturing design evolution at the metadata level
- Supporting merge/conflict resolution without geometry
- Enabling rich audit trails and change queries

---

## 📘 Key Concepts

### 1. **Semantic Diff vs. Textual/Geometric Diff**

- Textual diff: line-based (not reliable for structured metadata)
- Geometric diff: computationally heavy, not always meaningful
- **Semantic diff**: structure-aware, type-aware, constraint-aware

### 2. **Node Identity and Relationship Tracking**

- Diff is computed between two metadata snapshots (tree structures)
- Object identity preserved via UUIDs or path hash
- Changes are typed: ADD / REMOVE / MODIFY / RENAME

### 3. **Drift Detection**

- Semantic Drift is detected when a conceptually stable part diverges in role, usage, or constraint
- Example: A mounting bracket used as structural support in v1, later as cosmetic housing
- Detected via role metadata, relationship delta, or schema mismatch

---

## 🧪 Example DSL (Diff Scenario)

```dsl
<DIFF>
  <REMOVED>
    <PART id="P1001">'Camera Housing'</PART>
  </REMOVED>
  <ADDED>
    <PART id="P2001">'Camera Shell'</PART>
  </ADDED>
  <REPLACED>
    <STRUCTURE>
      'P1001' -> 'P2001'
      Reason: 'Functionally equivalent, rebranded'
    </STRUCTURE>
  </REPLACED>
</DIFF>
```

---

## ⚙️ Design Goals

| Capability            | Description                          |
| --------------------- | ------------------------------------ |
| Structure-aware diff  | Traverses tree, not text             |
| Identity-preserving   | Compares object IDs, not just labels |
| Human-readable        | Output usable for review/audit       |
| Machine-parseable     | Suitable for pipelines and scripting |
| Drift detection hooks | Detect semantic breakage or shifts   |

---

## 🚦 Status

Prototype diff logic implemented in internal tools (e.g., `catmeta diff`)

- Comparison engine operates on JSON structure files
- CLI shows line-based diff, semantic diff summary
- Planned: web viewer, context-aware alerts

---

## 🧭 What's Next

The next chapter will explore how to query and traverse these diffs:

**RedefineCAD-09: Query Language over Structural DSL**

We'll learn to extract change history, impacted components, and detect regressions via semantic queries.

> "Diffs are not just logs — they are **knowledge** about how a system evolves."

