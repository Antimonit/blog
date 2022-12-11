---
layout: post
title:  "Misconceptions about visibility modifiers"
date:   2022-12-11 11:21:00 +0900
tags:   programming
---

Imagine that you develop a commercial library.
Among other features, the library should allow consumers to embed a video player
into their apps. But as an ads-driven company, you want to ensure ads
are properly displayed in embedded video players as well.
The library should not provide a way for consumers to tinker with internals,
such as when and how ads are displayed, since consumers would most likely just
disable them, causing you to lose revenue.

A natural step would be to limit the visibility of the sensitive code but that
alone is not enough.

Both Java's *package-private* modifier and Kotlin's *internal* modifier provide
a false sense of encapsulation and should not be used as a means to hide
sensitive code since they do not really protect the code from being accessed
by code outside of the specified scope.

## Visibility modifiers recap

Let's go through the basic theory, just for the sake of completeness.

Our goal is to protect the `AdsManager` class from being publicly accessed by
consumers.

```text
Library
╰── src/main/java
    ╰── library
        ├── internal
        │   ├── AdsManager.java
        │   ╰── Utils.java
        ╰── Api.java
```

|              | public | package-private | private |
|-------------:|:------:|:---------------:|:-------:|
| `AdsManager` |   ✅    |        ✅        |    ✅    |
|      `Utils` |   ✅    |        ✅        |    ❌    |
|        `Api` |   ✅    |        ❌        |    ❌    |

It should not surprise anyone that the `AdsManager` itself can access all of its
*private*, *package-private*, and *public* methods, regardless of whether they
are declared *static* or not.

Similarly, it should be no surprise that other classes in the same package
can access *package-private* methods and that no other classes can access
*private* methods.

So far, everything seems to work as expected.

## Reflection

Even the *private* modifier—the most restrictive modifier—can be circumvented
through the use of reflection, and the same technique can be used for other
modifiers in the same way.

```java
Method method = AdsManager.class.getDeclaredMethod("packagePrivateVisibility");
method.setAccessible(true);
method.invoke(null);
```

Let's put reflection aside. It is very brittle in nature, requires great care,
and, in reality, we don't need to use reflection to get around
*package-private* and *internal* modifiers.

## Java's package-private visibility

Problems arise in a multi-module environment, including any projects that use
external libraries.
As hinted previously, since our `AdsManager` class is *package-private*, we
might assume that no code but ours can access it. Unfortunately, that's not
the case.

```text
Consumer
╰── src/main/java
    ╰── library
        ╰── internal
            ╰── Rogue.java
```

```java
package library.internal;

class Rogue {
    Rogue() {
        AdsManager.publicVisibility(); // ✅
        // Not even a warning
        AdsManager.packagePrivateVisibility(); // ✅
        // AdsManager.privateVisibility(); // ❌
    }
}
```

<!-- package instruction? feature? declaration? orientation?  -->

By replicating the package hierarchy and placing a `Rogue` class there, it
gained access to all *package-private* declarations of `AdsManager`, even though
it is declared in a different module!
This is called a **package split**, and it breaks the *package-private* modifier.

## Kotlin's internal visibility

Kotlin has given up the *package-private* modifier altogether, [likely][1]
because the *package-private* visibility could be circumvented so easily.
Instead, Kotlin came up with an *internal* modifier that limits the visibility
to the same module rather than the package. It promotes developing code in
a feature-by-feature modular fashion rather than a layer-by-layer monolithic
fashion.
Kotlin compiler throws a compilation error whenever Kotlin code attempts to
access an *internal* declaration located in another module.

The *internal* modifier is, however, purely a construct of Kotlin language, and
since neither Java nor JVM have a notion of *internal* visibility, this can be
easily circumvented as well.

```java
package library.internal;

class Rogue {
    Rogue() {
        AdsManagerKt.publicVisibility(); // ✅
        // Warning: Usage of Kotlin internal declaration from different module
        AdsManagerKt.internalVisibility(); // ✅
        // Error: 'privateVisibility()' has private access in 'library.internal.AdsManagerKt'
        // AdsManagerKt.privateVisibility(); // ❌
    }
}
```

