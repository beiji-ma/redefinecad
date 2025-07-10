# RedefineCAD-05: A Query Language for CAD Metadata

> A practical DSL for exploring structured CAD data

---

## Overview

In this fifth chapter of RedefineCAD, we move beyond structural metadata extraction and enter the realm of **queryability**.

Having established how metadata is:

- Stored in a structured object graph
- Visualized through dump parsing

...we now ask: **Can this be queried like a database?**

This article introduces the foundational query language powering RedefineCAD’s inspection tools—built not on SQL, but on a purpose-built DSL designed for CAD semantics.

## Motivation

Modern design metadata isn't just a flat list of attributes. It’s a **deep, nested graph**:

- Assemblies reference subassemblies
- Views contain annotations, which reference model features
- Parameter values are scoped, versioned, and even conditional

We needed a **lightweight query system** that:

- Is easy to embed into local or CLI tools
- Respects structural depth and context
- Avoids the overhead of full SQL or JSONPath engines

That’s why we created a **custom DSL** (Domain-Specific Language), purpose-built for metadata graphs.

## DSL Highlights

The RedefineCAD DSL is purpose-built to handle complex, nested CAD metadata structures. Key design principles include:

- **Tree-aware traversal**: Native support for `children[*]`, `descendants[]`, and `parent` navigation.
- **Concise syntax**: Minimal, readable syntax that maps closely to the structure of the data.
- **Function-driven filters**: Supports built-in functions like `has_property(...)`, `is_null(...)`, and `count(...)`.
- **File-first philosophy**: Designed to work with local JSON exports, not remote servers or APIs.
- **Python-native engine**: Built entirely in Python for easy integration with CLI tools or scripting pipelines.

This design avoids the complexity of adapting general-purpose query languages and instead focuses on **real-world use cases in CAD metadata analysis**.

## Example Queries

Here’s a taste of the expressive power we aim for:

```dsl
where (revision == "A") and (part_number starts_with "P10")
```

```dsl
select name, part_number from children where definition contains "Gasket"
```

```dsl
count where has_property("IMAN_ID")
```

These queries operate on a unified internal model, typically loaded from a `bom.json` or `meta.json` output.

## Syntax Highlights

- Expressions combine comparisons and logical conditions:

```dsl
(revision == "A") and (definition contains "Cover")
```

- Navigation supports:
  - `children[*]`
  - `descendants[]`
  - `parent`

- Utility functions:
  - `has_property("...")`
  - `count`, `is_null`, etc.

## DSL Grammar Sketch

```
expression := comparison | logical_expr
comparison := field operator value
logical_expr := expression ("and" | "or" | "not") expression
field := identifier ( "." identifier )*
operator := "==" | "starts_with" | "contains" | "in"
```

We also support utility functions like `has_property(...)` and `count(...)`.

## Engine Architecture

The query engine is composed of:

1. **Tokenizer**: Breaks raw string into tokens
2. **Parser**: Builds AST from token stream
3. **Evaluator**: Applies AST to target object model

All of this is implemented in pure Python, with zero external dependencies.

## CLI Integration

The `catmeta-cli` tool now includes a `filter` command:

```bash
catmeta filter --input bom.json --where "(revision == 'A') and (definition contains 'Cover')"
```

This produces a filtered list of nodes matching the condition.

Output can be:

- Printed to stdout
- Exported as new JSON
- Used in pipelines

## Use Cases

- 🔍 Filter out parts with missing revisions
- 🧱 Group objects by type or definition
- 🧾 Generate human-readable reports
- 🔐 Enforce policy checks on export or commit

---

## Next up

🔵 RedefineCAD-06: From Expression to Script

