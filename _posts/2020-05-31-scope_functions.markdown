---
layout: post
title:  "Semantical let, apply, also, run & with"
date:   2020-05-31 13:47:08 +0200
tags:   programming
---

Kotlin defines functions named `let`, `apply`, `also`, `run`, and `with` that can help us write cleaner, more focused code by creating temporary scopes.
Hence the name 'scope functions'.

> The scope functions do not introduce any new technical capabilities, but they can make your code more concise and readable.

I find it unsatisfactory to describe their differences just by focusing on their signatures, what do they return, and whether the receiver is accessed via `it` or `this`, like in the following examples. 

{% capture kotlin_also %}
```kotlin
val size = "Hello".also {
    println(it)
}.length
```
{% endcapture %}
{% capture kotlin_run %}
```kotlin
val size = "Hello".run {
    println(this)
    this.length
}
```
{% endcapture %}
{% capture kotlin_apply %}
```kotlin
val size = "Hello".apply { 
    println(this)
}.length
```
{% endcapture %}
{% capture kotlin_let %}
```kotlin
val size = "Hello".let {
    println(it)
    it.length
}
```
{% endcapture %}

<table class="code_samples">
<tr>
<td> {{ kotlin_let | markdownify }} </td>
<td> {{ kotlin_run | markdownify }} </td>
</tr>
<tr>
<td> {{ kotlin_also | markdownify }} </td>
<td> {{ kotlin_apply | markdownify }} </td>
</tr>
</table>

