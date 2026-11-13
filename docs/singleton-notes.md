# Singleton notes

[Back to collection](../README.md) · [Code examples](../singleton.md)

## Contents

- [Overview](#overview)
- [Implementation notes](#implementation-notes)
- [Validation](#validation)
- [Tradeoffs](#tradeoffs)

## Overview

Keep one shared instance and return it on subsequent access. The TypeScript examples initialize a random value once; the other languages use a fixed value to emphasize shared identity. Each example demonstrates that callers receive the same instance and value.

## Implementation notes

### TypeScript — modern

The `#instance` field is private at runtime. The definite assignment assertion (`!`) tells TypeScript that every returned instance has an initialized value. The constructor remains public to preserve the `new Singleton()` API.

### TypeScript — ES5-compatible style

Run the blocks separately: they intentionally reuse the same names. Each implementation caches one instance per evaluation of its class or closure, rather than sharing an instance across processes, workers, or separate module copies. Avoid subclassing these implementations: returning a cached base instance does not preserve subclass identity. The public `value` remains mutable, as in the original examples.

### Python

Initialization happens in `__new__`, because `__init__` would run on every constructor call. `@final` tells type checkers to reject subclasses; it does not enforce that restriction at runtime. The lock protects creation, while `_value` is private by convention. Deliberate reflection, copying, or serialization requires separate handling if uniqueness must hold through those operations.

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

The closure caches one table on first access and keeps its value in a private local variable. `rawequal` checks table identity without invoking equality metamethods. The accessor does not yield, so initialization completes before another coroutine in the same Lua state can access it. Separate Lua states or separate evaluations of the closure have their own instances. The returned table remains mutable: callers can replace its methods even though the captured value is private. For a reusable module, return the service API from a module and use `require` to cache it by module name within that Lua state. See the [Lua 5.5 reference manual](https://www.lua.org/manual/5.5/manual.html).

## Validation

The non-TypeScript examples use a fixed value (`0.5`) to keep attention on shared identity and initialization. Run each code block independently. All implementations are scoped to their program/module or class-loading environment; none coordinate a single instance across separate processes.

The following local checks ran on November 13, 2026. Every executed example passed its identity and value checks and printed two true results. The [release baselines in the README](../README.md#language-baselines) are separate from the installed toolchains below; execution on an older compiler does not verify the latest release's compiler.

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

## Tradeoffs

A singleton can represent shared configuration or a resource registry, but it introduces shared mutable state and makes tests harder to isolate. Prefer passing a dependency explicitly when callers need independent instances or controlled lifetimes.
