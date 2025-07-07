# RedefineCAD-03: Versioning Without Servers

> "Versioning is not a Git clone. It's a design decision."

In this third installment of the RedefineCAD series, we delve into how versioning can be elegantly solved without relying on centralized servers or heavyweight PLM systems.

---

## 🔁 What Does Versioning Mean for CAD?

Versioning in the CAD context is more than storing snapshots of geometry. It should include:

- Object identity and history
- Dependency tracking
- Divergence (branches) and convergence (merges)
- Metadata evolution, not just geometry

CAD systems often delegate versioning to external PDM tools—but that’s both **heavy** and **inflexible**.

We believe versioning should be:

- 🧱 **Built-in** at the protocol level
- 🗃️ **Lightweight**, supporting file-based workflows
- 🔍 **Transparent**, traceable without black-box databases

---

## 📐 Object Identity Revisited

In [RedefineCAD-02](https://github.com/beiji-ma/redefinecad/blob/main/articles/redefine_cad_02_storage.md), we introduced the idea of **content-addressable storage**, where objects are immutable and identified by their hash.

This naturally leads to the notion that **every version is a new object**, but linked via metadata.

### A versioned object might look like:

```json
{
  "id": "QmT123...",
  "type": "Sketch",
  "version": "A.2",
  "previous": "QmT122...",
  "timestamp": "2025-06-01T10:21:00Z",
  "author": "emma@acme.com",
  "comment": "Added constraint to left edge"
}
```

Here, `previous` forms a **chain**, similar to Git, but designed for **CAD semantics**.

---

## 🌿 Branches and Merges

What happens when multiple engineers work on the same assembly in parallel?

Our proposal:

- Each branch forms a **linked chain** of objects
- Merge is an explicit metadata object:

```json
{
  "id": "QmMerge001",
  "type": "Merge",
  "inputs": ["QmT200", "QmT198"],
  "result": "QmT201",
  "timestamp": "2025-06-03T15:11:00Z"
}
```

- No geometry is ever overwritten; instead, new versions are created
- **Conflict resolution** is handled by external tools or human intervention

This allows us to reconstruct full version history **without servers**.

---

## 🧪 Version Graph = DAG

All versioning metadata naturally forms a **Directed Acyclic Graph** (DAG):

- Each node = object version
- Each edge = derivation (previous version, merge input)
- Roots = original creations

### Benefits:

- Traceability
- Auditability
- Easy rollback
- Branching by default

This DAG can be **visualized** or queried, independent of file formats.

---

## 🔧 Implementation Hints

- Store version metadata in a `.meta/` folder alongside `.cad` files
- Use stable content hashes (SHA-256 or IPFS CID)
- Link objects using IDs, not filenames
- Embed `previous`, `author`, `timestamp`, `comment` fields in metadata
- Optionally compress deltas for geometry payloads (if storage is a concern)

This works **offline**, and integrates well with git-like or rsync-like tools.

---

## 🌀 Why Not Just Use Git?

While Git is powerful, it lacks CAD-specific semantics:

- No understanding of object relationships (assemblies, sketches, references)
- Binary diffs are opaque
- Merge conflicts are low-level and manual
- Lacks context like who modified which dimension, or why

Our format is optimized for **design reasoning**, not just file content.

---

## 📊 Versioning + Query = Power

Imagine asking:

- *"Show all revisions of part P123 where length > 100mm"*
- *"Which versions were made by Alice and later merged into production?"*

This is possible when:

- Version metadata is structured
- Geometry and metadata are decoupled
- Query engine can traverse the DAG

> This is where RedefineCAD truly shines: it makes CAD not just storable, but explorable.

---

## 🧭 Next Up: Deep Dive into CATIA Dump Protocol

In Part 4, we reverse-engineer a well-known proprietary CAD format—not for fun, but to **liberate** design data and inspire better protocols.

> Follow us as we dive deep into the binary trenches.

👉 [RedefineCAD-04: Deep Dive into CATIA Dump Protocol (Coming Soon)](https://github.com/beiji-ma/redefinecad/blob/main/articles/redefine_cad_04_dump.md)