[Official Kotlin documentation](https://kotlinlang.org/docs/reference/scope-functions.html) explains the technical aspect well and also presents code samples.
I don't want to delve into details about how they are defined but, instead, I would like to suggest a more semantical approach to choosing one over another.

Ultimately, any of them can, with minor differences, achieve the same goal.
But choosing the right function may better convey your intention---and as such, make the code easier to read and understand.


## When to use one

All of the scope functions (with an exception of `with`) work nicely in combination with `?.` and/or mutable receivers.

```kotlin
val content: View? = findViewById<View>(R.id.contennt)
if (content != null) {
    // do something with content
}
```
```kotlin
findViewById<View>(R.id.content)?.let { content: View ->
    // do something with content
}
```
The first obvious effect is that we don't have to explicitly check for the nullability of `content`. 
By using `?.` the scope function will execute only if the receiver is not null and non-null value is provided inside the function.

The other less obvious effect is that `content` is only available inside the `let` scope.
This is desirable as you should always strive towards limiting the scope of variables and functions. 
You would not declare a variable in a global scope if it was used only by a single class, would you?
Instead, you would just declare it in the class where it is used.
Likewise, you would not declare a variable in the class scope if it was only used locally in a single function.

Similar reasoning can be applied to variables inside functions as well.
Often a variable's sole purpose is to calculate the value of another variable.
In the first example, the scope of `content` leaks to all further, potentially dozens of expressions below the `if` statement. 
By using a scope function we can eliminate the variable and make it dead obvious it is not used again later in the function.


## Which one to use

### let

Often used in place of `if (value != null)` to conditionally execute a piece of code 
when there is not much difference between using `let` and other scope functions. 
Because it is used so commonly in many situations, using `let` does not provide as many semantical clues as other scope functions.

```kotlin
iconView?.let {
    when (intent.getStringExtra(EXTRA_ACTION)) {
        EXTRA_ACTION_CONCEAL -> conceal(it)
        EXTRA_ACTION_REVEAL -> reveal(it)
    }
}
```

<!-- When you don't need the receiver object after the scope is over or want the result of the block. -->

You can think of it as a `map` function being used on a single value.

```kotlin
fun stringToInstant(time: Long?): Instant? {
    return time?.let { Instant.ofEpochMilli(it) }
}
```
```kotlin
val fragment: Fragment = intent.extras?.getNullableLong(ARG_NOTE_ID).let { noteId ->
    if (noteId == null) {
        NoteDetailFragment.newAddFragment()
    } else {
        NoteDetailFragment.newEditFragment(noteId)
    }
}
```


### also

Commonly used to perform additional operations with the receiver object without the necessity of storing it in a variable.

```kotlin
return DocumentFile.fromTreeUri(context, uri)?.also {
    loadFolder(it)
}
```
You can think of it as an `onEach` function being used on a single value.

```kotlin
getSongsInPlaylist()
    .also(::println)
    .groupingBy(Song::author)
    .eachCount()
```


### apply

Use `apply` when your primary intention is to apply multiple changes to the receiver.

Use it primarily for **mutable** operations, i.e., in situations when the receiver would be on the **left** side of assignments or calling mutable methods on the receiver.

```kotlin
continueButton = findViewById<Button>(R.id.continue_button).apply {
    text = buttonText
    setOnClickListener {
        onContinueClicked()
    }
}
```

```kotlin
(parentFragment as? WordUpdatedListener
    ?: activity as? WordUpdatedListener)
    ?.apply {
        if (wordId == null) {
            onWordAdded(text)
        } else {
            onWordUpdated(wordId, text)
        }
    }
```

You can think of it as an alternative to a builder pattern that returns the same object after the invocation of each method. 

```kotlin
TransitionSet().apply {
    interpolator = LinearInterpolator()
    duration = 500
    ordering = TransitionSet.ORDERING_TOGETHER
    addTransition(Fade(Fade.OUT))
    addTransition(ChangeBounds())
    addTransition(Fade(Fade.IN))
}
```

```kotlin
return SongsListFragment().apply {
    arguments = Bundle().apply {
        putString(KEY_URL, url)
        putString(KEY_EDITABLE, false)
    }
}
```


### run & with

Use `run` or `with` when you want to have implicit access to members of the receiver but don't intend to modify it.

Use them primarily for **immutable** operations, i.e., in situations when the receiver would be on the **right** side of assignments or calling immutable methods on the receiver.

```kotlin
with(viewModel) {
    messagesLiveData.observe(viewLifecycleOwner, Observer {
        adapter.messages = it
    })
    continueEnabledLiveData.observe(viewLifecycleOwner, Observer {
        continueButton.isEnabled = it
    })
}
```

```kotlin
val confirmationMessage: CharSequence = with(context.resources) {
    buildSpannedString {
        append(getString(R.string.confirm_delete_songs_prefix))
        bold {
            append(getQuantityString(R.plurals.songs, count, count))
        }
        append(getString(R.string.confirm_delete_songs_suffix))
    }
}
```

`with` cannot be used in conjunction with `.?` to execute only when the receiver is non-null. In such cases use `run`.

```kotlin
arguments?.run {
    viewModel.loadWordDetail(getLong(ARG_WORD_ID))
}
```

Use `run` when the receiver object is a complex expression and would be difficult to read using `with`.

<!-- 
```kotlin
findViewById<Button>(if (isInEditMode) {
    R.id.saveButton
} else {
    R.id.editButton
}).apply {
    isVisible = true
    setTextColor(R.color.green)
}
```
-->


### Explicit it vs implicit this

`run`/`with` and `apply` functions override what object `this` refers to.
When it is not immediately obvious whose members are being accessed inside the scope of `run`/`with` or `apply`, consider replacing them with `let` or `also`.

```kotlin
return createView(AnkoContext.create(context)).apply {
    // Is `getTag()` called on the created view or the outer class?
    // Do we assign the value to a property of the created view or the outer class?
    layout = getTag() as View 
}
```

Likewise, consider using `let` or `also` when the scope of `run`/`with` or `apply` would shadow members of the outer scope.

```kotlin
class DocumentSelectionFragment : Fragment() {

    private var documentTypeSelectedListener: DocumentTypeSelectedListener? = null

    fun updateExtras(documentExtras: DocumentExtras?) {
        // Use `let` to avoid unnecessary or excessively long @labels.
        documentExtras?.run {
            this@DocumentSelectionFragment.documentTypeSelectedListener = documentTypeSelectedListener
        }
    }
}
```


### Beware of ?: opeartor

As the last point, I would like to warn about a common trend of using `?:` as a replacement for an `if` statement.

```kotlin
user?.let {
    process(it)
} ?: processDefault()
```

One would expect that if the `user` is null, `process` will be executed and `processDefault` otherwise.
In most cases, that assumption holds but there is a problem with the `let` function.
Since the return value of `let` is the last expression of the block, `?:` checks the nullability of the returned value, not the `user`.
If it is not obvious yet, in the case `process` function returned `null`, both `process` and `processDefault` would be executed.

This actually happened in my previous workplace. The `user` object sometimes got into an impossible state, causing us to scratch our heads for hours while searching for the bug.

As much as I love functional programming, I have since decided to avoid this approach and use plain `if` statements, so that it does not confuse developers not aware of the problem.
Another solution would be to use `also` which returns the same object as it was called on and does not suffer from the same problem as `let`.

<!-- 
```kotlin
if (user != null) {
    process(user)
} else {
    processDefault()
}
```
-->


## Conclusion

Hopefully, by now you can see the differences between scope functions
not only from the technical perspective but also from the semantical perspective.
Note that these are just personal suggestions on how to make the code a bit easier to understand by providing semantical hints to the reader
and they are certainly not meant to be strictly followed in every situation.

Sometimes it can be difficult to decide which of the functions to use in a given situation.
In case you want to use the receiver for both mutable and immutable operations, should you use `apply` or `run`?
Or should you rather separate mutable operations from the immutable ones and use both `apply` and `run`?
It heavily depends on the situation. My advice would be not to fret about it too much and use whatever feels more appropriate to you.
