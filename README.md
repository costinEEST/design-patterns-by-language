# Design patterns by language

A collection of design patterns with idiomatic examples in TypeScript, Python, Rust, Ruby, C, C++, Mojo, Java, Swift, Kotlin, Go, Nim, Zig, and Lua.

## Contents

- [Patterns](#patterns)
- [Language baselines](#language-baselines)
- [TypeScript compatibility](#typescript-compatibility)
- [Adding a pattern](#adding-a-pattern)

## Patterns

| Pattern | Code | Supporting documentation |
| --- | --- | --- |
| Singleton | [Examples in all fourteen languages](singleton.md) | [Run instructions, implementation notes, validation, and tradeoffs](docs/singleton-notes.md) |

Each pattern file contains only a title, language/variant headings, and code examples with usage and helpful inline comments. Run blocks independently.

## Language baselines

Use the latest stable release available by **November 13, 2026**, the repository's cutoff within November 2026. The versions below were verified on that date against the linked official sources. Exclude previews, betas, release candidates, and nightlies; do not assume later November releases are available. C and C++ baselines refer to published standards, independently of compiler versions; C++26 is not used as a published standard.

| Language | Baseline | Official source |
| --- | --- | --- |
| TypeScript | 7.0.2 (ES2022 or newer target) | [official release](https://github.com/microsoft/typescript-go/releases/tag/typescript%2Fv7.0.2) |
| Python | 3.15.2 | [official version history](https://www.python.org/doc/versions/) |
| Rust | 1.100.0, edition 2024 | [official release announcements](https://blog.rust-lang.org/releases/) |
| Ruby | 4.0.9 | [official release list](https://www.ruby-lang.org/en/downloads/releases/) |
| C | C23, ISO/IEC 9899:2024 | [published ISO standard](https://www.iso.org/standard/82075.html) |
| C++ | C++23, ISO/IEC 14882:2024 | [published ISO standard](https://www.iso.org/standard/83626.html) |
| Mojo | 1.2.0 | [official stable releases](https://mojolang.org/releases/) |
| Java | Java SE 27 / Oracle JDK 27.0.1.0.3 | [official consolidated release notes](https://www.oracle.com/java/technologies/javase/27all-relnotes.html) |
| Swift | 6.4.1 | [official toolchain list](https://www.swift.org/install/macos/) |
| Kotlin | 2.4.21 (JVM) | [official release](https://github.com/JetBrains/kotlin/releases/tag/v2.4.21) |
| Go | 1.27.4 | [official release history](https://go.dev/doc/devel/release) |
| Nim | 2.4.2 | [official release announcement on the Nim homepage](https://nim-lang.org/#recent-articles) |
| Zig | 0.17.1 | [official stable downloads](https://ziglang.org/download/) |
| Lua | 5.5.1 | [official version history](https://www.lua.org/versions.html) |

These shared baselines apply to every pattern. Actual local toolchain versions and execution results are recorded in each pattern's supporting documentation.

## TypeScript compatibility

Only TypeScript includes a legacy alternative: modern TypeScript and ES5-compatible style. ES5 is a JavaScript output target, not a second TypeScript release. Use ES2022 or newer for the modern Singleton example.

The [ES5-compatible Singleton example](singleton.md#typescript--es5-compatible-style) uses a closure, function constructor, and prototype method. Compile away its type annotations and assertions with a TypeScript compiler that supports `--target ES5` before running it as JavaScript. Compiler support for legacy targets must be checked separately from the modern language baseline. See the [target documentation](https://www.typescriptlang.org/tsconfig/target.html).

## Adding a pattern

Create a root-level Markdown file named after the pattern in lowercase kebab-case, following [singleton.md](singleton.md). Keep that file limited to labeled code examples and put explanations, run instructions, limitations, tradeoffs, and validation records in `docs/<pattern>-notes.md`.

Add links to both files in the pattern index above. Verify language versions against official sources and maintain the shared baseline table here. Keep documentation contents lists and relative links current.

Agent instructions live in [skills/design-patterns/SKILL.md](skills/design-patterns/SKILL.md).
