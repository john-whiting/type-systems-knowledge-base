---
theme: css/my-theme.css
---
## Type Systems Day 1
### By: Jaran Chao and John Whiting

---

## Overview

### High-Level properties of type systems

- Strong vs. Weak Typing
- Static vs. Dynamic
- Nominal vs. Structural
- Duck Typing
- Manifest vs. Inferred

### Advanced properties of type systems
- Type Inference
- Turing Completeness

---

## Types are a Spectrum

![[strong-weak-static-dynamic-spectrum-graph.webp]]

---

## Static vs. Dynamic

| Static Typing                                                                 |                                                           Dynamic Typing |
| :---------------------------------------------------------------------------- | -----------------------------------------------------------------------: |
| The type of a variable is known at <i style="color:#4e85e4;">compile time</i> | The type of a variable is known at <i style="color:#4e85e4;">runtime</i> |
| The type of a variable <i style="color:#ef3535;">cannot</i> change            |          The type of a variable <i style="color:#4def4d;">can</i> change |

---

## Strong vs. Weak

| Strong Typing                                           |                                              Weak Typing |
| :------------------------------------------------------ | -------------------------------------------------------: |
| <i style="color:#ef3535;">Never</i> possible to convert | <i style="color:#4def4d;">Always</i> possible to convert |

There are no standardized/agreed upon definitions for strong and weakly typed languages.
However, the above terms will be used for the rest of the presentation. <!-- element style="font-size: 1.5rem;" -->

---

## Python (Strong-Dynamic)

```python
print("hello world" + 10) # Gives a TypeError
print("hello world" + str(10)) # Prints "hello world10"

# Magic Method __str__ for type casting
class Test:
    def __str__(self):
        return "test"

print("hello world" + Test()) # Gives a TypeError
print("hello world" + str(Test())) # Prints "hello worldtest"
```

