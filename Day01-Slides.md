---
theme: simple
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

---

## Types are a Spectrum

![[strong-weak-static-dynamic-spectrum-graph.webp]]

---

## Static vs. Dynamic

| Static Typing                                                              |                                                        Dynamic Typing |
| :------------------------------------------------------------------------- | --------------------------------------------------------------------: |
| The type of a variable is known at <i style="color:#22d;">compile time</i> | The type of a variable is known at <i style="color:#22d;">runtime</i> |
| The type of a variable <i style="color:#e22;">cannot</i> change            |          The type of a variable <i style="color:#4e4;">can</i> change |

---

## Strong vs. Weak

Strong Typing                                         | Weak Typing
:-----------------------------------------------------|-----------------------------------------------------:
 <i style="color:#e22;">Never</i> possible to convert | <i style="color:#4e4;">Always</i> possible to convert

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

---

## Nominal vs. Structural

| Nominal                                                                |                                                                      Structural |
| :--------------------------------------------------------------------- | ------------------------------------------------------------------------------: |
| Determines if types are equal based on <i style="color:#22d;">name</i> | Determines if types are equal based on <i style="color:#22d;">structure</i>     |
| Types are <i style="color:#22d;">explicitly</i> declared to be related | Types are <i style="color:#22d;">implicitly</i> related if the structures match |

---

## Nominal Example 

Come back later to decide best language (probably just C++?)

---

## Structural Example

Come back later to decide best language (probably just Go?)

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

var _ Abser = (*Vector)(nil) // succeeds as a Vector implements Magnitude implicitly (it has a structure of a Magnitude)

type Box struct {
    Value float64
}

// var _ Abser = (*Box)(nil) // won't succeed as a Box is not an Abser implicitly (it doesn't have the structure of an Abser)
```

[Godbolt](https://godbolt.org/z/4ohxYc3Gh)

A C++ example with C++20 concepts is possible, should it be used?

```cpp
#include <concepts>

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

static_assert(Absable<Vector>); // succeeds as Vector has the abs method, dictated by the Absable concept

struct Box {
    double value;
};

// static_assert(Absable<Box>); // fails as Box does not have the abs method, dictated by the Absable concept
```

[Godbolt](https://godbolt.org/z/cEPb3Y89K)

---

## Duck Typing

If it walks like a duck and quacks like a duck, then it must be a duck! 

| Structural Typing                                                            |                                                             Duck Typing |
| :--------------------------------------------------------------------------- | ----------------------------------------------------------------------: |
| Checks for structural equivalence at <i style="color:#22d;">compile time</i> | Checks for structural equivalence at <i style="color:#22d;">runtime</i> |

---

## Duck Typing Example

Python?

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