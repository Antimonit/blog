---
layout: post
title:  "Algebraic Data Types"
date:   2023-02-12 18:32:00 +0900
tags:   programming
---

<style>
.admonition {
    border-width: 0.05rem;
    border-style: solid;
    border-radius: 0.4rem;
    display: flow-root;
    font-size: .875rem;
    margin: 1.5625em 0;
    padding: 1rem;
    page-break-inside: avoid;
}
.admonition > :last-child {
    margin-bottom: 0;
}
.hint {
    border-color: #448aff;
    background-color: #448aff1a;
}
.tip {
    border-color: #00c853;
    background-color: #00c8531a;
}
.note {
    border-color: #ff9100;
    background-color: #ff91001a;
}
</style>

**Algebraic Data Types** are the bread and butter of many functional languages. In languages such as
Haskell, there are no classes or inheritance. Haskell does not even have a `null` value. The absence
of a value is expressed through ADTs. You could probably even say that Haskell's type system _is_
ADTs and you would not be too far off.

In object-oriented languages, such as Kotlin, Swift, TypeScript, etc., we are not forced to design
our code using ADTs. We can still use `null`s wherever we want, use runtime condition checking, and
generally write our code in a very imperative C-style way. But using language features such as 
**sealed classes** and **exhaustive type checking**, we can replicate much of the ADTs' power even 
in non-functional languages.

Before proceeding to explain ADTs any further, let me first explain a related concept that we will 
use throughout the rest of this article.

## Type's size

Each type has a number of possible values it can represent. Let's refer to this number as the size
of a type. Here is a table of some primitive types and their sizes.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Primitive.drawio.svg" type="image/svg+xml"></object>
</p>

* The size of the `Boolean` type is **2** since it can represent only **true or false**.
* The size of the `Byte` type is **256** since it can represent all integer numbers ranging from
  **0 to 255**.
* Etc.

In mathematics, this attribute is generally called **cardinality**, but I never saw the term used in
the programming context, so I opted to using a more reader-friendly term.

Just be aware that "size" of a type in other contexts may refer to a different concept. In C, for
example, the `sizeof` operator is used to calculate how many bytes a type occupies in memory. The
cardinality of a type is much more abstract concept and is unrelated from the memory representation.
For the rest of the article, the size will always refer to the type's cardinality.

<div class="admonition hint" markdown="1">

`String` does have a theoretical limit, but it is so large that for any practical use, we can
pretend it is infinite.

On JVM, a `String` may hold up to [2<sup>31</sup>-1 characters][jvm-string-size], which brings us
to whopping (2<sup>8</sup>)<sup>2<sup>31</sup>-1</sup> = **256<sup>2147483647</sup>** possibilities.

In comparison, there are only 10<sup>82</sup> ≈ **256<sup>34</sup>** atoms in the observable
universe.

[jvm-string-size]: https://www.javatpoint.com/java-string-max-size

</div>

## Algebraic Data Types

ADTs are simply **types composed of other types**. This might be quite an anticlimactic revelation,
but there is a lot to unpack.
There are two fundamental ways how we can compose types together. These are represented by **Product
types** and **Sum types**.

<p align="center">
    <object data="/assets/images/algebraic_data_types/ADT.drawio.svg" type="image/svg+xml"></object>
</p>


<style type="text/css" rel="stylesheet">
.adt_columns_box {
  display: flex;
  gap: 20px;
}
.adt_columns_box>* {
  flex: 1 1 auto;
}
@media all and (max-width: 800px) {
  .adt_columns_box {
    flex-direction: column;
  }
}
</style>

### Sum types

<div class="adt_columns_box">
  <p align="center">
    <object data="/assets/images/algebraic_data_types/Sum.drawio.svg" type="image/svg+xml"></object>
  </p>

<div markdown="1">

Sum types are composed of components where only **a single component** is required to represent the
sum type.

These components have an **OR** relationship – to instantiate a sum type, we must specify one
component **OR** any other component.

The size of a `Direction` type is **4** since it can represent `up` or `down` or `left` or `right`
values.

The size of a `State` is 3 since it can represent **3** values—`initial`, `loading` or `loaded`.

</div>
</div>

### Product types

<div class="adt_columns_box">
  <p align="center">
    <object data="/assets/images/algebraic_data_types/Product.drawio.svg" type="image/svg+xml"></object>
  </p>