[Godbolt](https://godbolt.org/z/eP9c4fxj4)

---

## JavaScript (Weak-Dynamic)

```js
// Evalulates to 0
console.log(+[]);

// Evaulates to 1
console.log(+!![]);

// Evaulates to true
console.log([] == 0);

// Evaulates to "banana"
console.log(("b" + "a" + + "a" + "a").toLowerCase());
```

[Godbolt](https://godbolt.org/z/v47GG6Me9)

---

## C/C++ (Weak-Static)

```c
// Fast Inverse Square Root Algorithm for Quake III Arena
float q_rsqrt(float number) {
    long i;
    float x2, y;
    const float threehalfs = 1.5F;
    
    x2 = number * 0.5F;
    y  = number;
    i  = * ( long * ) &y; // evil floating point bit level hacking
    i  = 0x5f3759df - ( i >> 1 ); // what the...
    y  = * ( float * ) &i;
    y  = y * ( threehalfs - ( x2 * y * y ) );
    
    return y;
}
```

[Godbolt](https://godbolt.org/z/WqajehrvK)

---

## Java (Strong-Static)

```java
public class Main {
    static class Test {
        @Override
        public String toString() {
            return "test";
        }
    }
    public static void main(String[] args) {
        // Prints "hello world10"
        System.out.println("hello world" + 10);
        
        // Prints "hello worldtest"
        System.out.println("hello world" + new Test());
    }
}
```

[Godbolt](https://godbolt.org/z/a78jP8G96)

---

## Nominal vs. Structural

| Nominal                                                                   |                                                                         Structural |
| :------------------------------------------------------------------------ | ---------------------------------------------------------------------------------: |
| Determines if types are equal based on <i style="color:#4e85e4;">name</i> |     Determines if types are equal based on <i style="color:#4e85e4;">structure</i> |
| Types are <i style="color:#4e85e4;">explicitly</i> declared to be related | Types are <i style="color:#4e85e4;">implicitly</i> related if the structures match |

---

## Nominal Example (C)

```c
typedef struct {
    int x, y;  
} vector_2d;

typedef struct {
    int x, y, z;
} vector_3d;

double my_abs(vector_2d vec) {
    return sqrt(vec.x * vec.x + vec.y * vec.y);
}

int main() {
    vector_2d vec2d = { .x = 3, .y = 4 };
    vector_3d vec3d = { .x = 3, .y = 4, .z = 1000000 };

    printf("%f", my_abs(vec2d));
    printf("%f", my_abs(vec3d)); // error: incompatible type
}
```

[Godbolt](https://godbolt.org/z/oczWqsY8a)

---

## Structural Example (TS)

```typescript
interface Vector2d {
  x: number,
  y: number,
};

interface Vector3d {
  x: number,
  y: number,
  z: number,
};

function magnitude({ x, y }: Vector2d): number {
  return Math.sqrt(x * x + y * y);
}

const vec2d: Vector2d = { x: 3, y: 4 };
const vec3d: Vector3d = { x: 3, y: 4, z: 100000000 };

console.log(magnitude(vec2d) === 5); // Should be 5
console.log(magnitude(vec3d) !== 5); // Should not be 5, but it is
```

[ReplIt](https://replit.com/@JohnathanWhitin/StructuralTypeSystemExample)

---

## Structural Example (Go)

```go
type Abser interface {
    Abs() float64
}

type Vector struct {
    X, Y float64
}

func (v Vector) Abs() float64 {
    return math.Sqrt(v.X * v.X + v.Y * v.Y)
}

var _ Abser = (*Vector)(nil) // succeeds as a Vector implements Magnitude implicitly

type Box struct {
    Value float64
}

var _ Abser = (*Box)(nil) // won't succeed as a Box is not an Abser implicitly
```

[Godbolt](https://godbolt.org/z/4ohxYc3Gh)

---

## Structural Example (C++20)

```cpp
template<typename T>
concept Absable = requires(T t) {
    { t.abs() } -> std::convertible_to<double>;
};

struct Vector {
    double x, y;
    auto abs() -> double {
        return std::sqrt(this->x * this->x + this->y * this->y);
    }
};

static_assert(Absable<Vector>); // succeeds as Vector has the abs method

struct Box {
    double value;
};

static_assert(Absable<Box>); // fails as Box does not have the abs method
```

[Godbolt](https://godbolt.org/z/cEPb3Y89K)

---

## Duck Typing

If it walks like a duck and quacks like a duck, then it must be a duck!

| Structural Typing                                                               |                                                                Duck Typing |
| :------------------------------------------------------------------------------ | -------------------------------------------------------------------------: |
| Checks for structural equivalence at <i style="color:#4e85e4;">compile time</i> | Checks for structural equivalence at <i style="color:#4e85e4;">runtime</i> |

---

## Duck Typing Example

```python
from dataclasses import dataclass

@dataclass
class Vector:
    x: float
    y: float
    
    def abs(self):
        return (self.x * self.x + self.y * self.y) ** (0.5)

@dataclass
class Box:
    value: float

def absOf(a) -> float:
    return a.abs()

print(absOf(vec)) # prints 5.0
print(absOf(box)) # AttributeError: 'Box' object has no attribute 'abs'
```

[Godbolt](https://godbolt.org/z/ssxa43vof)

---

## Manifest vs. Inferred

| Manifest                                                          |                                                           Inferred |
| :---------------------------------------------------------------- | -----------------------------------------------------------------: |
| All types must be <i style="color:#4e85e4;">explicitly</i> stated | All types can be <i style="color:#4e85e4;">implicitly</i> inferred |

---

## Example

```cpp
auto main() -> int {
    double pi = 3.14;
    auto alsoPi = 3.14;
    
    double someConstant = 10; // known to be double
    auto whoops = 10;         // inferred as int instead of double
}
```

<table style="margin: 0;">
    <thead>
        <tr>
            <td colspan="5" style="text-align: center;">Common Keywords</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>auto</td>
            <td>let</td>
            <td>var</td>
            <td>val</td>
            <td>def</td>
        </tr>
    </tbody>
</table>

---

## Type Inference

|    | *Type Inference* [n]                                                      |
| -: | ------------------------------------------------------------------------- |
| 1. | The automatic detection of the type of an expression in a formal language |

A spectrum ranging from basic expression level to a full program.
An example type system that allows for full program type inference is called Hindley-Milner (HM).

---

## Type Inference (Rust)

```rust
// (note that this is typically not done for creating vectors.
// `vec![]` is used instead.)
fn main() {
    let mut items = Vec::new(); // Initially => Vec<{unknown}>

    items.push(2); // items => Vec<i32>
    items.push(6);
    items.push(1);
    items.push(3);

    println!("{items:?}");
}
```

[Godbolt](https://godbolt.org/z/c6vz4shze)

---

## Type Inference (Kotlin)

```kotlin
fun <T> MutableList<T>.maybeAddSomeElements() {}

fun printLong(long: Long?) {
    println(long ?: -1L)
}

inline fun <reified T> printInnerType(list: List<T>) {
    println("$list has type ${T::class}")
}

fun main() {
    val list = buildList {
        maybeAddSomeElements()
        
        val firstElem = getOrNull(0)
        printLong(firstElem)
    } // inferred to be List<Long>
    
    printInnerType(list)
}
```

[Kotlin Playground](https://pl.kotl.in/doHt0utnH)

---

## Type Inference (Algo W)

![[algorithm-w.png]]

[Wikipedia](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system#Algorithm_W)

---

## Type Inference (Algo J)

![[algorithm-j.png]]

[Wikipedia](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system#Algorithm_J)

---

## Type Inference (Algo M)

![[algorithm-m.png]]

[ACM Journal Vol. 20 No. 4](https://dl.acm.org/doi/10.1145/291891.291892)

---

## Turing Completeness

|    | *Turing Complete* [adj]                                                 |
| -: | ----------------------------------------------------------------------- |
| 1. | A type system is Turing complete if it can simulate any Turing machine. |

<table>
    <thead>
        <tr>
            <td></td>
            <td><b><i>Turing Machine</i> [n]</b></td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td style="border-bottom: unset">1.</td>
            <td style="border-bottom: unset">An abstract computation machine that manipulates symbols on a tape strip.</td>
        </tr>
        <tr>
            <td>2.</td>
            <td>A machine that can implement any algorithm.</td>
        </tr>
    </tbody>
</table>

---

## Turing Completeness (cont.)

Popular languages that have Turing complete type systems.

<table style="margin: 0;">
    <tbody>
        <tr>
            <td style="color: #3074c0;">TypeScript</td>
            <td style="color: #f74c00;">Rust</td>
            <td style="color: #6295cb;">C++</td>
            <td style="color: #d73322;">Scala</td>
            <td style="color: #5e5086">Haskell</td>
        </tr>
    </tbody>
</table>

---

## Turing Completeness (TS)

The BrainFuck language written using only types

```ts
import type { Brainfuck } from "@susisu/typefuck";

type Program = ">,[>,]<[.<]";
type Input = "Hello, world!";
type Output = Brainfuck<Program, Input>; // = "!dlrow ,olleH"
```

[typefuck](https://github.com/susisu/typefuck)
