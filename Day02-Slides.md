---
theme: css/my-theme.css
---
## Type Systems Day 2
### By: Jaran Chao and John Whiting

---

## Overview

### Advanced properties of type systems

- Language Safety
- Flow sensitive
- Intersection
- Refinement
- Dependent
- Gradual
- Substructural

---

## Language Safety

Type systems allow us to "prove" certain aspects of our programs are correct. Each of the covered type systems allow us to "prove" different aspects of our programs to ensure that they never fail.

---

## Flow Sensitive

The type is made more specific with control flow

Done by validating operations like type predicates, imperative updates, and control flow. <!-- element style="font-size: 1.5rem;" -->

---

## Flow Sensitive (Kotlin)

```kotlin
sealed interface Node {}

data class IntNode(val value: Int) : Node
data class NegNode(val node: Node) : Node
data class AddNode(val lhs: Node, val rhs: Node) : Node
data class MulNode(val lhs: Node, val rhs: Node) : Node

fun eval(n: Node): Int {
    return when (n) {
        is IntNode -> n.value
        is NegNode -> -eval(n.node)
        is AddNode -> eval(n.lhs) + eval(n.rhs)
        is MulNode -> eval(n.lhs) * eval(n.rhs)
    }
}

fun main() {
    println(eval(MulNode(
        NegNode(
            IntNode(3)
        ),
        AddNode(
            IntNode(4),
            IntNode(5),
        ),
    )))
}
```

