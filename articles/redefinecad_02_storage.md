# RedefineCAD-02-Storage: Immutable by Design

In the previous article, we laid the groundwork for RedefineCAD as a data-first rethink of the CAD stack. In this chapter, we descend into the bedrock: the **Storage Layer**.

## Why Storage Comes First

Before we can talk about versioning, collaboration, or metadata queries, we must decide: **What is a "design" and how is it stored?**

Most CAD systems treat storage as a secondary concern. Files are zipped binaries. Metadata is scattered in sidecars or remote databases. Revisions are saved with timestamps or opaque internal IDs.

RedefineCAD turns this inside out. Storage is not just persistence—it is **meaning**.

## Principles of the Storage Layer

1. **Immutability is the default**\
   All stored objects—geometry, metadata, relations—are immutable. Once created, they never change.

2. **Content-addressability**\
   Objects are identified by their hash (e.g. SHA-256), ensuring deduplication, traceability, and reproducibility.

3. **Structured as DAGs**\
   Designs are not saved as files but as **Directed Acyclic Graphs**, with nodes representing atoms (e.g. sketches, features, parameters) and edges encoding relationships.

4. **Self-contained & portable**\
   A design archive is a portable folder or blob that contains everything it needs—like a `.git` repo or an OCI container.

5. **No reliance on centralized state**\
   All logic operates without requiring a master database or coordination server.

## A Simple Example

Suppose a part contains:

- A sketch (hash `sk123`)
- An extrusion based on that sketch (hash `ex456`)
- Material metadata (hash `mt789`)

These are linked into a root node (hash `rt001`):

```
rt001
├── ex456
│   └── sk123
└── mt789
```

Each object lives in a `store/` folder as:

```
store/
  sk123.json
  ex456.json
  mt789.json
  rt001.json
```

A single file, `manifest.json`, may describe the entry point, type system, schema versions, and basic metadata.

💡 *Note: In this illustration, we use **``** folder to explain the object model. In practice, these objects are not tied to any specific file format or filesystem layout. They may reside in a Git packfile, a content-addressable database, or even an object storage like S3 or IPFS. The JSON representation is used here purely for conceptual clarity.*

## Benefits

- 🔒 **Auditable**: Every change is recorded as a new immutable object.
- 🔁 **Reproducible**: Hashes ensure exact dependency trees.
- 🌐 **Syncable**: Any object can be transferred across sites or peers.
- 🧱 **Composable**: Objects can be reused, forked, or embedded.
- 💾 **Storable anywhere**: Git, S3, IPFS, even email.

## Comparison with Traditional CAD

| Feature           | Traditional CAD   | RedefineCAD           |
| ----------------- | ----------------- | --------------------- |
| Storage unit      | Binary file       | Structured DAG        |
| Revision tracking | Manual save       | Immutable lineage     |
| Deduplication     | Manual or none    | Hash-based            |
| Data portability  | Low               | High (self-contained) |
| Queryability      | External database | Built-in graph        |

## Beyond Geometry

While geometry is often the star, RedefineCAD treats **metadata as first-class**. This means:

- Units, tolerances, materials
- Change histories, annotations
- Links to simulations, documents, parts

...are all stored in the same immutable, queryable store.

## Storage as the Foundation of Semantics

This storage model is more than a persistence layer. It is the **semantic bedrock** on which all higher-level behavior—versioning, querying, simulation, DSL interpretation—is built.\
Every node in the DAG is a **semantic atom**, and every link represents an intention or dependency.\
From this perspective, version histories are simply transformations on subgraphs. Queries become graph traversals. DSLs become declarative mappings onto DAG segments.

This is not just an implementation trick. It is an assertion that:

> 💡 **There is no meaning without structure. And in RedefineCAD, that structure is the graph.**

## Undo/Redo as Graph Navigation

Traditional CAD applications implement undo/redo by maintaining a stack of editing commands or snapshots. RedefineCAD approaches this differently—leveraging its immutable DAG model.

Every design edit results in a new root node in the DAG, referencing updated components while reusing unchanged ones. Undo simply means **moving the current HEAD pointer to a previous root node**. Redo moves it forward again.

This is made possible because all previous states persist as immutable objects. The system only needs to track the lineage of root nodes:

```
HEAD → rt002 → rt001 → rt000
```

Reversing or reapplying changes becomes graph traversal, not state mutation.

In this way, **Undo/Redo becomes a natural part of the storage model**, rather than an auxiliary feature. It also has resilience benefits:

- A crash or disk failure won't corrupt your state.
- Recovering from partial saves or reboots is as simple as switching HEAD.

This is especially powerful in distributed scenarios or environments with poor reliability: **data loss is not rollback—just resync**.

## Next Up

The next article in the series will focus on **Versioning Without Servers**—how RedefineCAD enables distributed collaboration and traceable history without requiring centralized databases or file servers.

---

**Beiji Ma**\
July 2025

