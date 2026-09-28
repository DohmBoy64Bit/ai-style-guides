# Professional Coding Style Guides for AI Agents

Drop-in style guides that make AI coding assistants write like an experienced maintainer of your codebase — not like a code generator.

## Why this exists

Left to their own defaults, AI coding agents tend to produce code that *looks* engineered but doesn't belong anywhere: generic names like `dataManager` and `resultProcessor`, comments that narrate every line, interfaces with a single implementation, `Manager`/`Service`/`Factory` layers stacked three deep, and defensive checks for states that can never occur.

Each guide in this repo is a curated set of rules that pushes back against those habits. The goal is always the same: **the smallest idiomatic production-quality change an experienced maintainer would reasonably write.**

## The guides

| Guide | Lines | Focus |
|---|---|---|
| [C#](csharp-style-guide.md) | 600 | Modern C#, records, LINQ restraint, nullable-reference-type-aware null handling |
| [C++](cpp-style-guide.md) | 5,943 | RAII over manual resource management, value semantics, modern C++ |
| [Rust](rust-style-guide.md) | 4,985 | Idiomatic ownership and borrowing, `Result`-based errors, simple over clever |
| [Go](go-style-guide.md) | 5,541 | Small consumer-defined interfaces, explicit errors, stdlib first, zero values |
| [Dart](dart-style-guide.md) | 5,932 | Strong static typing, correct null safety, immutable data, small functions |
| [Swift](swift-style-guide.md) | 4,596 | Value types first, protocol restraint, idiomatic Swift over Obj-C translation |
| [TypeScript](typescript-style-guide.md) | 2,823 | `type` vs `interface`, `unknown` over `any`, no type gymnastics, strict-mode discipline |
| [Python](python-style-guide.md) | 4,292 | Functions-first, no Java-style architecture, stdlib over frameworks, idiomatic typing |
| [Ruby](ruby-style-guide.md) | 5,793 | Simple objects, duck typing, guard clauses, idiomatic Ruby |
| [JavaScript](javascript-style-guide.md) | 3,724 | Plain objects over classes, no DI containers, boundary-only runtime validation |
| [HTML](html-style-guide.md) | 3,686 | Semantic elements, accessibility-first ARIA, native browser behavior |
| [CSS](css-style-guide.md) | 4,388 | Simple selectors, low specificity, design tokens, Flexbox/Grid, predictable cascade |

## Quick start

These are plain Markdown files — nothing to install or build. Copy the guide for your language into the instructions file your AI tool reads:

| Tool | Where to put it |
|---|---|
| Generic agents | `AGENTS.md` in the repo root |
| Claude Code | `CLAUDE.md` in the repo root |
| Cursor | `.cursor/rules/` directory |
| GitHub Copilot | `.github/copilot-instructions.md` |

Alternatively, paste a guide directly into a system prompt or custom-instructions field.

Working in more than one language? Combine guides, or keep separate instructions per project.

## What every guide enforces

- **Match the existing codebase.** Inspect nearby files first; consistency with the repository beats personal style preference.
- **Domain naming over AI-style names.** `parseWeapon()` and `assetPath`, never `processHandler()` or `genericService`.
- **Functions before classes.** Stateful objects only when state, lifecycle, or identity actually exists — and never architecture for one implementation.
- **Comments explain *why*, not *what*.** No narrating code, no tutorial prose, no decorative section banners.
- **Validate only at trust boundaries.** External input gets checked once; internal contracts are trusted afterward.
- **A before-finishing checklist.** Strip redundant comments, unused helpers, speculative TODOs, debug logging, and unrelated refactors — then run the project's formatter, linter, and tests.

## Make it yours

These guides are starting points, not law. Edit them to encode your team's own conventions — your preferred test style, logging library, or framework patterns — so the agent optimizes for *your* repository rather than a generic ideal.
