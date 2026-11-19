---
name: design-patterns
description: Add or revise this repository's code-only Markdown design-pattern entries and maintain shared language baselines in README.md and supporting notes in docs/.
---

# Design patterns collection

Read `README.md`, the relevant entry, and its `docs/<pattern>-notes.md` file first. Keep one root-level, lowercase kebab-case Markdown file per pattern, using `singleton.md` as the initial example. Update the README index to reflect only implementations actually present.

The collection covers TypeScript, Python, Rust, Ruby, C, C++, Mojo, Java, Swift, Kotlin, Go, Nim, Zig, Lua, Scala, Haskell, OCaml, F#, Clojure, and Elixir. Implement the languages requested for the current task; leave other languages for subsequent contributions.

## Versions

- Use the latest stable release available by November 13, 2026, the current cutoff within the requested November 2026 baseline. Advance the cutoff only when requested.
- Verify exact versions and release dates against official release notes or repositories. Record the version, verification date, and source link in README.md's language baseline section. Reuse those shared baselines across patterns rather than repeating them in pattern files. Exclude previews, betas, release candidates, and nightlies. State uncertainty when verification is unavailable.
- For C and C++, distinguish the published language standard from the compiler version and feature support. For Mojo, describe release status accurately if no stable release exists.
- Only TypeScript may include a legacy alternative: a modern example plus ES5-compatible output/style. ES5 is a JavaScript target, not a TypeScript version. Verify compiler target support and document a separate legacy compiler if needed.

## Entries

Pattern files contain only a pattern title, language/variant headings, and language-tagged fenced code examples. Do not add baseline paragraphs, source links, explanations, run instructions, validation tables, or tradeoff sections to those files. Keep shared language baselines in README.md. Put pattern-specific explanations, run instructions, limitations, tradeoffs, and validation records in `docs/<pattern>-notes.md`. Link the code and notes in the README pattern index.

Use idiomatic standard-library features and concise inline comments for non-obvious behavior. Include runnable usage demonstrating the pattern's observable invariant, such as object identity for Singleton. Preserve the pattern’s intent using each language’s idioms. Do not force class-based mechanics onto functional or module-oriented languages. For Singleton, distinguish object identity, a shared immutable value, a module service, and a managed process owning state; specify the scope and lifecycle of each guarantee. Explain intentional API or behavior changes in the supporting notes. Implementations need not use identical mechanics across languages.

In the pattern's supporting notes, explain when blocks must run separately and document material limitations involving concurrency, initialization, ownership, mutation, subclassing, or process boundaries where relevant. Keep run instructions, tradeoffs, and validation results there too. Maintain `docs/language-guide.md` when adding languages, including what each adds and when to use it.

## Validation

Compile or run changed examples with an appropriate available toolchain and check their observable behavior. For formatting-only moves, verify that code blocks are unchanged instead of rerunning them. Keep extracted snippets and generated output outside the repository. Record what was actually checked and any missing toolchains in the pattern's supporting notes. Check Markdown fences, relative links, the README index, and that pattern files contain no prose outside headings and code fences.

Maintain the README contents list when its sections change. Keep supporting documentation linked from the README and update its contents list when present.
