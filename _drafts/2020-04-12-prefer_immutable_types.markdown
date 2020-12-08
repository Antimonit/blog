---
layout: post
title:  "Prefer immutable data"
date:   2020-04-12 14:36:49 +0200
tags:   programming
---

Immutability. One of the evergreens that took me a long time to appreciate.

`val` vs `var`

`List` vs `MutableList`

Be explicit.

```kotlin
fun <T> sort(list: List<T>): List<T>
```
```kotlin
fun <T> sort(list: MutableList<T>): List<T>
```


## Immutability via constructor parameters

Constructor parameters are the best immutable place

At any point of the 

Unfortunatelly, in the Android world it is never simple. There are two notable cases that deserve extra attention. We don't have complete control over instantiation of some of the Android components, such as `Activity` or `Service`, and limited control for others, such as `Fragment` or `ViewModel`.

### Fragments

We cannot use custom constructors for Fragments because they can be destroyed and recreated by the system. In such case the system just calls an empty constructor. If an empty constructor is not found, it crashes the app. (Let's forget about `FragmentFactory` for a moment that solves this very problem.)

Instead, we should pass all arguments via `fun Fragment.setArguments(args: Bundle?)` when we create the Fragment for the first time.

```kotlin
fun PreviewFragment.newInstance(message: String) = PreviewFragment().apply {
    arguments = Bundle().apply {
        putString(ARG_MESSAGE, message)
    }
}
```

```kotlin
class PreviewFragment : Fragment() {

    private lateinit var message: String

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        message = requireArguments().getString(ARG_MESSAGE)!!
    }
}
```

```kotlin
class PreviewFragment : Fragment() {

    private val message: String by lazy {
        requireArguments().getString(ARG_MESSAGE)!!
    }
}
```

### Pass arguments to ViewModels through constructor

```kotlin
class PreviewViewModel : ViewModel() {

    private lateinit var contentType: ContentType

    fun setContentType(contentType: ContentType) {
        this.contentType = contentType
    }

    // Other members
}
```

```kotlin
class PreviewViewModel(
    private val contentType: ContentType
) : ViewModel() {

    // Other members
}
```

You should probably use some kind of depdency injection so that you don't have deal with `ViewModelProvider.NewInstanceFactory()` boilerplate inside the Fragment.
