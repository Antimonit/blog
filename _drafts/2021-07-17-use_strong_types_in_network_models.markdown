---
layout: post
title:  "Use strong types in network models"
date:   2021-07-18 17:50:00 +0200
tags:   programming
---

In the project I am currently working on, I've discovered dozens of places in the app
that define network models with **dates** (e.g. birthday, passport expiration date, ...)
or datetimes (e.g. payment timestamp) as plain `String`s, just like the following model:

```kotlin
data class PersonalInfo(
    val firstName: String,
    val middleNames: List<String>,
    val lastName: String,
    val birthDate: String,
)
```

That is not too bad as long as you parse the field manually into a strong type
as soon as possible, preferably in a network mapper.
Unfortunately, you could often see these Stringly-typed dates escape from
the network layer all the way to the View layer where they were parsed for the first time.

<!-- Declare - define -->

All the popular serialization libraries that I know of (Moshi, KotlinX Serialization and even Gson)
allow installing adapters to serialize/deserialize custom non-primitive types.
Many developers don't realize they can safely declare a field in a network model as a `LocalDate`
and let the serialization library to do the marshalling for them.

Instead of using `String` types for all properties, we can (_and should_) define the model as follows:

```kotlin
data class PersonalInfo(
    val firstName: String,
    val middleNames: List<String>,
    val lastName: String,
    val birthDate: LocalDate,
)
```

<!-- Fail fast, crash app ASAP and uncover bugs ASAP. -->

It is a good practice to convert network responses to proper strong types as soon as posssible
and fail fast in case of misalignment between FE and BE.

<!-- It is better to let the app crash -->
<!-- It is better to fail fast, as soon as we receive a response rather than letting the data pass -->

<!-- This will uncover misalignment between FE and BE much sooner during development -->

Get rid of `String` types wherever possible.
Even if we didn't have Moshi/KotlinX serialization adapter to handle the
serialization/deserialization for us, we should convert the `String` to a `LocalDate` manually
when mapping network models into domain models.

There are surprisingly many places in our app where birthdate is passed around
as a `String` and parsed to a `Date` or `Calendar` only when some calculations or
customized formatting is necessary. There are some issues with this.

### No type-safety

Nothing prevents us from passing `"1993/03/28"` as birthDate.
    Though it might look reasonable at a glance, it is possible for it to later fail just because another
    part of the code expected it to be in a `yyyy-MM-dd` format.
    Remember that compilation itself can be regarded as a form of unit test.
    If the code compiles with strong types like `LocalDate`,
    you can be sure that you won't run into parsing problem down the line.

### More burdensome to work with

If you want to compare whether one date is before another,
    you should define a format in which they are represented,
    create an appropriate `Formatter`, parse them to a strong type and compare results.
  On the other hand, if you had two `LocalDate`s, all you need to do is just `start < end`.

## Outdated `java.util` package

It should be pointed out that the old Time API found in `java.util` package
is widely regarded as deprecated.
That includes classes like `Date`, `Calendar`, `SimpleDateFormat` and similar.
It is not well-designed API, and it is better to avoid it.
Working with `Calendar` is cumbersome,
parsing using `SimpleDateFormat` is too lenient by default and
`Date` does not understand the concept of timezones/offsets and does very little 
to prevent incorrect comparison of `Date`s with and without timezones accounted for.

Instead, you really should start using `LocalDate`, `LocalDateTime` & `Instant`
(and `OffsetDateTime` and `ZonedDateTime` if you feel like a pro, but I didn't
really have a need for those myself so far)
But from my personal observation, those are necessary only when

Depending on the minSdk of your project, you might not be able to use classes from `java.time`
package directly and need to use an alternative option, such as ThreeTenBP library or desugaring
of Java 8 features.

