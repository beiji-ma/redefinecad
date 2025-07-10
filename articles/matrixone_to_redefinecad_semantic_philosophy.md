# Semantic Modeling in Practice: From MatrixOne to RedefineCAD

## Introduction

Semantic modeling is often discussed in the abstract, dressed in the language of OWL, RDF, and knowledge graphs. But for those of us who’ve built enterprise systems, managed product data, or configured PLM platforms, semantic modeling isn’t just a theory — it’s a way of thinking, building, and maintaining clarity in complexity.

We follow the philosophy of semantic modeling — not as an academic artifact, but as a working method for building scalable, understandable systems. **In our view, practical engineering adoption is the ultimate test of truth.**

Nowhere is this more evident than in the transition from **MatrixOne**, the semantic backbone of ENOVIA, to **RedefineCAD**, a new generation tooling framework designed to semantically express CAD structures and metadata outside their native silos.

This article traces how the **semantic modeling philosophy** — as practiced in MatrixOne — is being reimagined and extended through RedefineCAD.

---

## Legacy and Intention: A Personal Reflection

The ideas expressed in this article are not theoretical abstractions — they come from lived experience.

For the author, MatrixOne was not just a tool, but a medium for expressing complex enterprise semantics with elegance and precision. Having worked on PLM systems where *type definitions* were more expressive than schemas, and *lifecycles* were real governing models rather than code paths, this was the seed of a lifelong understanding.

What RedefineCAD carries forward is not just a toolchain, but a spirit: that of **engineering-centric semantic clarity**.

In a world awash with low-code platforms, fragile configurations, and accidental complexity, the rediscovery of semantic modeling as a first-class engineering asset is not nostalgia — it’s necessity.

To carry forward this philosophy is not merely a technical act, but an act of preservation. And perhaps, of justice.

---

## MatrixOne: Engineering Semantics Before It Was Called That

MatrixOne introduced the world to a different kind of modeling:

- Types defined objects, not just tables
- Policies governed action, not just roles
- Relationships had direction, cardinality, and business meaning
- Lifecycles were enforced as part of object behavior, not code logic

This wasn't just configuration — it was **semantic modeling in action**, long before the term became fashionable.

Even today, ENOVIA’s type trees, policy editors, and lifecycle graphs are among the most structured expressions of **enterprise-level ontologies**, though few call them that.

---

## RedefineCAD: Bringing Semantics Back to CAD

While ENOVIA governs product knowledge and process, CAD files remain trapped in opaque geometry and fragile links. RedefineCAD takes a bold step:

- **Decouples** the geometry from metadata
- Introduces **queryable structures** (Parts, Views, References) in open formats
- Emphasizes **explicit relationships**, not inferred links
- Exports structure and metadata in forms that are **describable and machine-interpretable**

This is not just a tooling convenience — it is **semantic modeling for the CAD domain**.

RedefineCAD follows the same lineage:

- From hardcoded links → to declarative relationships
- From implicit behavior → to explicitly governed structure
- From inaccessible attributes → to queryable, reusable metadata

> For example, instead of assuming a CATDrawing implicitly contains its Views, RedefineCAD makes these relationships explicit — `Sheet → View → Geometry → Feature`, each queryable and annotated.

---

## Shared Principles Across Two Generations

| Principle              | MatrixOne                         | RedefineCAD                             |
| ---------------------- | --------------------------------- | --------------------------------------- |
| Type System            | Type + Attribute                  | Object class + Key-Value model          |
| Relationship Semantics | Named, directional, typed         | Explicit parent-child + reference graph |
| Lifecycle & States     | Modeled transitions, Policy rules | Planned: model state-based workflows    |
| Queryability           | MQL (Matrix Query Language)       | Planned DSL query interface             |
| System Governance      | Metadata-driven control           | Schema-based CLI + stat validation      |

---

## From Theory to Practice: Why It Matters

This is not an academic convergence — it’s an engineering necessity.

Semantic modeling is not about OWL. It’s about:

- **Describing what things are**
- **Understanding how they relate**
- **Controlling how they behave**
- **Enabling others (and machines) to reason across them**

MatrixOne achieved this for PLM. RedefineCAD is extending it to CAD.

Together, they close the loop:

- One governs the **semantic structure of product information**
- The other reveals the **semantic structure embedded in design tools**

---

## Looking Forward: A Vision of Semantic Continuity

Just as MatrixOne liberated product data from rigid schema, RedefineCAD aspires to liberate CAD metadata — making it queryable, explainable, and portable across platforms and time.

This is not just CAD tooling — it’s **semantic infrastructure** for the next generation of design automation, versioning, and integration.

> *Structure is meaning. Semantics are infrastructure.*

RedefineCAD doesn’t reinvent the philosophy — it inherits it. And just like MatrixOne made semantics usable in the PLM world, RedefineCAD is doing the same for CAD.

---

## Additional Notes: Grounding Semantics in Practice

### ✳ Example: A Mini Semantic Model in RedefineCAD DSL

```dsl
ObjectType <Part> {
  attributes: ["PartNumber", "Revision"]
  relationships: ["hasSubpart"]
}

Relationship <hasSubpart> {
  from: <Part>
  to: <Part>
  cardinality: "1..*"
}
```

This simple structure illustrates RedefineCAD’s commitment to:

- Semantic expressiveness
- Declarative modeling
- Engineering practicality

### ✳ DSL Design Philosophy

> In RedefineCAD, a lightweight DSL is used to express type hierarchies, relationships, and constraints — enabling versioned, human-readable semantic definitions across evolving domains.

### ✳ Engineering, Not Ontology Compliance

Our focus is not OWL compliance or triple-store fidelity. Instead:

- We prioritize **interpretable**, **executable** models
- Favor **schema-aware CLIs** over inference engines
- Believe **engineering adoption is the measure of semantic success**

This is not a rebellion against academia — it's a reflection of what has worked in industry. MatrixOne proved it. RedefineCAD builds on it.

---

## What’s Next

- [MatrixOne and the Rise of Practical Semantic Modeling](https://github.com/beiji-ma/redefinecad/blob/main/articles/matrixone_semantic_modeling.md)
- Upcoming: **Designing DSLs for CAD Metadata — A Semantic Perspective**
- Join the discussion or contribute on [GitHub](https://github.com/beiji-ma/redefinecad)

Let’s continue this practical journey — one layer of meaning at a time.

