# Singleton

## TypeScript

```typescript
// A module owns one immutable service; export Singleton in a reusable module.
const Singleton = (() => {
  const value = Math.random();
  return Object.freeze({
    getValue(): number {
      return value;
    },
  });
})();

const first = Singleton;
const second = Singleton;

if (first !== second || first.getValue() !== second.getValue()) {
  throw new Error("Shared service invariant failed");
}
console.log(first === second); // true
console.log(first.getValue() === second.getValue()); // true
```

## TypeScript — ES5-compatible style

```typescript
interface SharedService {
  getValue(): number;
}

var Singleton: SharedService = (function () {
  var value = Math.random();
  return Object.freeze({
    getValue: function (): number {
      return value;
    },
  });
})();

var first = Singleton;
var second = Singleton;

if (first !== second || first.getValue() !== second.getValue()) {
  throw new Error("Shared service invariant failed");
}
console.log(first === second); // true
console.log(first.getValue() === second.getValue()); // true
```

## Python

```python
from dataclasses import dataclass


@dataclass(frozen=True, slots=True)
class _SharedService:
    value: float

    def get_value(self) -> float:
        return self.value


# Module initialization publishes one service; ordinary imports reuse the module.
_shared = _SharedService(0.5)


def instance() -> _SharedService:
    return _shared


first = instance()
second = instance()

assert first is second
assert first.get_value() == second.get_value()
print(first is second)  # True
print(first.get_value() == second.get_value())  # True
```

## Rust

```rust
mod singleton {
    use std::sync::LazyLock;

    pub struct Singleton {
        value: f64,
    }

    // LazyLock initializes once, even when multiple threads access it concurrently.
    static INSTANCE: LazyLock<Singleton> = LazyLock::new(|| Singleton { value: 0.5 });

    impl Singleton {
        pub fn instance() -> &'static Self {
            &INSTANCE
        }

        pub fn get_value(&self) -> f64 {
            self.value
        }
    }
}

fn main() {
    use singleton::Singleton;

    let first = Singleton::instance();
    let second = Singleton::instance();

    assert!(std::ptr::eq(first, second));
    assert_eq!(first.get_value(), second.get_value());
    println!("{}", std::ptr::eq(first, second)); // true
    println!("{}", first.get_value() == second.get_value()); // true
}
```

## Ruby

```ruby
require "singleton"

class SharedService
  include Singleton

  def initialize
    @value = 0.5
    freeze
  end

  def get_value
    @value
  end
end

first = SharedService.instance
second = SharedService.instance

raise "Different instances" unless first.equal?(second)
raise "Different values" unless first.get_value == second.get_value
puts first.equal?(second) # true
puts first.get_value == second.get_value # true
```

## C

```c
#include <assert.h>
#include <stdio.h>

// Public declarations: put these in singleton.h in a multi-file program.
typedef struct Singleton Singleton;
const Singleton *singleton_instance(void);
double singleton_get_value(const Singleton *self);

// Implementation: keep the layout and storage in singleton.c.
struct Singleton {
    double value;
};

static const Singleton instance = {.value = 0.5};

const Singleton *singleton_instance(void) {
    return &instance;
}

double singleton_get_value(const Singleton *self) {
    return self->value;
}

int main(void) {
    const Singleton *first = singleton_instance();
    const Singleton *second = singleton_instance();

    assert(first == second);
    assert(singleton_get_value(first) == singleton_get_value(second));
    puts(first == second ? "true" : "false");
    puts(singleton_get_value(first) == singleton_get_value(second) ? "true" : "false");
}
```

## C++

```cpp
#include <cassert>
#include <iostream>

class Singleton final {
public:
    static const Singleton& instance() {
        // Function-local static initialization is thread-safe.
        static const Singleton instance;
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    Singleton(Singleton&&) = delete;
    Singleton& operator=(Singleton&&) = delete;

    double get_value() const noexcept {
        return value_;
    }

private:
    Singleton() = default;
    ~Singleton() = default;
    const double value_ = 0.5;
};

int main() {
    const auto& first = Singleton::instance();
    const auto& second = Singleton::instance();

    assert(&first == &second);
    assert(first.get_value() == second.get_value());
    std::cout << std::boolalpha << (&first == &second) << '\n'; // true
    std::cout << (first.get_value() == second.get_value()) << '\n'; // true
}
```

