# Singleton

## TypeScript

```typescript
class Singleton {
  static #instance: Singleton | undefined;

  // Later construction calls return the existing object before assigning value.
  value!: number;

  constructor() {
    if (Singleton.#instance !== undefined) {
      // A constructor can return an existing object instead of the new one.
      return Singleton.#instance;
    }

    this.value = Math.random();
    Singleton.#instance = this;
  }

  getValue(): number {
    return this.value;
  }
}

const first = new Singleton();
const second = new Singleton();

console.log(first === second); // true
console.log(first.getValue() === second.getValue()); // true
```

## TypeScript — ES5-compatible style

```typescript
interface Singleton {
  value: number;
  getValue(): number;
}

var Singleton = (function () {
  // The closure keeps the cached instance inaccessible to callers.
  var instance: Singleton | undefined;

  function Singleton(this: Singleton): Singleton {
    if (instance === undefined) {
      this.value = Math.random();
      instance = this;
    }

    return instance;
  }

  Singleton.prototype.getValue = function (this: Singleton): number {
    return this.value;
  };

  // Describe the construct signature of the function to TypeScript.
  return Singleton as unknown as { new (): Singleton };
})();

var first = new Singleton();
var second = new Singleton();

console.log(first === second); // true
console.log(first.getValue() === second.getValue()); // true
```

## Python

```python
from threading import Lock
from typing import ClassVar, Self, final


@final
class Singleton:
    _instance: ClassVar[Self | None] = None
    _lock: ClassVar[Lock] = Lock()
    __slots__ = ("_value",)

    def __new__(cls) -> Self:
        # Lock the complete check-and-create operation, including on free-threaded builds.
        with cls._lock:
            if cls._instance is None:
                instance = super().__new__(cls)
                instance._value = 0.5
                cls._instance = instance
            return cls._instance

    def get_value(self) -> float:
        return self._value


first = Singleton()
second = Singleton()

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
    -- The closure owns the cached instance; callers cannot replace the cache.
    local instance

    return {
        instance = function()
            if instance == nil then
                local value = 0.5
                instance = {
                    getValue = function(self)
                        return value
                    end,
                }
            end
            return instance
        end,
    }
end)()

local first = Singleton.instance()
local second = Singleton.instance()

assert(rawequal(first, second))
assert(first:getValue() == second:getValue())
print(rawequal(first, second)) -- true
print(first:getValue() == second:getValue()) -- true
```
