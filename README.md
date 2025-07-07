# RedefineCAD

> "We don't fix CAD. We redefine it."

RedefineCAD is an architectural rethink of how Computer-Aided Design systems should manage, govern, and evolve **design metadata**—not just geometry.

It is a layered specification, toolset, and philosophy aimed at:

- Making design metadata **durable**, **structured**, and **queryable**
- Supporting **versioning**, **traceability**, and **branching** without servers
- Enabling **automation** and **analysis** without requiring PDM/PLM platforms
- Empowering **individual designers** as well as **enterprises**

This is a multi-phase, long-term project that starts small—but is deeply ambitious.

---

## 🧠 Start Reading

This project is documented through a series of articles:

- 🟢 [RedefineCAD-01: Foundation and Motivation](https://github.com/beiji-ma/redefinecad/blob/main/articles/redefine_cad_01_foundation.md)
- 🟢 [RedefineCAD-02: Storage Layer and Object Identity](https://github.com/beiji-ma/redefinecad/blob/main/articles/redefine_cad_02_storage.md)
- 🟢 [RedefineCAD-03: Versioning Without Servers](https://github.com/beiji-ma/redefinecad/blob/main/articles/redefine_cad_03_versioning.md)
- 🔵 RedefineCAD-04: Deep Dive into CATIA Dump Protocol *(Coming Soon)*

🧩 Supplement:

- [Why RedefineCAD? (Philosophy and Ecosystem Primer)](https://github.com/beiji-ma/redefinecad/blob/main/articles/why_redefinecad.md)

To cite this work:

```
Beiji Ma, "RedefineCAD: Rethinking CAD from First Principles", 2025, https://github.com/beiji-ma/redefinecad
```

---

## 🧭 Key Concepts from the Architecture Diagrams

### 🔹 Snapshot
A **Snapshot** is a lightweight, point-in-time capture of the entire design metadata structure. It:
- Stores only object hashes (not full data)
- Enables fast diff, branching, rollback
- Is fully offline, no server involved

> Think of it as a Git commit—but designed for CAD metadata graphs, not files.

### 🔹 Object
The smallest unit of meaningful metadata—geometry, sketch, view, note, feature, etc.—each with a unique hash.

### 🔹 Tree
A hierarchical structure that aggregates objects. A snapshot points to one or more trees.

### 🔹 Diff
A compact, semantic difference between two trees or snapshots, enabling lightweight versioning and merging.

These concepts appear repeatedly in our visual and textual design. Familiarity will help readers understand the versioning strategy better.

---

## 📁 Repository Structure

| Folder               | Contents                                 |
| -------------------- | ---------------------------------------- |
| `articles/`          | Published articles and documentation     |
| `assets/`            | Diagrams and architectural illustrations |
| `examples/`          | Sample data files                        |
| `articles/NOTICE.md` | Author rights and legal disclosure       |

---

## 📜 License

This work is licensed under the **Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)**.

© 2025 Beiji Ma. All rights reserved.

Commercial use, republication, or derivative works are **not permitted** without written consent.

See [`articles/NOTICE.md`](https://github.com/beiji-ma/redefinecad/blob/main/articles/NOTICE.md) for full details.

---

## 📬 Contact & Attribution

For collaboration, feedback, or citation:

- GitHub: [beiji-ma](https://github.com/beiji-ma)
- Email: *Available upon request via GitHub profile*

> Built with 💡 and 🤖. Ideas welcome.