<div markdown="1">

Product types are composed of components where **all components** are required to represent the
product type.

These components have an **AND** relationship – to instantiate a product type, we must specify one
component **AND** all other components.

The size of a `Format` type is **8** since it can represent any of
the (⊤, ⊤, ⊤), (⊤, ⊤, ⊥), (⊤, ⊥, ⊤), (⊤, ⊥, ⊥), (⊥, ⊤, ⊤), (⊥, ⊤, ⊥), (⊥, ⊥, ⊤), (⊥, ⊥, ⊥)
combinations.

The size of a `Color` is **2<sup>24</sup>** since each color component can represent 2<sup>8</sup> values and all 3
color components are required at the same time.

</div>
</div>

## Abstract types

All the types we've shown so far were **concrete types** and have a fixed, constant size.

**Abstract types** (i.e. types parametrized by other types) fit into the ADT system as well.
Unlike concrete types, the size of an abstract type depends on other types, which may be
concrete but also abstract types.

<style type="text/css" rel="stylesheet">
.abstract_adt_columns_box {
  display: flex;
  gap: 20px;
}
.abstract_adt_columns_box>* {
  flex: 1 1 50%;
}
@media all and (max-width: 800px) {
  .abstract_adt_columns_box {
    flex-direction: column;
  }
}
</style>

<div class="abstract_adt_columns_box">

<div>
<div markdown="1">

### Abstract sum types
</div>
  <div>
    <object data="/assets/images/algebraic_data_types/Abstract Sum.drawio.svg" type="image/svg+xml"></object>
  </div>
</div>

<div>
<div markdown="1">

### Abstract product types
</div>
  <div>
    <object data="/assets/images/algebraic_data_types/Abstract Product.drawio.svg" type="image/svg+xml"></object>
  </div>
</div>

</div>

For example, the size of `Optional<T>` is **T+1** where **T** simply refers to the size of the type
**T**. This is similar to variables in mathematical equations.
We know now that the size of a `Boolean` is **2** and the size of a `Byte` is **256**,
thus `Optional<Boolean>` has a size of **3**, while `Optional<Byte>` has a size of **257**.

## Combination

Finally, we can also combine sum types and product types to represent other complex types.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Notification.drawio.svg" type="image/svg+xml"></object>
</p>

<div class="admonition tip" markdown="1">

Although primitive types are typically not regarded as ADTs, if you squint your eyes enough, they
can be seen as a form of Sum and Product types as well.

They can be regarded as **Sum types**, each simply enumerating their values:

<div>
  <object data="/assets/images/algebraic_data_types/Primitive Sum.drawio.svg" type="image/svg+xml"></object>
</div>

Or alternatively as **Product types**, representing how they are physically stored in memory:

<div>
  <object data="/assets/images/algebraic_data_types/Primitive Product.drawio.svg" type="image/svg+xml"></object>
</div>

</div>

## Relation with algebra

**Sum types** and **Product types** are called **Algebraic Data Types** because
they exhibit properties similar to [**Algebraic structures**][Algebraic Structures].
You might remember some of these from school:

* [(ℤ, +)][Z+] Integers under addition form an [abelian group].
* [(ℤ, ×)][Z×] Integers under multiplication form a [semigroup].
* [(ℝ, +)][R+] Real numbers under addition form an [abelian group].
* [(ℝ∖{0}, ×)][Z×] Real numbers without zero under multiplication form an [abelian group].
* [(ℝ, +, ×)][R+×] Real numbers under addition and multiplication form a [ring].
* [(ℕ, +, ×)][N+×] Natural numbers under addition and multiplication form a [commutative monoid].

[Z+]: https://proofwiki.org/wiki/Additive_Group_of_Integers_is_Countably_Infinite_Abelian_Group

[Z×]: https://proofwiki.org/wiki/Integers_under_Multiplication_form_Semigroup

[R+]: https://proofwiki.org/wiki/Real_Numbers_under_Addition_form_Infinite_Abelian_Group

[Z×]: https://proofwiki.org/wiki/Non-Zero_Real_Numbers_under_Multiplication_form_Abelian_Group

[R+×]: https://proofwiki.org/wiki/Real_Numbers_form_Ring

[N+×]: https://proofwiki.org/wiki/Natural_Numbers_under_Addition_form_Commutative_Monoid