As a courtesy of IntelliJ-based IDEs, we get a friendly warning when we access
*internal* declarations from Java code located in another module, but that's
about it regarding the support of the *internal* modifier in Java.

Java compiler will compile the code above just fine.
That's because all *internal* declarations are compiled with *public* visibility.
The information about the internal visibility is stored only in the `Metadata`
annotation attached to every compiled Kotlin class.

```kotlin
package library.internal

internal fun internalVisibility() = Unit
```

```java
package library.internal;

import kotlin.Metadata;

@Metadata(
    mv = {1, 7, 1},
    k = 2,
    xi = 2,
    d1 = {"\u0000\b\n\u0000\n\u0002\u0010\u0002\n\u0000\u001a\b\u0010\u0000\u001a\u00020\u0001H\u0000¨\u0006\u0002"},
    d2 = {"internalVisibility", "", "Playground.library.main"}
)
public final class AdsManagerKt {
    public static final void internalVisibility() {
    }
}
```

Kotlin compiler uses the information stored in `Metadata` annotations (and
perhaps the `META-INF/<module name>.kotlin_module` file too) to enforce
*internal* visibility.

Unfortunately, there is nothing in the bytecode that would prevent the Java
compiler from accessing *internal* declarations, and the compiler will not even
report this as a warning.

For the Java compiler, *internal* declarations are effectively *public*, which
breaks the *internal* modifier.

## Project Jigsaw

Kotlin's *internal* modifier existed for a long time before its first stable
release in 2016. Although I was not able to track down specific dates, it has
likely been there since Kotlin's inception in 2011.
The *internal* modifier even [used to be the default visibility modifier][2]
for quite some time if no modifier was stated explicitly, akin to Java's
*package-private* modifier.

The *internal* modifier was most likely an answer to a problem that the Java
platform did not address in time. Years later, the Java platform eventually
addressed the issue in the project Jigsaw, which finally saw release as a part
of JDK 9.

Project Jigsaw added the concept of modules as a first-class citizen to the JVM.
Using the `module-info.java` file, we can specify what packages a module
exports. In other words, anything not specified in the `module-info.java` file
will not be visible to other modules, even if it is marked as *public*.

This effectively achieves the same as what the *internal* modifier does.
Moreover, since modules are natively supported by the JVM itself, it is
available not only to Java but to all other languages running on the JVM
as well.

In addition to checking access violations during the compilation, it is also
better enforced at runtime. Any attempt to use reflection to make a
*package-private* or *private* declaration accessible would yield an exception:

> java.lang.reflect.InaccessibleObjectException: Unable to make static void library.internal.AdsManager.packagePrivateVisibility() accessible: module Playground.library does not "opens library.internal" to module Playground.consumer

Finally, the module system also verifies that the same package cannot be defined
more than once across all modules in the application, preventing the package
split trick discussed above.

## Conclusion

Java's module system introduced in Java 9 achieves a better encapsulation than
Kotlin's *internal* modifier. It also addresses some of the issues of the Java
platform itself, such as package split or reflection.

Unfortunately, using Java 9 is not an option on some platforms, such as
Android. With the rise of Kotlin in the Android ecosystem, there is only
a minuscule chance that Google would invest in upgrading the Android platform
to support Java 9.

Furthermore, I could not find a way to force consuming projects to use
the module system and obey the visibility rules defined in `module-info.java`.
Seemingly, because of the backward compatibility, the module system feature is
opt-in, even if the consumer runs on Java 9. This leaves an open backdoor,
and even with the module system in place, the visibility modifiers can still
be circumvented.

That was only my observation, anyway. But just because I did not find a way
does not mean there is no way. If I ever discover how to enforce the use of
the module system, I will update this article too.

[1]: https://discuss.kotlinlang.org/t/kotlin-to-support-package-protected-visibility/1544
[2]: https://discuss.kotlinlang.org/t/kotlins-default-visibility-should-be-internal/1400
