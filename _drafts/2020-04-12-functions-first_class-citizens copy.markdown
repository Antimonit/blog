---
layout: post
title:  "Replace interfaces with lambdas"
date:   2020-04-12 14:36:49 +0200
tags:   programming
---
<!-- 
Alternative titles
- Functions as first class citizens
- Replace callback interfaces with Function types
- Replace interfaces with lambdas
- Function types instead of interfaces
- Lambdas in place of interfaces
- Kotlin Function types
-->

<!-- ## Preface -->

Functions as first-class citizens is a powerful language feature commonly found in modern languages. It allows us to treat functions as variables, pass them as arguments to functions or return functions as a result of other functions.
It is not something unique to Kotlin --- support for lambdas, method references and functional interfaces was added already in Java 8 and before then it was available in dozens of other languages.
Virtually anything we can do in Kotlin regarding lambdas and higher-order functions is also possible in Java.

`java.util.function` package contains over 40 interfaces that describe various combinations of parameter types and return types. 
To give a general idea, following functions:
```java
void toVoid();
double toDouble();
void longToVoid(long other);
boolean longToBoolean(long other);
double longToDouble(long other);
double longLongToDouble(long other, long other2);
```
can be represented using the following functional interfaces:
```java
Runnable v = this::toVoid;
Supplier<Double> d = this::toDouble;
Consumer<Long> lv = this::longToVoid;
Predicate<Long> lb = this::longToBoolean;
Function<Long, Double> ld = this::longToDouble;
BiFunction<Long, Long, Double> lld = this::longLongToDouble;
```

<!-- 
`Runnable`, `Consumer`, `Supplier`, `Predicate`, `BiPredicate`, `UnaryOperator`, `BinaryOperator`, `Function`, `BiFunction`
-->

It might be just me, but I found it awkward to work with types like `Runnable`, `Consumer`, `Supplier`, `Predicate`, `UnaryOperator`, `BiFunction`, etc. and mentally translating what do they accept as parameter types and what are their return types. 
Not to mention all the specialized interfaces working with primitive types, such as `IntToDoubleFunction` or `ToDoubleBiFunction`. 
But the worst of all, these interfaces work with a maximum of two parameters -- if you have more than two, you had to create a custom interface.

## Function types

These shortcomings were alleviated in Kotlin by introduction of **Function Types** [<sup>[1]</sup>](https://kotlinlang.org/docs/reference/lambdas.html#function-types).
Kotlin uses a special notation that closely matches parameter and return types of the function, which I find highly intuitive.

<!-- 
```kotlin
val v: Function0<Unit> = ::toVoid
val lv: Function1<Long, Unit> = ::longToVoid
val lb: Function1<Long, Boolean> = ::longToBoolean
val d: Function0<Double> = ::toDouble
val ld: Function1<Long, Double> = ::longToDouble
val lld: Function2<Long, Long, Double> = ::longLongToDouble
``` 
-->
```kotlin
val v: () -> Unit = ::toVoid
val d: () -> Double = ::toDouble
val lv: (Long) -> Unit = ::longToVoid
val lb: (Long) -> Boolean = ::longToBoolean
val ld: (Long) -> Double = ::longToDouble
val lld: (Long, Long) -> Double = ::longLongToDouble
```

It also allows to conveniently define parameter names directly in the function type:
```kotlin
val saveFullName: (firstName: String, lastName: String) -> Boolean
```
receiver types:
```kotlin
val builder: Body.() -> Unit
```
functions producing other functions:
```kotlin
val funProducer: () -> () -> Unit
```
or even all of the above combined together
```kotlin
val graphIntialization: (size: Int) -> Node.() -> Boolean
```

## When to prefer lambdas over interfaces

A typical use of lambdas is as a replacement for Callback or Listener interfaces.

Consider the following example, where a Fragment passes a callback interface to a Controller to observe clicks on the items of the controller. 

```kotlin
class InboxFragment : Fragment() {

    val controller = InboxController(object : InboxController.Callbacks() {

        override fun onMessageClicked(messageId: Long) {
            startActivity(MessageDetailActivity.getIntent(context, messageId))
        }

        override fun onAddMessageClicked() {
            NewMessageDialog().show(childFragmentManager, null)
        }
    })
}

class InboxController(
    private val callbacks: Callbacks
) : EpoxyController() {

    override fun buildModels() {
        // create message models utlizing `callbacks.onMessageClicked`
        // create addMessage model utilizing `callbacks.onAddMessageClicked`
    }

    interface Callbacks {
        fun onMessageClicked(messageId: Long)
        fun onAddMessageClicked()
    }
}
```

In situation like this, if there is only a single boundary between **where a Listener is implemented** and **where it is consumed**, consider using lambdas instead.

<!-- 
Tradeoff between *having them decoupled* and *programming against an interface*.
 -->

```kotlin
class InboxFragment : Fragment() {

    val controller = InboxController(
        onMessageClicked = {
            startActivity(MessageDetailActivity.getIntent(context, it))
        },
        onAddMessageClicked = {
            NewMessageDialog().show(childFragmentManager, null)
        }
    )
}

class InboxController(
    private val onMessageClicked: (messageId: Long) -> Unit,
    private val onAddMessageClicked: () -> Unit
) : EpoxyController() {

    override fun buildModels() {
        // create message models utlizing `onMessageClicked`
        // create addMessage model utilizing `onAddMessageClicked`
    }
}
```

You might raise an objection and say, it is not **programming against an interface**. 
That is a very valid point but in surprisingly many situations an interface sits in between only two components.
In this example **the controller itself became the interface**.
Instead of making changes to the interface, we directly make changes to the controller.
From the outside perspective, we have to update the usage in either case.

<!--
Testing sucks?
How to mock lambdas? How to verify they were called?
With new method added to the listener, tests break too.
-->

<!-- Whenever you need to expand or otherwise make changes to the interface, you would have to update the interface definition and interface implementation.

Implementation using lambdas does not require additional code changes compared to implementation using interface. -->

## When to prefer interfaces over lambdas

Although lambdas can make the code more legible and concise without sacrificing any functionality, interfaces still have their place.

### When the interface is resolved dynamically at runtime
 
In any place where we refer to the interface through `is` or `as` checks.

This might
Bad design of the Android framework, but there is not much we can do about this.

This typically happens when we cannot pass a reference to the observer due to how Android framework works.

One such situation occurs when we need to 
```kotlin
saveButton.setOnClickListener {
    (parentFragment as? WordUpdatedListener
        ?: activity as? WordUpdatedListener)
        ?.onWordAdded(wordEditText.text.toString())
}
```

### When the interface spans more than two components

When there are two or more boundaries between *where the listener is implemented* and *where it is consumed*.
when the interface has to pass multiple layers. 

<!-- through some mediator -->


```kotlin
class InboxFragment : Fragment() {
    
    val controller = InboxController(
        onMessageClicked = viewModel::onMessageClicked
    )
}
```

```kotlin
class InboxFragment : Fragment() {
    
    val controller = InboxController(viewModel)
}
```

### When the interface has multiple consumers


## Data class 
```kotlin
fun login(username: String, password: String)
```

```kotlin
data class Credentials(val username: String, val password: String)

fun login(credentials: Credentials)
```

```kotlin
Animator(
    onStart: () -> Unit,
    onEnd: () -> Unit
)
```
```kotlin
interface Callbacks {
    fun onStart()
    fun onEnd()
}

Animator(callbacks: Callbacks)
```

<!-- ## Performance -->

<!-- At runtime it is the same, even the functional approach is compiled down to an anonymous class. -->

<!-- `InboxFragment$anonymous$controller(this)` -->