> ### ThreeTenBP
>
> For those unfamiliar, [**ThreeTenBP** library](TODO add link) is a **b**ack**p**ort of JSR-**310**
> specification. The specification was first introduced in Java 8 more commonly known as `java.time` package.
> ThreeTenBP can be used in Java 6 and higher.
>
> In the **A**ndroid world, devices running Android SDK 26 and lower still run Java 6.
> [ThreeTen**A**BP library](TODO add link) is almost an exact clone of ThreeTenBP library with the
> only difference being the internal loading of timezone information.
> Other than that those libraries work the same and even package names are identical.
> That is also the reason why in Gradle configuration you can often see the following two lines together:
>
> ```kotlin
> implementation("com.jakewharton.threetenabp:threetenabp:1.3.1")
> testImplementation("org.threeten:threetenbp:1.5.1")
> ```
>
> With this setup,
> code compiled to an APK uses an implementation optimized for Android,
> meanwhile JVM tests use the original implementation.

> ### Java 8 Desugaring
>
> Desugaring of Java 8 features aims to solve the same problem as ThreeTenABP – to bring Java 8 features to old platforms.
> During compilation, the desugaring plugin replaces references to `java.time` package with `j$.time` and
> bundles a custom implementation in the APK/AAB.
>
> Unfortunately, this did not work out well for us since mangling of package names of desugared classes
> does not include XML files, such as in the case of navigation graphs with SafeArgs.
> See this issue: TODO

from java.time package (in Android environment the aforementioned org.threeten.bp).
This API is safer to use - the strong types will prevent you from mistakes like
comparing `Date` with timezone information with a `Date` without timezone information.

Also, in my opinion it is simpler to use, for example when it comes to managing time
between two moments (using Period or Duration classes) or managing timezones/offests.
The API might look more complex than the former Date-related classes, but in my opinion,
it is just because the API is designed in a way to prevent you from doing things that
were possible previously but should not have been.

If you properly handled the difficult stuff like timezones before, it should not be
too difficult to use java.time.

## What types to use?

Backend should not do any assumptions about the timezone of a client.
Only the frontend should leave out human-readable format up to the Frontend.
I can’t think of a situation where
Network models should use only
`LocalDate`s for dates without specific time of the day and
`Instant`s for specific dates and times of a day.

`LocalTime` for repeated events or alarms.

The proper formatter should be `yyyy-MM-dd'T'HH:mm:ss` that conforms to `ISO-8601`.

We (and BE as well!) should utilize standard formatters defined in DateTimeFormatter class, such as:

- `DateTimeFormatter.ISO_INSTANT` -> used by `Instant`
- `DateTimeFormatter.ISO_LOCAL_DATE` -> used by `LocalDate`
- `DateTimeFormatter.ISO_LOCAL_DATE_TIME` -> used by `LocalDateTime`
- `DateTimeFormatter.ISO_LOCAL_TIME` -> used by `LocalTime`
- `DateTimeFormatter.ISO_OFFSET_DATE_TIME` -> used by `OffsetDateTime`
- `DateTimeFormatter.ISO_ZONED_DATE_TIME` -> used by `ZonedDateTime`

There are other formatters as well, but they are not really important for us.

I cannot think of a situation where we would need to use anything else than Instant and LocalDate in our network models.

## What is wrong with LocalDateTime?

The problem with response like `2011-12-03T10:15:30` is that it does not contain any
offset/timezone information.
As such, the only thing we can do is to hope that the user is in the Philippines.
Instead, it would be better to use `Instant` that might look like `2011-12-03T10:15:30Z`.
The **Z** at the end stands for **Z**ulu which is equivalent to GMT or UTC timezone.

Convert Instant to LocalDateTime

Then we can calculate the actual time to display to the user given the timezone he actually resides in.

### Difference between Timezone and Offset

### TLDR

- Don't use old `Date`, `Calendar`, `SimpleDateFormat` API and instead start using `java.time`
  API like `LocalDate`, `LocalDateTime`, `Instant`, `Duration`, `Period` etc.
- Avoid using `String` type for date/time-related information in network models and
  define them directly as `LocalDate` or `Instant` type.