[Kotlin Playground](https://pl.kotl.in/-_-SXizFK)

---

## Flow Sensitive (Rust)

Pattern matching is an alternative to flow sensitive

```rust
enum Node {
    Int(i32),
    Neg(Box<Node>),
    Add(Box<Node>, Box<Node>),
    Mul(Box<Node>, Box<Node>),
}

fn eval(n: Node) -> i32 {
    match n {
        Node::Int(i) => i,
        Node::Neg(node) => -eval(*node),
        Node::Add(lhs, rhs) => eval(*lhs) + eval(*rhs),
        Node::Mul(lhs, rhs) => eval(*lhs) * eval(*rhs),
    }
}

pub fn main() {
    println!("{}", eval(Node::Mul(
        Box::new(
            Node::Neg(
                Box::new(Node::Int(3)),
            ),
        ),
        Box::new(
            Node::Add(
                Box::new(Node::Int(4)),
                Box::new(Node::Int(5)),
            ),
        ),
    )));
}
```

[Godbolt](https://godbolt.org/z/dEh4cos9b)

---

## Flow Sensitive Example (TS)

```ts
interface User {
  __typename: "user",
  email: string,
};

interface Group {
  __typename: "group",
  name: string,
};

function stringify(obj: User | Group) {
  if (obj.__typename === "user") {
    return `User(${obj.email})`;
  }
  return `Group(${obj.name})`;
}

const userExample = {
  __typename: "user" as const,
  email: "johndoe@example.com",
};

const groupExample = {
  __typename: "group" as const,
  name: "My Group",
};

const roleExample = {
  __typename: "role" as const,
  accessLevel: 3,
}

console.log(stringify(userExample));
console.log(stringify(groupExample));
console.log(stringify(roleExample)); // Error: Not assignable to User | Group
```

[TS Playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChnID6hYAngA4QhwC2EAXMgEQCumUTANPshDXMAA2jdGCigA5twC+Ably5QkWIhQBxKAHsW5HD2JlK1OoyYStOrj2MNko8SCm45CmCxAIwwTSDtjJwDCkABSaAEYAVowYWAA+yBra5ACUegSByKGRAHQGFFS0KAC8Jcxs0EypeAQEUBBgLFC+AAYxUMEAJNjhEdl8AoLSyc3yBNIKtfWNLYk6nd05NkMjzgoIPqLI5VAAogAetOSCxWlEJPk2pttMyHDoyOsgotwE-UKmEZoAFiAAJpoQAACEAONCOEGy6xoVhcuEem3MSX2h2OyCKpzyRkKpkRllu93hYBeyEuzAAsqQEhZyDD5HCNmBkFpjsiwaj0dUzoYCiZmMyIDc7g8GcTEEh0OgADIQABuEGEyAAzDI1htNMdsoJNBJgvYAkFgttWeDksl5PD1RCtTq9Y5AiFceRjcdTfIAPRu4VPS2a7W6-x2g3850QV3ID3IHZQLRQRgAOU0jLu6GAEmoYVRYE0aHYyHis3IQA)

---

## Intersection

Result of combining two types without contradictions

Note that this is more like a union operation in set theory than an intersection operation. <!-- element style="font-size: 1.5rem;" -->
$$\{1, 2, 3\}\cup\{2, 3, 4\}=\{1, 2, 3, 4\}$$

---

## Intersection Example (TS)

```ts
interface Reader {
  email: string,
  readBooks: string[],
};

interface Author {
  name: string,
  publishedBooks: string[],
};

type AuthorWhoReads = Reader & Author;

function stringify(obj: AuthorWhoReads): string {
  return `
${obj.name} (${obj.email}) has published:
  - ${obj.publishedBooks.join("\n    - ")}

And has read:
  - ${obj.readBooks.join("\n    - ")}
`;
}

const reader = {
  email: "wshakespeare@example.com",
  readBooks: [
    "The Tragicall Historye of Romeus and Juliet",
    "Menaechmi",
  ]
};

const author = {
  name: "William Shakespeare",
  publishedBooks: [
    "Romeo and Juliet",
    "The Brothers Menaechmus",
  ],
};

const authorWhoReads = { ...reader, ...author };

console.log(stringify(authorWhoReads));
console.log(stringify(reader)); // Error: missing properties: name, publishedBooks
console.log(stringify(author)); // Error: missing properties: email, readBooks
```

[TS Playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgEoTgE2sg3gKGWQgFs5gAbALmQGcwpQBzAGkOSg0wCEB7XgNa0a9RiCYBtALpsAvgG58+UJFiIUAQQCuYABa8oediDgkIIhszZEADloBGFYLV0Qe-IRbGSZ+BUrAATxtNHX0oAHV9dCxaZABeNC4cADJkbT0DRXwYLRAEMGBeEDpLcWAYQIAKXnsAKxoM8KjeGMxaAEovZiMiTjAtKBKAA3wAElxauoA6EzNZZCqJqenScgpZDuRdODi7R2dXTCp2AFpkZfrp-acXNz5BWmm63lAqgCIAHRLkc-eO2RKDQgTDbXYcLgnIjnS4zThYB5CZ6vEAfb7Q5D-QHDbKAhDFegQrA4RIEIhrSg0d4AdxccAEEFoITgnAAAhAAB6mGwUCDTfEkd7WInuR40CTsIjvAAqrmQ0qgcCYwAQcAoFGQAAlnGADIEULwYGheGYtHE4CDkAApLROCBgIWSzEAWQgJggCF0JGAjqIUj82XxIEJcDCBgSvWQc3MmIilCcpmQAGUdgymRhOL7kDdDvcPMJkBKiFLUCaILxkBbQTa7Q7hVLZShuFBeHpoHFXe7PSQzVnfP58EGQ2HItEuHFSchptP4dgoCwp9PQ5lDAOh7xedMKLwmFVRMwKtVl80x7EOh1FAB6S-Ideb7e7-flSpVWfQc-yZDX5AAUSgLagGhvVoWgehsFsQigQpGRoaMFxzO5RSEfBvzvPkHz3MplRfY8DA-L8bz-ACgOcUDxGzCDoGggsKQoBdZ0RWggA)

---

## Intersection Example (TS)

Contradicting types cannot be combined

```ts
interface Foo {
    bar: number,
};

interface Baz {
    bar: string,
};

type Qux = Foo & Baz;

const test: Qux = {
    bar: true, // Error: type 'boolean' is not assignable to type 'never'
};
```

[TS Playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgGIHt3IN4ChkHIBGcUAXMiAK4C2R0ANLgL4DcuuoksiKAQnABeOfIRLlkAZzBRQAcyZsOYAJ4AHFAEUqAD2QBeNJmQAyZAMHtcCdCGnJI0itr2G8hYqQoyqEBsgB6AOQAUSgodAlVDWQAciJMABsIOBBY5GBJSnQwZDhJSWA5EDgiZIcsaJRYkAgAN2hYllYgA)

---

## Refinement

The type is made more specific based on a predicate

$$f : \mathbb{N}\rightarrow\{n\in\mathbb{N}\:\vert\:n\gt5\}$$

Note that this is restricted form of intersection types. <!-- element style="font-size: 1.5rem;" -->

---

## Refinement Example (TS)

```ts
interface User {
  email: string,
  accessLevel: number,
};

interface AuthorizedUser {
  authorizedEmail: string,
};

function authorize(user: User): AuthorizedUser {
  if (user.accessLevel < 5) {
    throw new Error(`${user.email} is not authorized! Level: ${user.accessLevel}.`);
  }
  return { authorizedEmail: user.email };
}

function doAdminThings(user: AuthorizedUser) {
  console.log(`${user.authorizedEmail} deleted all database tables.`);
}

  

const adminUser = {
  email: "admin@example.com",
  accessLevel: 5,
};

const authedAdminUser = authorize(adminUser);
doAdminThings(authedAdminUser);



const regularUser = {
  email: "regular@example.com",
  accessLevel: 2,
}

const authedRegularUser = authorize(regularUser); // Throws runtime error
doAdminThings(authedRegularUser); // Never gets called
```

[TS Playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgKoGdrIN4ChnIQC2cwANgFzLphSgDmANPsokuugDIQBuElyEAFciAI2jMAvgG5cuUJFiIUAQSFgAFgHs6ALwgATDFjwE467XsMBREuSo06IJrhlyYQkAjDAtIVhY6wPoAFEKYUFTGUACUVGqaQfpGETgswDDIYREAdGwQHNx8ZMgAPMgArDFpBASaUFoA7oIQzdZQDVAhAAYAJNjh0DnEpGSSyMDoglpgAYlWBgCEyEX8VP2DUHkI7Fy8-JI53TGyBJJyBFAQYEJQ-thzlsE2dgKbw6-Ibue4Hl4+fmQBi0KgMRFAABUNAx0NloPFAgtotVTMgEH50FoyBAcmQtPQehtcuZ5s8DLZRuMDPxroZWGQSgY4GA4KI4JhkCzRNj0EcTq45HJ0SAaKwwaBosgALw1QivKgAIjg4pAAAEIAAPOBEAAO2Jy6KICuYZh2BT2xSoFSkslwwtFJI0hlB4JAkpljqSEBCytdyNkwJdkOhzlhjudKv9grtGNmV3oQjIcCg7tlI3syAV8cTyfVWt1+sNxpY+UK+wEACYpELY49DAAlCAJpMp1IexHPELZlv+5AAej7yChDUaUygnh8RBQ0E6uEDKqhMJ9FgbTZzregJ37g4AcvsoMh6Ncpgg4AzDEA)

---

## Refinement Example (L-H)

```hs
{-@ type VectorEq a X = {v:Vector a | vlen v == vlen X} @-}

{-@ dotProduct :: (Num a) => x:Vector a -> VectorEq a x -> a @-}
dotProduct x y = sum [ (x ! i) * (y ! i) | i <- [0 .. n - 1]]
    where
        n = length x
```

[LiquidHaskell](https://ucsd-progsys.github.io/liquidhaskell/)

Refinement types commonly utilize SMT Solvers to prove refinements <!-- element style="font-size: 1.5rem;" -->

---

## Refinement Wrappers

Popular languages that have refinement capabilities.

- Rust: [Flux](https://github.com/flux-rs/flux) ([ACM Digital Library](https://dl.acm.org/doi/10.1145/3591283))
- TypeScript: [refscript](https://github.com/UCSD-PL/refscript) ([ACM Digital Library](https://dl.acm.org/doi/10.1145/2908080.2908110))
- Scala: (v1+) [refined](https://github.com/fthomas/refined), (v3+) [Scala docs](https://scala-lang.org/files/archive/spec/3.4/03-types.html#refined-types)

---

## Dependent

The definition of the type is dependent on the type's value. Often seen in the theorem proving languages.

<table style="margin: 0;">
    <tbody>
        <tr>
            <td><a href="https://wiki.portal.chalmers.se/agda/pmwiki.php">Agda</a></td>
            <td><a style="color: #d5be99;" href="https://coq.inria.fr/">Coq</a></td>
            <td><a style="color: #00f;" href="https://www.cs.bu.edu/~hwxi/atslangweb/">ATS</a></td>
            <td><a style="color: #ee131f;" href="https://www.fstar-lang.org/">F</a></td>
            <td><a style="color: #aa0000;" href="https://www.idris-lang.org/">Idris</a></td>
            <td><a href="https://lean-lang.org/">Lean</a></td>
        </tr>
    </tbody>
</table>

Done by predicate logic through type predicates, universal, and existential quantifiers. <!-- element style="font-size: 1.5rem;" -->

---

## Dependent vs Refinement

Dependent types expose a general logical language as powerful as higher order logic, whereas refinement types expose a very restricted logical language that SMT solvers can handle.

---

## Gradual

Some types are annotated and checked at compile time and some may be left untyped to be checked at runtime. This can be seen as a merging of static and dynamic typing.

---

## Some Type Theory Terminology

|    | *Top Type* [n]                                                                    |
| -: | --------------------------------------------------------------------------------- |
| 1. | A type that is the supertype of all other types.                                  |
| 2. | The type in which <i style="color:#4def4d;">all</i> variables can be assigned to. |

|    | *Bottom Type* [n]                                                                |
| -: | -------------------------------------------------------------------------------- |
| 1. | A type that is the subtype of all other types.                                   |
| 2. | The type in which <i style="color:#ef3535;">no</i> variables can be assigned to. |

---

## How does it work

A untyped expression is given the type dynamic, commonly denoted as `?`. This type acts as the top type and is cast-able down the type hierarchy to the bottom type. 

In practice, to achieve the cast-ability down to bottom type, expression of the dynamic type are assumed to support any operation. <!-- element style="font-size: 1.5rem;" -->

---

## Gradual Example (Python)

```python
def identity(x):
    return x

def square(num: int | float):
    return num * num
    
print(square(10))
print(identity(10))

print(square(15.5))
print(identity(15.5))

print(square("oops")) # TypeError: can't multiply sequence by non-int of type 'str'
print(identity("oops"))
```

[Godbolt](https://godbolt.org/z/jdqEs9Pfh)

---

## Gradual Example (C#)

```csharp
using System;

class Program
{
    static dynamic Square(dynamic num) => num * num;
    static void Main()
    {
        Console.WriteLine(Square(10));
        Console.WriteLine(Square(15.5));
        Console.WriteLine(Square("oops")); // Operator '*' cannot be applied to operands of type 'string' and 'string'
    }
}
```

[Godbolt](https://godbolt.org/z/vdqxT1893)

---

## Substructural

The types are constrained by the substructural rules from substructural logic. 

Put another way, the types are defined based on number of uses.

| Type     |                   Use |
| :------- | --------------------: |
| Ordered  | Exactly once in order |
| Linear   | Exactly once          |
| Affine   | At most once          |
| Relevant | At least once         |
| Normal   | Arbitrarily           |

---

## Affine Example (Rust)

```rust
struct Coin;
struct Candy;
struct Drink;

fn buy_candy(_: Coin) -> Candy { Candy }
fn buy_drink(_: Coin) -> Drink { Drink }

pub fn main() {
    let coin = Coin;
    let candy = buy_candy(coin); // The lifetime of the coin variable ends here.
    let drink = buy_drink(coin); // Compilation error: Use of moved variable that does not possess the Copy trait.
}
```

[Godbolt](https://godbolt.org/z/5EK94h9cr)

---

## Linear Example

Theoretical behavior in Rust but not actually enforced. Languages with Linear Types include [Vale](https://vale.dev/), [ATS](https://www.cs.bu.edu/~hwxi/atslangweb/), and [Austral](https://github.com/austral/austral/tree/master).

```rust
{
    // Must be passed on, not dropped.
    let token = HotPotato{};

    // Suppose not every branch does away with it:
    if (!queue.is_full()) {
        queue.push(token);
    }

    // Compilation error: Holding an undroppable object as the scope ends.
}
```

---

## Normal Example (C++)

```cpp
struct Coin {};
struct Candy {};
struct Drink {};

auto buy_candy(std::unique_ptr<Coin> c) { return Candy{}; }
auto buy_drink(std::unique_ptr<Coin> c) { return Drink{}; }

auto main() -> int {
    auto coin = std::make_unique<Coin>();
    auto candy = buy_candy(std::move(coin));
    auto drink = buy_drink(std::move(coin)); // This is valid C++.
}
```

[Godbolt](https://godbolt.org/z/q9avEv5cz)
