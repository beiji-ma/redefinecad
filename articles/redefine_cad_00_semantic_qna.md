# RedefineCAD-06: From Expression to Script

> Designing an extensible language for navigating complex CAD graphs

---

## Why Stop at Expressions?

In RedefineCAD-05, we introduced a lightweight query language tailored for CAD metadata. It enables developers and engineers to filter and navigate object trees using expressive, SQL-like conditions.

But expressions are just the beginning.

In real-world workflows, engineers often need to:

- Perform multiple queries sequentially
- Assign intermediate results
- Use control flow (e.g., for/if)
- Define reusable functions or macros

In short, they need a **scripting layer**.

## Inspiration and Constraints

Rather than invent a complex language from scratch, we aim for:

- 🧩 **Minimal syntax**: Readable and embeddable in CLI/JSON
- 🧠 **Composable constructs**: Expression → Statement → Script
- 🛠️ **CAD-aware primitives**: Iterate over children, match revisions, invoke exports
- 🔒 **Safe sandboxing**: Prevent arbitrary system access

We take inspiration from:

- **MQL (Matrix Query Language)** for object expansion
- **TCL-like macro composition** for repeatable operations
- **Python subset** for conditionals and iteration

## Core Design Elements

### 1. Contextual Execution

Unlike SQL or GraphQL, the DSL operates on **loaded object trees** in memory. Scripts execute in a context with predefined variables:

```dsl
set roots = load("bom.json")

for part in roots.descendants[] {
  if (part.revision == "A") {
    export part
  }
}
```

### 2. Scripting Primitives

We support these statement forms:

```dsl
set <var> = <expression>
for <var> in <collection> { ... }
if (<condition>) { ... } else { ... }
export <object>
print <expression>
```

This enables automation scenarios like:

- Batch filtering and export
- Dry-run enforcement rules
- Post-processing and cleanup

### 3. Extensible Commands

We allow DSL-level commands to be extended from CLI:

```bash
catmeta run --script myscript.dsl --input bom.json
```

Or inline:

```bash
echo 'set x = load("bom.json")' | catmeta run --stdin
```

Custom commands (`export`, `print`, `tag`, `update`) are pluggable.

## Roadmap Preview

| Milestone | Feature                       | Status          |
| --------- | ----------------------------- | --------------- |
| v0.1      | Expression parser + evaluator | ✅ Done          |
| v0.2      | CLI filter support            | ✅ Done          |
| v0.3      | Script block + control flow   | 🏗️ In Progress |
| v0.4      | Plugin interface for commands | 🔜 Planned      |
| v0.5      | Sandboxed execution + tests   | 🔜 Planned      |

## What It Enables

- Automated audits before file delivery
- Expressive filtering without coding Python
- Modular reuse of validation logic
- Declarative definition of export policies

## Closing Thought

The future of CAD isn’t just visualization—it’s **programmable inspection**.

Let’s make metadata first-class.

---

## Next up

🔵 RedefineCAD-07: Building a Visual Inspector

