# RedefineCAD-06: From Expression to Script

## Introduction

At the heart of RedefineCAD is a belief that CAD metadata should be:

- **Describable**: Easily expressed and structured
- **Queryable**: Allowing humans and machines to extract meaning
- **Executable**: Driving downstream automation or validation

This document illustrates how a declarative expression of structure becomes an actionable script — forming a bridge between design semantics and engineering automation.

---

## 1. Starting with Structure: A DSL Example

Consider the following DSL snippet:

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

This expresses a minimal semantic model. But what does it *do*?

> ℹ️ The DSL syntax follows a lightweight, human-friendly format similar to JSON or TypeScript declaration blocks. It is not tied to any particular programming language, and focuses on type clarity and readability over formal grammar.

---

## 2. Parsing into Intermediate Representation

Once parsed, the model transforms into an intermediate tree structure:

```json
{
  "types": {
    "Part": {
      "attributes": ["PartNumber", "Revision"],
      "relationships": ["hasSubpart"]
    }
  },
  "relationships": {
    "hasSubpart": {
      "from": "Part",
      "to": "Part",
      "cardinality": "1..*"
    }
  }
}
```

This forms the basis of:

- **Code generation** (e.g., Python classes)
- **Validation scripts**
- **Query planning**

---

## 3. Generating Executable Code

From this structure, RedefineCAD can auto-generate validation logic:

```python
class Part:
    def __init__(self, part_number, revision):
        self.part_number = part_number
        self.revision = revision
        self.subparts = []

    def add_subpart(self, part):
        if not isinstance(part, Part):
            raise TypeError("Expected a Part instance")
        self.subparts.append(part)
```

Or generate a schema-aware checker:

```json
{
  "Part": {
    "required": ["PartNumber", "Revision"],
    "relationships": {
      "hasSubpart": {
        "type": "Part",
        "min": 1
      }
    }
  }
}
```

---

## 4. Toward Full Roundtrip

In the future, the goal is **semantic roundtrip**:

- Express → Parse → Validate → Execute → Reflect → Evolve

Every change in design intent should reflect in script behavior. This is the core philosophy behind RedefineCAD’s script engine.

> 🧠 In short: **semantics aren’t stored — they’re enacted**.

For instance, updating the DSL to change cardinality from `1..*` to `0..1` would immediately affect the behavior of the generated Python validator and the runtime relationship rules — no manual update needed.

---

## 5. What This Enables

- **Data Validators**: Enforce model constraints in real time
- **Metadata Compilers**: Translate DSL to JSON/YAML for pipelines
- **Query Optimizers**: Plan queries using structural hints
- **Change Checkers**: Detect semantic drift between versions

This is where RedefineCAD stands apart — not just a metadata linter, but a **semantic processor**.

> 🔍 Unlike traditional BOM validators that rely on brittle field-level rules, RedefineCAD builds validation and automation from declarative intent — making it resilient, evolvable, and transparent.

## 6. Recap

| Layer            | Description                                      |
| ---------------- | ------------------------------------------------ |
| DSL              | Declarative expression of structure              |
| IR (AST/JSON)    | Machine-readable semantic representation         |
| Scripts          | Generated checkers, validators, or code adapters |
| Automation Hooks | Integration points for pipelines & editors       |

Together, they enable not just documentation — but transformation.

---

## What’s Next

- RedefineCAD-07: Metadata Diff and Semantic Drift
- RedefineCAD-08: Query Language over Structural DSL

Stay tuned.