## Mojo

```mojo
from std.builtin.globals import global_constant
from std.memory import Pointer
from std.origin import ImmStaticOrigin


@fieldwise_init
struct Singleton(Copyable):
    var value: Float64

    def get_value(self) -> Float64:
        return self.value


def singleton_instance() -> ref[ImmStaticOrigin] Singleton:
    # Store one immutable compile-time value in static storage.
    comptime instance = Singleton(0.5)
    return global_constant[instance]()


def main():
    # Use ref to borrow the shared storage, rather than copying its value.
    ref first = singleton_instance()
    ref second = singleton_instance()

    print(Pointer(to=first) == Pointer(to=second))  # True
    print(first.get_value() == second.get_value())  # True
```

## Java

```java
public enum Singleton {
    INSTANCE;

    private final double value;

    Singleton() {
        value = 0.5;
    }

    public double getValue() {
        return value;
    }

    public static void main(String[] args) {
        Singleton first = Singleton.INSTANCE;
        Singleton second = Singleton.INSTANCE;

        if (first != second || first.getValue() != second.getValue()) {
            throw new AssertionError("Singleton invariant failed");
        }
        System.out.println(first == second); // true
        System.out.println(first.getValue() == second.getValue()); // true
    }
}
```

## Swift

```swift
final class Singleton: Sendable {
    static let shared = Singleton()

    private let value: Double

    private init() {
        value = 0.5
    }

    func getValue() -> Double {
        value
    }
}

let first = Singleton.shared
let second = Singleton.shared

precondition(first === second)
precondition(first.getValue() == second.getValue())
print(first === second) // true
print(first.getValue() == second.getValue()) // true
```

## Kotlin

```kotlin
object Singleton {
    private val value: Double = 0.5

    fun getValue(): Double = value
}

fun main() {
    val first = Singleton
    val second = Singleton

    check(first === second)
    check(first.getValue() == second.getValue())
    println(first === second) // true
    println(first.getValue() == second.getValue()) // true
}
```

## Go

```go
package main

import (
    "fmt"
    "sync"
)

type singleton struct {
    value float64
}

// OnceValue publishes the same initialized pointer to every caller.
var instance = sync.OnceValue(func() *singleton {
    return &singleton{value: 0.5}
})

func (s *singleton) getValue() float64 {
    return s.value
}

func main() {
    first := instance()
    second := instance()

    if first != second || first.getValue() != second.getValue() {
        panic("singleton invariant failed")
    }
    fmt.Println(first == second) // true
    fmt.Println(first.getValue() == second.getValue()) // true
}
```

## Nim

```nim
type Singleton = ref object
  value: float64

# Initialize once when this module loads; keep the binding private.
let sharedInstance = Singleton(value: 0.5)

proc instance(): Singleton =
  sharedInstance

proc getValue(self: Singleton): float64 =
  self.value

when isMainModule:
  let first = instance()
  let second = instance()

  # Equality of ref objects checks identity, not field contents.
  doAssert first == second
  doAssert first.getValue() == second.getValue()
  echo first == second # true
  echo first.getValue() == second.getValue() # true
```

## Zig

```zig
const std = @import("std");

const Singleton = struct {
    value: f64,

    // One immutable value with static lifetime; no lazy initialization needed.
    const shared: @This() = .{ .value = 0.5 };

    fn instance() *const @This() {
        return &shared;
    }

    fn getValue(self: *const @This()) f64 {
        return self.value;
    }
};

pub fn main() void {
    const first = Singleton.instance();
    const second = Singleton.instance();

    if (first != second or first.getValue() != second.getValue()) {
        @panic("singleton invariant failed");
    }
    std.debug.print("{}\n", .{first == second}); // true
    std.debug.print("{}\n", .{first.getValue() == second.getValue()}); // true
}
```

## Lua

