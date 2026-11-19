# Singleton notes

[Back to collection](../README.md) · [Code examples](../singleton.md)

## Contents

- [Overview](#overview)
- [Observable guarantees](#observable-guarantees)
- [Implementation notes](#implementation-notes)
- [Validation](#validation)
- [Tradeoffs](#tradeoffs)

## Overview

Preserve a single access point to a shared service using each language's idioms. Most examples demonstrate reference identity and a stable value. Haskell and OCaml demonstrate an immutable value exposed through one binding; their equality checks do not prove unique allocation. Elixir demonstrates a single named process owning changing state. See [why each language is included](language-guide.md).

TypeScript initializes a random value once; the other immutable services use `0.5`. Elixir starts at `0.5` and changes to `0.75` to demonstrate shared state ownership. Run blocks independently.

## Observable guarantees

| Approach | Languages | Demonstrated guarantee | Boundary |
| --- | --- | --- | --- |
| Frozen module service | TypeScript (both styles), Python | Repeated access returns the same read-only service and value. | One module evaluation/import identity; Python privacy and freezing are ordinary API protections. |
| Runtime-managed shared object | Ruby, Java, Swift, Kotlin, Scala | Repeated access has reference identity. | One class/module loading context; construction safety does not synchronize later mutation. |
| Static storage or once-only accessor | Rust, C, C++, Go, Nim, Zig, Mojo | Repeated access refers to shared storage. | One linked storage/module instance; several languages still permit constructing other values of the type. |
| Module service with a private captured value | Lua | Repeated access returns the same table and captured value. | One closure evaluation/Lua module cache entry; the table itself remains mutable. |
| Immutable value behind an API | Haskell, OCaml | Repeated access yields equal service data. | No physical identity claim; Haskell's optimizer may change allocation and sharing. |
| Module-bound immutable reference | F#, Clojure | Repeated access returns the same immutable record/map reference. | One loaded module/namespace binding; reloads and deliberate rebinding have separate semantics. |
| Supervised named state owner | Elixir | A second named start fails while the process is alive; clients observe its serialized state updates. | One BEAM node and registered name; restart replaces PID and resets state. |

These are related solutions, not interchangeable uniqueness guarantees. Use the identity-based implementations when one reference is itself a requirement. Use immutable values when shared data is sufficient. Use a managed process when clients need one owner coordinating mutable state. None of these examples enforces uniqueness across independent operating-system processes or machines.

## Implementation notes

### TypeScript — modern

Run as `node singleton.ts` with a Node version supporting TypeScript type stripping, or compile with a compatible TypeScript compiler targeting ES2022 or newer. One module binding exposes a frozen object whose method closes over the initialized value. In a reusable module, export this binding and import it from clients. `Object.freeze` is shallow, but this service contains only a function and a captured scalar. Workers, independent module copies, and re-evaluation create separate services.

This replaces the former public constructor that returned a cached instance. Callers now use `Singleton` directly instead of `new Singleton()`; the random value is private, and creation happens during module evaluation rather than on first construction. This intentionally removes the mutable public `value` and subclassing behavior.

### TypeScript — ES5-compatible style

Run the blocks separately: they intentionally reuse the same names. Compile with a TypeScript compiler supporting `--target ES5`. The closure and frozen object provide the same module-service API as the modern block without modern JavaScript syntax. `Object.freeze` is available in ES5. Node's type stripping can check its runtime behavior but does not verify ES5 compiler output or type checking. This replaces the former function-constructor/prototype cache and mutable public value.

### Python

Run with `python3 singleton.py`. Ordinary imports reuse the initialized module and its frozen dataclass service; no custom `__new__` or creation lock is needed. Keep the service and accessor in a dedicated module in a real program. The public API changes from `Singleton()` to `instance()` while preserving shared identity and read access. The frozen record prevents ordinary field assignment; its scalar field needs no deeper freezing. `_SharedService` and `_shared` are private by convention, so callers can deliberately create another record or replace module bindings. Module reloads, separate interpreters, alternate import identities, and separate processes do not share this guarantee.

### Rust

The private field prevents callers outside the module from constructing the type. Shared references expose read-only access. If the service later needs mutation, use synchronization such as a `Mutex` rather than a mutable global. Static values are not automatically dropped at program exit.

### Ruby

Ruby's standard-library `Singleton` module makes `new` and `allocate` private and provides synchronized lazy creation through `.instance`. The class is named `SharedService` to avoid shadowing that module. `freeze` prevents changes to this instance's fields; it does not recursively freeze referenced objects. A subclass gets its own singleton, so keep this service unsubclassed when one shared instance is intended.

### C

Compile with `cc -std=c23 singleton.c -o singleton`.

Static initialization happens before execution and needs no lazy-initialization lock. The returned pointer refers to read-only storage and must not be freed. Splitting the opaque declaration into a header prevents clients from constructing the type; keep the storage in exactly one implementation file to avoid separate instances in each translation unit. Runtime initialization would require a separate once-only mechanism.

### C++

Compile with `c++ -std=c++23 singleton.cpp -o singleton`.

The private constructor and deleted copy/move operations prevent ordinary callers from creating another instance. Initialization is lazy, and access is read-only. The instance is destroyed at program termination; avoid using it from other static destructors after its lifetime has ended.

### Mojo

Run with `mojo run singleton.mojo`.

This is an immutable shared-instance approach using [`global_constant`](https://mojolang.org/docs/std/builtin/globals/global_constant/), rather than a runtime lazy constructor. It supports trivial, self-contained values and cannot initialize a runtime resource such as a socket. References preserve shared storage; copying the struct creates a separate value, and its public initializer remains callable. The example therefore demonstrates a shared service accessor rather than enforcing unique construction of the type. See the official [materialization guide](https://mojolang.org/docs/manual/metaprogramming/materialization/) and [pointer API](https://mojolang.org/docs/std/memory/pointer/Pointer/). This example has been checked against the documentation but not executed locally because Mojo is unavailable.

### Java

Save as `Singleton.java` and run with `java Singleton.java`.

A single-element enum provides a shared instance with JVM-managed initialization. It also preserves identity during Java serialization and prevents ordinary reflective constructor calls. The scope is one instance per defining class loader, rather than one instance across every class loader in a JVM.

### Swift

Run with `swift -swift-version 6 singleton.swift`.

`static let` initializes lazily and once. `final`, immutable stored properties, and `Sendable` allow safe sharing with Swift's strict concurrency checks. If the service needs mutable state, isolate it with an actor or an appropriate lock rather than adding unsynchronized mutable fields.

### Kotlin

Save as `Singleton.kt`, compile with `kotlinc Singleton.kt -include-runtime -d singleton.jar`, and run with `java -jar singleton.jar`.

An `object` declaration gives the JVM implementation a lazily initialized, thread-safe shared instance without a public constructor. Initialization safety does not make mutable members thread-safe. As with Java, separately defining the class in multiple class loaders creates separate instances.

### Go

Run with `go run singleton.go`.

`sync.OnceValue` provides lazy initialization safe for concurrent callers. The unexported type and field restrict external package access, but code in this package can still construct or copy the type. In a reusable package, expose an accessor and keep construction internal. Protect any later mutable state separately; once-only initialization does not synchronize future writes.

### Nim

Run with `nim c -r singleton.nim`.

The module creates one reference object during initialization and the accessor returns that same reference. Reference equality demonstrates shared identity. The example follows the [Nim manual](https://nim-lang.org/docs/manual.html); it targets ordinary single-threaded module access. The private binding and unexported type keep the service internal, but code in this module can still construct another object or mutate the field. `let` fixes the reference binding; it does not make the referenced object immutable. Sharing mutable state between threads requires a separate design for synchronization and ownership.

### Zig

Run with `zig run singleton.zig`.

The accessor returns a pointer to one immutable struct with static lifetime, so reads need no lazy-initialization lock or allocator. Pointer comparison demonstrates shared identity. The type can still be constructed or copied within code that can access it; this is a shared service accessor, not a restriction on all construction. The functions are file-local; add `pub` to the intended declarations when exposing an API from a separate module. `std.debug.print` writes the two results to standard error. See the [Zig 0.17.1 reference](https://ziglang.org/documentation/0.17.1/).

### Lua

Run with `lua singleton.lua` using the Lua 5.5.1 interpreter.

The closure creates one service table during evaluation and keeps its scalar value in a private local. `rawequal` checks identity. This replaces the former lazy constructor cache while preserving `Singleton.instance()` and the observed value. The returned table and accessor table remain mutable; callers can replace methods even though the captured value is private. For a reusable module, return the service API from a file and use `require`, whose cache is scoped to a module name within one Lua state. Separate evaluations and states have their own service. See the [Lua manual](https://www.lua.org/manual/5.5/manual.html).

### Scala

Save as `singleton.scala` and run with `scala run singleton.scala --scala 3.10.1` using the Scala CLI launcher.

A top-level `object` directly expresses a lazily initialized singleton; no handwritten constructor cache is needed. `eq` checks reference identity. The private immutable scalar supports concurrent reads; mutable fields would require their own synchronization. The JVM scope is one defining class loader, not all JVMs or every class loader. See [singleton objects](https://docs.scala-lang.org/tour/singleton-objects.html).

### Haskell

Save as `singleton.hs` and run with `runghc singleton.hs`, or compile with `ghc -Wall singleton.hs -o singleton`.

`sharedService` is a pure top-level binding. The example checks structural equality and a stable field value, not pointer identity or exactly-once resource acquisition. Lazy evaluation and compiler optimization can alter physical sharing without changing the program's meaning. The constructor stays internal through `Main`'s export list; in a library, move the service into its own module and export the abstract type, binding, and getter. No `unsafePerformIO` or mutable global is needed. Acquire real resources in `IO` and pass them explicitly with a controlled lifetime. See [pure values](https://www.haskell.org/tutorial/goodies.html) and [IO](https://www.haskell.org/tutorial/io.html).

### OCaml

Save as `singleton.ml` and run with `ocaml singleton.ml`, or compile with `ocamlopt -o singleton singleton.ml`.

The sealed module signature exposes an abstract type, an immutable instance, and a getter. Clients cannot ordinarily construct or update the hidden record. The example checks structural equality (`=`); physical equality of immutable values is not an API promise. Module initialization supplies the binding, rather than a class constructor or runtime resource cache. Functor applications or different loaded implementations can introduce other service modules. See [modules](https://ocaml.org/docs/modules).

### F#

Save as `singleton.fsx` and run with `dotnet fsi --langversion:11.0 singleton.fsx` using .NET SDK 11.0.100. No preview language features are required.

A module binds one immutable record with a private representation. `obj.ReferenceEquals` deliberately checks reference identity rather than record structural equality. Ordinary clients cannot invoke its record constructor outside the module. Module initialization is managed by the runtime; separate loading contexts or script evaluations can produce separate bindings. Adding `mutable` fields would need synchronization. See [F# modules](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/modules).

### Clojure

Save as `singleton.clj` and run with `clojure -Sdeps '{:deps {org.clojure/clojure {:mvn/version "1.12.6"}}}' -M singleton.clj`.

`defonce` binds one immutable map and preserves it across ordinary namespace re-evaluation; `identical?` checks the returned reference. The Var itself can still be deliberately rebound or removed, and removing/recreating a namespace creates a new binding. `^:private` expresses an API boundary, not a security boundary. Namespace loading and REPL redefinition are different from serializing access to mutable state. For changing state, choose an atom or ref as appropriate rather than mutating the immutable map. See [Vars](https://clojure.org/reference/vars).

### Elixir

Save as `singleton.exs` and run with `elixir singleton.exs`. Run in a fresh VM so no service is already registered under `SharedService`.

The supervisor starts one named `GenServer`. Concurrent attempts to register the same local name are rejected by the runtime; the example checks `{:already_started, pid}` and proves that a subsequent read observes the state update. Synchronous calls are processed serially. The private process state changes through messages; module code itself is not the stateful instance. In an application, place the child under the application's existing supervision tree. The script stops its supervisor after the demonstration.

The registered name is unique only on one BEAM node. On failure, supervision can start a replacement with a new PID and initial state `0.5`; it does not preserve in-memory state. Calls during failure/restart can exit, and a captured PID can become stale. Distributed uniqueness and persistent state need additional coordination. See [GenServer](https://hexdocs.pm/elixir/GenServer.html).

## Validation

The immutable examples check stable values; most also check identity. Haskell and OCaml check value equality, while Elixir checks registration exclusivity and updated state. See the guarantee table above for what each check establishes.

The following historical checks ran on November 13, 2026, before the module-service revisions below. The Python result applies to the previous constructor-based example. Every executed example passed its identity and value checks and printed two true results. The [release baselines in the README](../README.md#language-baselines) are separate from the installed toolchains below; execution on an older compiler does not verify the latest release's compiler.

| Example | Local toolchain | Result |
| --- | --- | --- |
| Python | Python 3.15.2 | Passed |
| Rust | rustc 1.99.1, edition 2024 | Passed |
| Ruby | Ruby 4.0.9 | Passed |
| C | Apple Clang 21.0.0, `-std=c23 -Wall -Wextra -Werror` | Passed |
| C++ | Apple Clang 21.0.0, `-std=c++23 -Wall -Wextra -Werror` | Passed |
| Mojo | Not installed | Documentation review only; execution unverified |
| Java | OpenJDK 25.0.1 | Passed |
| Swift | Swift 6.3.3, `-swift-version 6` | Passed |
| Kotlin | Not installed | Execution unverified |
| Go | Go 1.27.3 | Passed |
| Nim | Not installed | Documentation review only; execution unverified |
| Zig | Not installed | Documentation review only; execution unverified |
| Lua | Not installed | Execution unverified |

### November 19, 2026 revision checks

| Changed or new example | Local check | Result |
| --- | --- | --- |
| TypeScript — modern | Node v26.13.0, native TypeScript stripping | Passed shared-identity and stable-value checks; type checking/compiler output unverified |
| TypeScript — ES5-compatible style | Node v26.13.0, native TypeScript stripping | Passed runtime checks; ES5 compilation and type checking unverified (no TypeScript compiler installed) |
| Python | Python 3.15.2 | Passed identity/value checks, ordinary import reuse, and rejection of field assignment |
| Lua | No interpreter installed | Reviewed module-service API and closure scope; execution unverified |
| Scala | No Scala launcher/compiler installed | Reviewed against singleton-object documentation; execution unverified |
| Haskell | No GHC/runghc installed | Reviewed pure value semantics and usage; execution unverified |
| OCaml | No interpreter/compiler installed | Reviewed module abstraction and structural equality; execution unverified |
| F# | No .NET SDK installed | Reviewed module and immutable-record API; execution unverified |
| Clojure | No Clojure CLI installed | Reviewed namespace binding and identity checks; execution unverified |
| Elixir | No Elixir/Erlang toolchain installed | Reviewed named registration, callbacks, supervision, and cleanup; execution unverified |

The eleven other language code blocks were compared against the previous revision and are byte-for-byte unchanged. Checked all 21 blocks (twenty languages plus the TypeScript legacy variant), the code-only entry format, balanced Markdown fences, relative documentation links and anchors, README language coverage, and whitespace with `git diff --check`. Missing toolchains mean the new examples' runtime behavior remains unverified locally; documentation review is not an execution result.

## Tradeoffs

A singleton can represent shared configuration or a resource registry. When mutable, it introduces shared state and makes tests harder to isolate. Prefer passing a dependency explicitly when callers need independent instances or controlled lifetimes.
