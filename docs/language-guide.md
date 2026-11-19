# Why these languages

[Back to collection](../README.md) · [Singleton examples](../singleton.md) · [Singleton guarantees](singleton-notes.md#observable-guarantees)

## Contents

- [What each language adds](#what-each-language-adds)
- [Functional additions](#functional-additions)
- [Preserving pattern intent](#preserving-pattern-intent)

## What each language adds

These are reasons for including each language in this collection and practical situations in which to consider it. They are editorial judgments about useful contrasts, rather than a popularity ranking or a claim that one language is universally better.

| Language | What it adds | When to use it |
| --- | --- | --- |
| TypeScript | Structural types, closures, and JavaScript module services; compares compile-time contracts with runtime guarantees. | JavaScript applications needing explicit API and domain types. |
| Python | Readable, dynamic code and import-cached modules; demonstrates when a class-based pattern can become a module service. | Automation, data processing, and services where iteration speed matters. |
| Rust | Ownership, borrowing, explicit synchronization, and static lifetimes. | Native services and libraries requiring memory safety and control of resources. |
| Ruby | Expressive objects, mixins, and a standard-library Singleton implementation. | Applications and tools benefiting from a dynamic object model and concise domain APIs. |
| C | Opaque handles, translation-unit storage, and explicit lifetime management. | Embedded software, systems interfaces, and portable native libraries. |
| C++ | RAII, restricted construction, and synchronized function-local static initialization. | Native applications requiring resource control and integration with C++ libraries. |
| Mojo | Value semantics and explicit references to materialized static storage. | Compute-oriented code where data layout and specialized native execution matter; evaluate ecosystem and toolchain requirements first. |
| Java | Enums, class-loader boundaries, and JVM-managed initialization. | JVM services needing established libraries and explicit object contracts. |
| Swift | Immutable shared references, `Sendable`, and actor-based options for mutable state. | Apple-platform applications and Swift services needing concurrency checks. |
| Kotlin | Built-in singleton objects and concise JVM interoperation. | Android applications and JVM services with Kotlin libraries. |
| Go | Package boundaries, small APIs, and once-only initialization with explicit concurrency tools. | Network services and command-line tools with straightforward deployment. |
| Nim | Module initialization, reference/value distinctions, and native compilation. | Native utilities where concise syntax and C integration are useful. |
| Zig | Explicit allocation, compile-time values, and static storage without hidden initialization machinery. | Systems components needing predictable memory use and C integration. |
| Lua | Closures, tables, and module caching within an interpreter state. | Embedded scripting, application customization, and game logic. |
| Scala | Functional and object-oriented composition, with a native singleton-object construct. | JVM applications combining typed domain models and existing Java APIs. |
| Haskell | Pure functions, lazy evaluation, and explicit effects; exposes where identity is unnecessary. | Programs benefiting from strong functional modeling and explicit boundaries around effects. |
| OCaml | Algebraic data types, inference, and abstract module interfaces without requiring classes. | Compilers, analyzers, and native applications with rich symbolic models. |
| F# | Functional domain modeling and modules alongside .NET interoperability. | .NET applications that benefit from immutable data and concise transformations. |
| Clojure | Dynamic Lisp, immutable collections, and explicit mechanisms for managing changing references. | JVM applications organized around data transformations and interactive development. |
| Elixir | Immutable data and state owned by supervised processes. | Concurrent services requiring process isolation and recovery from failures. |

## Functional additions

### Scala

Scala bridges the existing Java/Kotlin examples and functional programming. Its `object` declaration expresses Singleton directly; algebraic data types, pattern matching, and function values also let future patterns be expressed without unnecessary class hierarchies. Use it when that combination and JVM interoperability are useful. The collection targets Scala 3 on the JVM; class-loader and initialization boundaries still matter. [Official overview](https://www.scala-lang.org/) · [Singleton objects](https://docs.scala-lang.org/tour/singleton-objects.html)

Further reading: [Sergiy Shcherbyna’s perspective on Scala for full-stack development](https://www.linkedin.com/posts/sergiy-shcherbyna-b904751_scala-scala3-scalajs-share-7521472221115154433-_WsX). The post discusses JVM, JavaScript, and native targets, type-driven effect handling, and sharing application logic across frontend and backend. It offers a practitioner’s perspective on Scala’s strengths and ecosystem limitations.

### Haskell

Haskell provides the clearest contrast with mutable, identity-based designs. Pure functions and lazy evaluation encourage separating reusable values from resource-owning actions. It makes a useful teaching language for asking whether Strategy needs an interface, or whether a shared service needs unique physical identity. The Singleton example intentionally promises value semantics, not allocation identity. A socket, connection pool, or other runtime resource should instead be created in `IO`, passed explicitly, and given a controlled lifetime. [Official introduction](https://www.haskell.org/onlinereport/intro.html) · [Effects and IO](https://www.haskell.org/tutorial/io.html)

### OCaml

OCaml demonstrates how module signatures hide representations and constructors while exposing a small service API. Its algebraic data types and pattern matching are useful for compilers, interpreters, and domain models. Modules provide an alternative to translating every pattern into classes; functors support parameterized implementations when multiple services are needed. The example seals an immutable record behind an abstract type and checks structural equality rather than promising physical identity. [Modules](https://ocaml.org/docs/modules) · [Language styles](https://ocaml.org/docs/guidelines)

### F#

F# adds functional-first design within .NET. Records, discriminated unions, and functions support compact domain models, while ordinary .NET APIs remain available. Its module-based shared record shows that a Singleton-like service need not have a public class constructor or a constructor cache. For changing state, use an explicitly synchronized owner rather than assuming that a module makes writes safe. [Official overview](https://fsharp.org/) · [Modules](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/modules)

### Clojure

Clojure brings a Lisp perspective alongside the statically typed functional languages. Immutable collections let code share data safely while functions supply behavior. Its shared map avoids a dedicated service class; `defonce` also illustrates namespace re-evaluation during interactive development. If changing state is required, choose a reference mechanism such as an atom or ref according to the coordination needed. A private Var is an API boundary, not protection against deliberate redefinition. [Functions](https://clojure.org/guides/learn/functions) · [Vars and reference mechanisms](https://clojure.org/reference/vars)

### Elixir

Elixir adds a different answer to shared mutable state: one process owns it and clients send messages. A named `GenServer` under a supervisor demonstrates serialized updates and a managed lifecycle. This is useful for concurrent services, but a single process can become a bottleneck. Its registered name is unique on one BEAM node; restarting it changes the PID and resets the example's state. Distributed uniqueness, persistence, and partition handling require additional design. [Official overview](https://elixir-lang.org/) · [GenServer](https://hexdocs.pm/elixir/GenServer.html)

## Preserving pattern intent

Start with the problem a pattern solves, then choose the language's natural tools. Strategy can be a function, Factory can be a constructor function, and a service can live in a module. Keep classes when their identity, lifecycle, or interfaces help express the requirement.

For Singleton, distinguish three needs: reusable immutable data, one shared service reference, and one owner coordinating changing state. Equality of values proves neither unique allocation nor shared mutable state. A module can expose one service without preventing construction elsewhere. A registered process can enforce one active name while allowing replacement after failure. Document those differences and demonstrate the actual invariant instead of printing an identity claim the language does not promise.

The [Singleton notes](singleton-notes.md) specify initialization, mutation, scope, and lifecycle limits for the implementations. Prefer passing dependencies explicitly when tests or callers need independent instances.
