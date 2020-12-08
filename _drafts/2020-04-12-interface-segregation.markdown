---
layout: post
title:  "Interface segregation"
date:   2020-04-12 14:36:49 +0200
tags:   programming
---

The Home screen composed of
`HomeActivity` and `HomePresenter`


In the project I am working on there is a class that grew over time. It grew so much that now it has over 1000 lines of code. I am talking about our main Fragment -- `HomeFragment`.

But it is not only the Fragment to blame. The `HomePresenter` has beefy 600 LOC, `DashboardActivity` over 800 LOC.

What is common to all of these classes is that there is so much logic in each of the classes that it is no longer clear, what are the classes responsible for. It handles everything from navigation, remote config, sizing of items in Toolbar, user profile, analytics drawer, menu

There is no bright future for this class. Noone has the capacity to fix it anymore. It has to be removed and rewritten with better separation of concerns in mind.


```kotlin
class AddWordViewModel(
    languageDao: LanguageDao,
    labelDao: LabelDao,
    translationDao: TranslationDao,
    private val spelling: SpellingUseCase = SpellingSection(translationDao),
    private val language: LanguageUseCase = LanguageSection(languageDao),
    private val definition: DefinitionUseCase = DefinitionSection(),
    private val translations: TranslationsUseCase = TranslationsSection(translationDao),
    private val labels: LabelsUseCase = LabelsSection(labelDao),
    private val examples: ExamplesUseCase = ExamplesSection()
) : ViewModel(),
    SpellingUseCase by spelling,
    LanguageUseCase by language,
    DefinitionUseCase by definition,
    TranslationsUseCase by translations,
    LabelsUseCase by labels,
    ExamplesUseCase by examples {

    fun onSaveClicked() {
        // Combine data from all use cases and send to server / store to database
    }
}
```

Each `UseCase` can hold an internal state that no other section necessarily has to know about. 

Each `UseCase` is highly cohesive.
