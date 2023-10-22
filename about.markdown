---
layout: page
title: About
permalink: /about/
---

Hi, I am David! I am a software engineer from Prague, Czech Republic enjoying my life in Seoul, South Korea.

I occasionally put up some articles about Android, Gradle, Kotlin or programming in general, usually when something
noteworthy comes up at work.

### Profession

For the majority of my career, I have led the technical development of projects with millions of users.
I typically took on the role of improving the architecture of the projects and building more robust foundations for 
other developers to build on top of.

I currently work as an Android developer at [LINE][line] where I strive to improve the architecture and code quality of
the [LINE Pay][line-pay] feature.

In my day-to-day work, I use [Kotlin][kotlin], but I have a lot of interest in other programming languages as well.
I believe learning other languages can make us more proficient in our primary languages as well.
The two languages I learned a lot from are [Rust][rust] and [Haskell][haskell].
I made a presentation a couple of years ago titled [Learn From the Enemies][learn from the enemies] which focuses
on some of the functional programming principles and how they can be effectively applied to the object-oriented
paradigm as well.

When actually programming, my goal isn't to create code that works at that moment but to write code that is guaranteed
to work even if the context of the code changes. A large volume of bugs occur because the code depends on unwritten
contracts which are checked neither by any tests nor by the compiler. Algebraic data types, strong typing and/or 
immutability may help to express such contracts in code. If these contracts are not satisfied, the compiler will 
complain, making the compilation act as a form of unit test.

But rather than fixing the potential issues on my own, I want other developers to understand the background of the 
problems themselves.
Over the years, I have prepared dozens of presentations for other developers.
These covered a wide range of topics, including, but not limited to:

* Dependency Injection
* Android themes and styles
* Clean/Layered architecture
* Coroutines main-safety
* Structured concurrency
* Testing coroutines
* Algebraic data types
* Gradle convention plugins
* ISO 8601 and java.time
* Composition over inheritance
* Unit testing MVVM
* Function composition
* Pure functions
* Function colorability
* OpenAPI specification

I often refer to SOLID principles, pointing out where the code violates these principles and providing a better
alternative. I also guide other developers in avoiding certain bad practices, such as the use of
the `@VisibleForTesting` annotation, or compatibility issues behind internally published APIs, and
help them write idiomatic Kotlin code rather than just using Kotlin to write Java code.

### Education

I studied at [Czech Technical University in Prague][ctu].
In 2015, I graduated with honours earning Bachelor's degree in Software Engineering.
In 2018, I earned a Master's degree, also in Software Engineering.

During my studies, I went to Korea as an exchange student
to [Seoul National University of Science and Technology][seoultech] in 2014,
[University of Seoul][uos] in 2016 and [Ajou University][ajou] in 2017.

### Other stuff

Outside of work, I like experimenting with and learning other technologies. I learned about quantum computing and wrote
and published a [Quantum computer emulator][quantum] using Kotlin DSL. I have also released
an [IntelliJ IDEA plugin][lowlighting] and contributed to and
published [several][anko] [open-source][eventsubject] [libraries][spdelegates],
including [Kotlin multiplatform][directorytree] projects.
I also have experience with reverse engineering Android apps and rooting my personal phone.

## About website

This website is hosted on [GitHub Pages][github-pages] platform and uses [Jekyll][jekyll] static site generator to
render content written mostly in Markdown.

[line]: https://line.me/

[line-pay]: https://pay.line.me/

[kotlin]: https://kotlinlang.org/

[rust]: https://www.rust-lang.org/

[haskell]: https://www.haskell.org/

[learn from the enemies]: https://docs.google.com/presentation/d/1BhlOcjYYxmKXrnML1PWxA8olS7WJdguRUI1iuLJVhb8/edit?usp=sharing

[ctu]: https://www.cvut.cz/en

[seoultech]: https://en.seoultech.ac.kr/

[uos]: https://english.uos.ac.kr/

[ajou]: https://www.ajou.ac.kr/

[quantum]: https://github.com/Antimonit/Quantum

[lowlighting]: https://github.com/Antimonit/Lowlighting

[anko]: https://github.com/AckeeCZ/anko-constraint-layout

[eventsubject]: https://github.com/Antimonit/EventSubject

[spdelegates]: https://github.com/Antimonit/SharedPreferencesDelegates

[directorytree]: https://github.com/Antimonit/DirectoryTree

[github-pages]: https://pages.github.com/

[jekyll]: https://jekyllrb.com/