[abelian group]: https://proofwiki.org/wiki/Definition:Abelian_Group

[semigroup]: https://proofwiki.org/wiki/Definition:Semigroup

[ring]: https://proofwiki.org/wiki/Definition:Ring_(Abstract_Algebra)

[commutative monoid]: https://proofwiki.org/wiki/Definition:Commutative_Monoid


Algebraic structures are not limited only to infinite numbers though:

* The [Rubik's cube][rubik's cube] as well can be defined as a [group][group], commonly known as
  the [Rubik's group].
* [Quaternions][quaternions] form a cyclic [group] known as
  the [quaternion group][quaternion group].
* Even a simple clock can be [described as a group][clock group].

[rubik's cube]: https://mathworld.wolfram.com/RubiksCube.html

[group]: https://mathworld.wolfram.com/Group.html

[rubik's group]: https://mathworld.wolfram.com/RubiksGroup.html

[quaternions]: https://mathworld.wolfram.com/Quaternion.html

[quaternion group]: https://mathworld.wolfram.com/QuaternionGroup.html

[clock group]: https://plus.maths.org/content/maths-a-minute-groups

[Algebraic Structures]: https://en.wikipedia.org/wiki/Algebraic_structure

Although we cannot say that ADTs _are_ an algebraic structure, if we bend the
rules a little, use **sum types** to represent the + operation, and use
**product types** to represent the × operation, we can observe that ADTs in many
ways behave like an algebraic structure!

## Isomorphism

For example, the concept of [isomorphism] (as applied to [graphs][graph isomorphism]
or [groups][group isomorphism], for example) applies to ADTs as well.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Graph Isomorphism.drawio.svg" type="image/svg+xml"></object>
</p>

Two types (graphs, groups, ...) are [isomorphic] if there is a [bijection] between the two types 
(graphs, groups, ...). In other words, there has to be a **1:1 mapping** between the elements of one
and the elements of the other.

What this means in practice is that we can, for example, substitute a **Sum type** composed of two
elements with a **Boolean type**.

<p align="center">
  <object data="/assets/images/algebraic_data_types/State Isomorphism.drawio.svg" type="image/svg+xml"></object>
</p>

Of course, **State** and **Boolean** types are not equal. Passing one into a function that expects
the other one would yield a compilation error. But we can create functions that map one type to
another without losing any information along the way.

[isomorphism]: https://mathworld.wolfram.com/Isomorphism.html

[graph isomorphism]: https://mathworld.wolfram.com/GraphIsomorphism.html

[group isomorphism]: https://mathworld.wolfram.com/IsomorphicGroups.html

[isomorphic]: https://mathworld.wolfram.com/Isomorphic.html

[bijection]: https://mathworld.wolfram.com/Bijection.html

### Associativity, commutativity, distributivity, identity

Understanding isomorphism between types allows us to apply other algebraic properties to ADTs as
well.

<style type="text/css" rel="stylesheet">
.algebraic_properties_box {
  display: flex;
  gap: 10px;
  margin: 2rem 1rem
}
.algebraic_properties_box>* {
  flex: 1 1 auto;
}
@media all and (max-width: 800px) {
  .algebraic_properties_box {
    flex-direction: column;
  }
}
</style>

<div class="algebraic_properties_box">
<div>
<div markdown="1">

**Additive associativity**

(a + b) + c = a + (b + c)
</div>
<div markdown="1">

**Additive commutativity**

a + b = b + a
</div>
<div markdown="1">

**Additive identity**

0 + a = a + 0 = a
</div>
</div>

<div>
<div markdown="1">

**Multiplicative associativity**

(a × b) × c = a × (b × c)
</div>
<div markdown="1">

**Multiplicative commutativity**

a × b = b × a
</div>
<div markdown="1">

**Multiplicative identity**

1 × a = a × 1 = a
</div>
</div>

<div>
<div markdown="1">

**Left and right distributivity**

a × (b + c) = (a × b) + (a × c)

(b + c) × a = (b × a) + (c × a)
</div>

<div markdown="1">

**Zero element (?)**

a × 0 = 0 × a = 0
</div>
</div>
</div>

For example, the **multiplicative associativity** rule tells us that the following two types are
isomorphic.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Pair Pair Isomorphism.drawio.svg" type="image/svg+xml"></object>
</p>

Indeed, with a bit of refactoring, one type can be replaced with the other one since both types have
the same size and represent the same amount of possibilities.

But what about the **additive identity** and **multiplicative identity** rules? How can we represent
a type with a **size of 1** or even a **size of 0**?

### Any, Unit, and Nothing

Compared to Java, where the `Void` type was more of a keyword rather than an actual type, 
Kotlin's `Unit` and `Nothing` types form a much sounder type system.

* `Any` can be literally anything and thus has a size of **∞**.
* `Unit` is a singleton and has a size of **1**.
* `Nothing` cannot be initialized and has a size of **0**.

These types fit very nicely into the algebra of ADTs!

* The `Unit` type can be used to represent the multiplicative identity.
* The `Nothing` type can be used to represent the additive identity.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Identities.drawio.svg" type="image/svg+xml"></object>
</p>

<div class="admonition tip" markdown="1">

The same concept can be found in many other languages as well. These types are commonly referred to
as the [top][type-top], [unit][type-unit] and [bottom][type-bottom] types. 

In languages that support subtyping, roughly speaking, the top type is the supertype of all other
types and the bottom type is the subtype of all other types (ignoring nullability and primitive
types for simplicity).

|            |      [top][type-top]      |              [unit][type-unit]               |   [bottom][type-bottom]    |
|:----------:|:-------------------------:|:--------------------------------------------:|:--------------------------:|
|   Kotlin   |     [Any][kotlin-top]     |             [Unit][kotlin-unit]              |  [Nothing][kotlin-bottom]  |
|   Scala    |     [Any][scala-top]      |              [Unit][scala-unit]              |  [Nothing][scala-bottom]   |
|   Swift    |  [AnyObject][swift-top]   |              [Void][swift-unit]              |   [Never][swift-bottom]    |
| TypeScript | [unknown][typescript-top] | [null][typescript-unit][^typescript-details] | [never][typescript-bottom] |
|    Rust    |      [Any][rust-top]      |               [()][rust-unit]                |      [!][rust-bottom]      |
|  Haskell   |             ❌             |              [()][haskell-unit]              |   [Void][haskell-bottom]   |
|    Dart    |    [Object][dart-top]     |                      ❌                       |    [Never][dart-bottom]    |
|    Java    |    [Object][java-top]     |                      ❌                       |             ❌              |
|    C++     |    [std::any][cpp-top]    |                      ❌                       |             ❌              |

[type-top]: https://en.wikipedia.org/wiki/Top_type
[type-unit]: https://en.wikipedia.org/wiki/Unit_type
[type-bottom]: https://en.wikipedia.org/wiki/Bottom_type

[kotlin-top]: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/
[kotlin-unit]: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-unit/
[kotlin-bottom]: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-nothing.html

[scala-top]: https://scala-lang.org/api/3.x/scala/Any.html
[scala-unit]: https://scala-lang.org/api/3.x/scala/Unit.html
[scala-bottom]: https://scala-lang.org/api/3.x/scala/Nothing.html

[swift-top]: https://developer.apple.com/documentation/swift/anyobject
[swift-unit]: https://developer.apple.com/documentation/swift/void
[swift-bottom]: https://developer.apple.com/documentation/swift/never

[typescript-top]: https://www.typescriptlang.org/docs/handbook/2/functions.html#unknown
[typescript-unit]: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#null-and-undefined
[typescript-bottom]: https://www.typescriptlang.org/docs/handbook/2/functions.html#never
[typescript-type-system]: https://gist.github.com/laughinghan/31e02b3f3b79a4b1d58138beff1a2a89
[^typescript-details]: TypeScript's type system is [complicated][typescript-type-system].

[rust-top]: https://doc.rust-lang.org/std/any/trait.Any.html
[rust-unit]: https://doc.rust-lang.org/std/primitive.unit.html
[rust-bottom]: https://doc.rust-lang.org/std/primitive.never.html

[haskell-unit]: https://hackage.haskell.org/package/ghc-prim/docs/GHC-Tuple.html
[haskell-bottom]: https://hackage.haskell.org/package/base/docs/Data-Void.html
[haskell-boring]: https://hackage.haskell.org/package/boring

[dart-top]: https://api.flutter.dev/flutter/dart-core/Object-class.html
[dart-bottom]: https://dart.dev/null-safety/understanding-null-safety#top-and-bottom

[java-top]: https://docs.oracle.com/en/java/javase/19/docs/api/java.base/java/lang/Object.html

[cpp-top]: https://en.cppreference.com/w/cpp/utility/any

[//]: # (https://wiki.haskell.org/Bottom)
[//]: # (https://softwareengineering.stackexchange.com/a/277271)

</div>

### Nullable types

The use of optional types is so ubiquitous that most modern languages offer a succinct syntax to
represent them, usually by appending an `?` to the type.

Effectively, `Optional<T> ≅ T?`.

<div class="admonition tip" markdown="1">

There is an interesting relationship between **nullable** types and the `Nothing` type.

As we already know, the `Nothing` type represents an uninstantiable value and has a **size of 0**.
There is simply no value we can assign to `val result: Nothing`.

But the `Nothing?` type has a **size of 1+0=1**. As it turns out, there is exactly one value we can
assign to `val result: Nothing?`—the `null` value.

</div>

<div class="admonition note" markdown="1">

Kotlin's **nullable** types and `Optional` types are not strictly isomorphic.

While we can declare a type such as `Optional<Optional<T>>` with a size of **2 + T**, in
Kotlin `T??` effectively becomes `T?`. There is no way to differentiate between `None`
and `Some(None)` since both map to `null`.

</div>

### More isomorphisms

Now that we learned that the `Unit` type has a **size of 1** and the `Nothing` type has a **size of
0**, let's define a few more isomorphisms.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Type Isomorphisms.drawio.svg" type="image/svg+xml"></object>
</p>

The list of examples here is not exhaustive as there are infinitely many more isomorphisms.

<div class="admonition hint" markdown="1">

In mathematical terms, ADTs, when expressed as their size, represent a [semiring][wolfram semiring].

ADTs do not qualify as a [ring][wolfram ring] since they are missing the [additive inverse]
and [multiplicative inverse] properties, but they satisfy other properties not required for a
semiring such as [additive identity], [multiplicative identity], and [multiplicative commutativity].

</div>

[wolfram semiring]: https://mathworld.wolfram.com/Semiring.html

[wolfram ring]: https://mathworld.wolfram.com/Ring.html

[additive inverse]: https://mathworld.wolfram.com/AdditiveInverse.html

[multiplicative inverse]: https://mathworld.wolfram.com/MultiplicativeInverse.html

[additive identity]: https://mathworld.wolfram.com/AdditiveIdentity.html

[multiplicative identity]: https://mathworld.wolfram.com/MultiplicativeIdentity.html

[multiplicative commutativity]: https://mathworld.wolfram.com/MultiplicativeCommutativity.html

## Why do we care?

Although all this theory is without a doubt very interesting, where can we actually put it to use?

It turns out that ADTs are great for modeling the domain layer in a way that makes it impossible to
represent illegal states, effectively removing dead code for cases when such an illegal state would
be actually reached.

Furthermore, sum types can be narrowed down to one of their possible values. This can be used to
express runtime checks at the type level, essentially exalting certain runtime checks to compile
time checks.

### Making illegal states unrepresentable

#### Network responses

It is not uncommon to represent a network response with a structure similar to the following.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Response Naive.drawio.svg" type="image/svg+xml"></object>
</p>

Every network response will always contain `code` and `message` meanwhile `exception` and `data`
fields are nullable. For clarity, we can replace the nullable types with an `Optional` type since
they are isomorphic.

The problem with this design is that not all possible values of `Response` are valid. From the
type-system perspective, it's possible to instantiate a `Response` with both `exception` AND `data`
fields empty (or, in a similar fashion, to have both `exception` AND `data` fields populated). We
know that this situation should never occur, so why don't we reflect that information in our type
system?

We need to reflect the exclusivity of `data` or `exception` in our type, which can be easily
achieved using a sum type. But what about the other two pieces of data—`code` and `message`? We can
model this either as a **sum** of **product** types or a **product** of **sum** types.

<p align="center">
  <object data="/assets/images/algebraic_data_types/Response ADT.drawio.svg" type="image/svg+xml"></object>
</p>

By applying the **distributive property**, we can actually convert one into the other. Both types
are isomorphic. In the end, which one to use comes down to personal preference and convenience.

<div class="admonition tip" markdown="1">

We can even prove the isomorphic inequality between the original and new types!

<pre align="center">
c×m×(d+1)×(e+1) ≟ (c×m×d)+(c×m×e)
c×m×(d×e+d+e+1) ≟  c×m×(d+e)     
     d×e+d+e+1  ≟       d+e      
     d×e    +1  ≠       0        
</pre>

From the **d×e+1 ≠ 0** inequality we can conclude the two types are not isomorphic.
It also gives us a hint about what is the difference between the two types.

* The left side of the equation has two terms left—**d×e** and **1**—which loosely translates 
  to a state where both `data` and `exception` are present or a state where neither `data` nor
  `exception` is present.
* The right side of the equation has no terms left, meaning there are no additional states that
  can not be represented by the original state.
</div>

#### TextFieldState

As a more complex example, imagine that we are implementing a custom `TextField` widget.
We got a rough list of requirements what the widget should be able to do, and so we reflected them 
in a `TextFieldState` type.

<p align="center">
  <object data="/assets/images/algebraic_data_types/TextFieldState Naive.drawio.svg" type="image/svg+xml"></object>
</p>

Unfortunately, some combinations of these attributes above are not valid.

* The `cursorColor` is not used when `isEditable = false`.
* The `isStrikethrough` is not used when `isEditable = true`.
* The `isMasked` is not used when `isEditable = true`.
* The `isMasked = true` is not valid together with `isStrikethrough = true`.
* The `mask` is not used when `isMasked = false` (and honestly, it makes little to no sense 
  to mask the text when `isEditable = true`).

Can we do better?

<p align="center">
  <object data="/assets/images/algebraic_data_types/TextFieldState ADT.drawio.svg" type="image/svg+xml"></object>
</p>

Notice that wherever we previously had a Boolean (`isEditable`, `isStrikethrough`, `isMasked`), now
the same information is expressed on the level of types (`Editable` or `Static`, `Normal`
or `Strikethrough` or `Masked`).

By expressing this information on the type level, we can leverage the type safety guaranteed by the
compiler and convert some runtime checks into compile-time checks!

Furthermore, by removing the senseless combinations, we potentially saved ourselves from writing
essentially dead code. In the best case, illegal combinations do not require any special treatment,
and we do not have to write any additional code. However, more often than not, we have to address
these conflicting requirements. In the better case, we throw an exception, delegating the problem
closer to its root. In the worse case, we may try to recover from the illegal state ourselves.
That, however, often leads to even more inconsistencies soon after and potentially even more
unnecessary code.

### Type narrowing

To better explain the power of type narrowing, imagine we are implementing a platform where users
review restaurants and give them ratings on a scale from 1 to 5 stars. When viewing details about
a restaurant, we want to display the average rating given by the users. That could be as simple as
collecting all the ratings in a `List<Int>` and calculating the `average()` on it.

```kotlin
@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    Text(ratings.average().toString())
}
```

While some operations, such as `list.sum()`, are safe to execute on an empty collection; other operations, such as `list.average()`, are not. If a list is empty, calculating `list.average()` will result in a runtime error since we attempt to divide by zero when calculating list.sum() / list.size. We should check whether the collection is empty first.

```kotlin
@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    if (ratings.isNotEmpty()) {
        Text(ratings.average().toString())
    } else {
        Icon(Icons.Default.QuestionMark)
    }
}
```
This is a trivial example and in reality, this logic could be split across multiple functions, classes, or even modules. Let's extract the formatting into a separate function.

```kotlin
fun formatAverageRating(ratings: List<Int>): String =
    String.format("%.1f ★", ratings.average())
 
@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    if (ratings.isNotEmpty()) {
        Text(formatAverageRating(ratings))
    } else {
        Icon(Icons.Default.QuestionMark)
    }
}
```

By extracting this logic, we have unfortunately introduced another problem.
Although the `Restaurant` function checks the contents of `ratings`, we opened the possibility 
for other components to call `formatAverageRating` with an empty list.
Every consumer of the `formatAverageRating` function should check also whether the passed `ratings`
parameter is not empty.
Alternatively, we can add an `isNotEmpty()` check inside of `formatAverageRating()` too, 
_"just to be safe"_, but what value should we return then?

This situation gets worse and worse as we make function call stack deeper and deeper and the gap
between validation and execution gets further apart.

```kotlin
fun formatAverageRating(ratings: List<Int>): String {
    ...
    scale(..., ratings)
    ...
}
 
fun scale(..., ratings: List<Int>): String {
    ...
    rotate(ratings)
    ...
}
 
fun rotate(..., ratings: List<Int>): String {
    ...
    translate(..., ratings)
    ...
}
 
fun translate(..., ratings: List<Int>): String {
    ...
    ratings.average()
    ...
}
 
@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    if (ratings.isNotEmpty()) {
        Text(formatAverageRating(ratings))
    } else {
        Icon(Icons.Default.QuestionMark)
    }
}
```

The problem is that the validation (e.g. `ratings.isNotEmpty()`) and execution of the validated
data (e.g. `ratings.average()`) is separated by layers of other functions and this "unwritten
contract" is soon lost in heaps of code.

In any bigger codebase, this is obviously unsustainable. It also often leads to adding more
validations in the intermediate steps _"just to be safe"_ even though there is no good way to
resolve the edge case.

#### Option 1: Just rethrow the exception

One solution would be to simply rethrow the exception, but this is what we wanted to prevent in the
first place. Although Kotlin does not enforce checked exceptions, we still can, at the very least,
add the `@Throws` annotation as a hint that the function is "dangerous".

```kotlin
@Throws
fun formatAverageRating(ratings: List<Int>): String
@Throws
fun scale(ratings: List<Int>): String
@Throws
fun rotate(ratings: List<Int>): String
@Throws
fun translate(ratings: List<Int>): String =
    ratings.average()

@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    try {
        Text(formatAverageRating(ratings))
    } catch (ex: IllegalArgumentException) {
        Icon(Icons.Default.QuestionMark)
    }
}
```

#### Option 2: Return a Sum type

Another solution would be to return a widened type representing either a success or a failure.
This can be modeled using a Sum type, such as `Either<Failure, String>` or `Validated<String>`.

```kotlin
fun formatAverageRating(ratings: List<Int>): Validated<String>
fun scale(ratings: List<Int>): Validated<String>
fun rotate(ratings: List<Int>): Validated<String>
fun translate(ratings: List<Int>): Validated<String> =
    if (ratings.isEmpty()) {
        Invalid
    } else {
        Valid(ratings.average())
    }

@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    when (val rating = formatAverageRating(ratings)) {
        is Valid -> Text(rating.value)
        is Invalid -> Icon(Icons.Default.QuestionMark)
    }
}
```

This is basically the same as checked exceptions. 
This feels very monadic because instead of operating on plain `String` values, we are required to
manipulate the `Validated` result in intermediate steps using `map`, `flatMap`, `fold`, and similar
operations, which may feel clunky at times.

Whether it is better to represent illegal states using exceptions or monads is an endless topic on 
its own, so we will not delve into the differences any further.

#### Option 3: Pass in a narrowed type

Remember that this whole ordeal is caused by the `list.average()` operation not being safe when
invoked on an **empty list**. Even though we check for `list.isEmpty()`, the result of this check is
not reflected in the type system and the information cannot be safely propagated throughout other
functions. So why don't we create a type representing a **non-empty list**?

We can make use of the fact that any implementation of a `List` is isomorphic to a `LinkedList`.
`LinkedList` is simply a chain of `Node`s that each hold a _value_ and reference to the _next_ 
`Node` or an `Empty` object, symbolizing the end of the chain.

<p align="center">
  <object data="/assets/images/algebraic_data_types/LinkedList.drawio.svg" type="image/svg+xml"></object>
</p>

In ADT terms, `LinkedList` is a Sum type of either `Empty` or `Node` types. Since there are only two
types to choose from, the `Node` type represents a list that is **not** `Empty`—a
**non-empty list**. That's exactly what we are after!
Let's rename the `Node` to a `NonEmptyList` and represent the `Empty` state using an `Optional<T>`.

<p align="center">
  <object data="/assets/images/algebraic_data_types/NonEmptyList.drawio.svg" type="image/svg+xml"></object>
</p>

This structure is still isomorphic to a `List` but allows us to narrow the type to a `NonEmptyList`
when we check for `list.isEmpty()`.

```kotlin
class NonEmptyList<T>(
    val head: T,
    val tail: List<T>,
) : List<T> by listOf(head) + tail

fun <T> List<T>.toNonEmptyListOrNull(): NonEmptyList<T>? =
    if (isEmpty()) null else NonEmptyList(first(), drop(1))

fun NonEmptyList<Int>.safeAverage(): Int =
    sum() / size // Can not divide by zero
```

```kotlin
fun formatAverageRating(ratings: NonEmptyList<Int>): String
fun scale(ratings: NonEmptyList<Int>): String
fun rotate(ratings: NonEmptyList<Int>): String
fun translate(ratings: NonEmptyList<Int>): String =
    ratings.safeAverage()

@Composable
fun Restaurant(..., ratings: List<Int>) {
    ...
    val nonEmptyRatings: NonEmptyList<T>? = ratings.toNonEmptyListOrNull()
    if (nonEmptyRatings != null) {
        Text(formatAverageRating(nonEmptyRatings))
    } else {
        Icon(Icons.Default.QuestionMark)
    }
}
```

The crucial part of this example is the isomorphic transformation of the `List<T>` type to the
`NonEmptyList<T>?` type. Here, the `null` value represents an empty list, and we can narrow down the
type to a non-empty list with a simple null check.

The `NonEmptyList` type expresses the `ratings.isEmpty() == false` check on the level of types. All
functions that transitively depend on the `ratings.average()` call now accept a `NonEmptyList`
instead of a `List`. This way, we completely removed the dredged edge case scenario from all the
functions.

<div class="admonition tip" markdown="1">

Kotlin has an idiomatic way of representing results of potentially problematic operations using 
nullable types, e.g.:
* `fun String.toIntOrNull(): Int?`
* `fun List<T>.firstOrNull(): T?`
* `fun List<T>.toNonEmptyListOrNull(): NonEmptyList<T>?`

This works exceptionally well with the Kotlin compiler which automatically refines
[null-checked][kotlin-null-checks] and [type-checked][kotlin-type-checks] types using 
[smart casting][kotlin-smart-cast] and provides a succinct 
[null coalescing operator][kotlin-null-coalescing-operator] (a.k.a. Elvis operator) and
[safe calls][kotlin-safe-calls] on nullable receivers.

[kotlin-null-coalescing-operator]: https://kotlinlang.org/docs/null-safety.html#elvis-operator
[kotlin-safe-calls]: https://kotlinlang.org/docs/null-safety.html#safe-calls
[kotlin-null-checks]: https://kotlinlang.org/docs/null-safety.html#checking-for-null-in-conditions
[kotlin-type-checks]: https://kotlinlang.org/docs/typecasts.html#is-and-is-operators
[kotlin-smart-cast]: https://kotlinlang.org/docs/typecasts.html#smart-casts

</div>

## Conclusion

Algebraic data types are a foundation for a strong type system that goes beyond primitive number 
types and strings. By expressing business constraints on the level of types, we can convert some
runtime checks into compile-time checks. They allow us to reason about the program more easily 
and with more confidence since the compilation of the program can be seen as a form of a unit test.

A strong type system allows us to build applications where not only we guaranteed that the business
logic is valid but also guards us against inadvertently breaking type invariants when making changes
to the application. 

## Further reading

### Functional types

[Functional types][function_types], such as `(A) -> B`, can also be regarded as Algebraic Data
Types with an exponential cardinality of **B<sup>A</sup>**. They fit into the algebra described
above just like one would expect from exponentiation. I intentionally left it out since I could
not find a practical example where the knowledge of this type's existence could be used to write
better programs.
Also, the article was getting too long.

### Howard-Curry Correspondence

[Howard-Curry Correspondence][howard_curry_correspondence] is a fascinating observation that there
is an isomorphism between computer programs and mathematical proofs. In other words, we can convert
programs to mathematical proofs but also express mathematical proofs as programs. This, however,
requires the programming language to have a strong type system.

At the heart of Howard-Curry Correspondence
* programs correspond to proofs,
* types correspond to formulae, 
* sum types correspond to disjunction, 
* product types correspond to conjunction,
* function types correspond to implication,
* the unit (`Unit`) type corresponds to truth and
* the bottom (`Nothing`) type corresponds to falsity.

Through more complex structures we can also express negation, universal quantification, and
existential quantification.
Through Howard-Curry Correspondence, the compilation of a program using a type system to represent
formulae we aim to prove serves as a formal proof (or lack thereof) that the formulae indeed hold.

[function_types]: https://en.wikipedia.org/wiki/Function_type
[howard_curry_correspondence]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence
