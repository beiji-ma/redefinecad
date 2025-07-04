# RedefineCAD

![Status](https://img.shields.io/badge/status-in%20progress-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Vision](https://img.shields.io/badge/vision-cad%20redefined-orange)

**A Structural Rethinking of CAD Systems for the Future**

RedefineCAD is a visionary initiative to reimagine the foundations of CAD systems by borrowing battle-tested principles from database internals, Linux filesystems, and distributed version control systems.

This project proposes a new layered architecture for CAD data that separates logical structure from physical representation and introduces strong data governance, queryability, and recoverability—all without relying on conventional file systems.

## ✨ Key Concepts

* **CAD Autonomy**: CAD data as self-governing structures, not bound to files.
* **Data Integrity**: Inspired by redo/undo, checkpointing, and block-level recovery mechanisms.
* **Logical vs Physical Layers**: Decoupled management of metadata, geometry, and versioning.
* **Query Engine + DSL**: Treating geometry and metadata as queryable, composable data.
* **Version Control Redefined**: Integrated at the logical level, not imposed at storage.

## 🧭 Vision

> “To redefine the CAD ecosystem through architectural clarity, data autonomy, and engineering-grade recoverability.”

We aim to:

* Eliminate dependence on file-based storage
* Enable scalable DMU across entire assemblies
* Treat design data like structured, composable knowledge
* Inspire a new class of CAD governance and logic-driven protocols

## 📁 Repository Structure

```text
redefinecad/
├── articles/            # Published design articles and vision documents
│   ├── RedefineCAD-01-Foundation.md
│   ├── RedefineCAD-02-DataIdentity.md
│   └── ...
├── dsl-engine/          # DSL tokenizer, parser, and query executor
├── storage-core/        # Core logic for structure & persistence layer
├── examples/            # Sample datasets and testing BOMs
└── README.md
```

> 🔁 Formerly known as `docs/`; renamed to `articles/` for clarity.

## 🌍 Status

This repo contains selected publications, design notes, and DSL prototypes.
The actual implementation is in a private research repository (`redefinecad-drafts`).

## 📬 Contact

If you're interested in collaboration, partnership, or research exchange:

* GitHub: [beiji-ma](https://github.com/beiji-ma)
* Email: *available upon request*

## 📜 License

MIT License — For educational and non-commercial use.
Please contact the maintainer for commercial licensing discussions.

---

> 🪴 *“First we redefine the protocol, then we govern the future of design.”*