```lua
local Singleton = (function()
    -- One module service; no constructor or lazy cache is needed.
    local value = 0.5
    local service = {
        getValue = function(self)
            return value
        end,
    }
    return { instance = function() return service end }
end)()

local first = Singleton.instance()
local second = Singleton.instance()

assert(rawequal(first, second))
assert(first:getValue() == second:getValue())
print(rawequal(first, second)) -- true
print(first:getValue() == second:getValue()) -- true
```

## Scala

```scala
object Singleton:
  private val value: Double = 0.5
  def getValue: Double = value

@main def singletonDemo(): Unit =
  val first = Singleton
  val second = Singleton
  require(first eq second)
  require(first.getValue == second.getValue)
  println(first eq second) // true
  println(first.getValue == second.getValue) // true
```

## Haskell

```haskell
module Main (main) where

-- A pure shared service value; no mutable global or pointer-identity promise.
data SharedService = SharedService { getValue :: Double }
  deriving (Eq)

sharedService :: SharedService
sharedService = SharedService 0.5

main :: IO ()
main = do
  let first = sharedService
      second = sharedService
  if first == second && getValue first == getValue second
    then do
      print (first == second) -- True: value equality
      print (getValue first == getValue second) -- True
    else error "Shared value invariant failed"
```

## OCaml

```ocaml
module Singleton : sig
  type t
  val instance : t
  val get_value : t -> float
end = struct
  type t = { value : float }
  let instance = { value = 0.5 }
  let get_value self = self.value
end

let () =
  let first = Singleton.instance in
  let second = Singleton.instance in
  (* Structural equality checks the abstract service's immutable value. *)
  assert (first = second);
  assert (Singleton.get_value first = Singleton.get_value second);
  Printf.printf "%b\n%b\n"
    (first = second)
    (Singleton.get_value first = Singleton.get_value second)
```

## F#

```fsharp
module Singleton =
    type Service = private { Value: float }
    let instance = { Value = 0.5 }
    let getValue (service: Service) = service.Value

let first = Singleton.instance
let second = Singleton.instance

if not (obj.ReferenceEquals(first, second)) then
    failwith "Different service instances"
if Singleton.getValue first <> Singleton.getValue second then
    failwith "Different values"
printfn "%b" (obj.ReferenceEquals(first, second)) // true
printfn "%b" (Singleton.getValue first = Singleton.getValue second) // true
```

## Clojure

```clojure
(ns singleton-demo)

;; defonce preserves this immutable map across ordinary namespace re-evaluation.
(defonce ^:private shared-service {:value 0.5})

(defn instance [] shared-service)
(defn get-value [service] (:value service))

(let [first (instance)
      second (instance)]
  (when-not (and (identical? first second)
                 (= (get-value first) (get-value second)))
    (throw (ex-info "Shared service invariant failed" {})))
  (println (identical? first second)) ; true
  (println (= (get-value first) (get-value second)))) ; true
```

## Elixir

```elixir
defmodule SharedService do
  use GenServer

  def start_link(_opts), do: GenServer.start_link(__MODULE__, 0.5, name: __MODULE__)
  def instance, do: Process.whereis(__MODULE__)
  def get_value, do: GenServer.call(__MODULE__, :get_value)
  def set_value(value), do: GenServer.call(__MODULE__, {:set_value, value})

  @impl true
  def init(value), do: {:ok, value}

  @impl true
  def handle_call(:get_value, _from, value), do: {:reply, value, value}
  def handle_call({:set_value, value}, _from, _old), do: {:reply, :ok, value}
end

{:ok, supervisor} = Supervisor.start_link([SharedService], strategy: :one_for_one)

try do
  first = SharedService.instance()
  second = SharedService.instance()
  true = is_pid(first) and first == second
  {:error, {:already_started, ^first}} = SharedService.start_link([])

  # All clients address the same process, which serializes state updates.
  :ok = SharedService.set_value(0.75)
  true = SharedService.get_value() == 0.75
  IO.puts(first == second) # true
  IO.puts(SharedService.get_value() == 0.75) # true
after
  Supervisor.stop(supervisor)
end
```
