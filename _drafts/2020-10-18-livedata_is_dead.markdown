---
layout: post
title:  "LiveData is dead, long live LiveData"
date:   2020-10-18 18:50:00 +0900
tags:   programming
---

<!-- repeatOnLifecycle -->
<!-- flowWithLifecyclee -->

As of today, in late 2020, there are three major contenders when it comes to asynchronous programming on Android.
We are talking about none other than **RxJava**, **LiveData**, and Kotlin **Coroutines**.

**RxJava** has been around for many years before either LiveData or Coroutines saw the light of day and is infamous for its steep learning curve.
Understanding backpressure, hot vs. cold streams, hundreds of operators… One could easily be overwhelmed by its complexity, and mastering RxJava can take years of experience.
<!-- It felt like learning a completely new language. -->

**LiveData** is a simpler alternative to RxJava developed by Google tailored specifically for Android development.
The entry barrier is much lower than with RxJava as there is no notion of backpressure or whether the stream is hot or cold.
There are only 3 simple operators and it is subjectively harder to make a mistake, such as leaking resources by forgetting to properly dispose of no longer needed Observables.

**Coroutines**, and more specifically the recent release 1.4.0, have been the reason for this article.

Matured

<!-- map vs flatMap -->
<!-- filter vs flatFilter ?? -->

 <!-- Yet this extreme simplicity is not always welcome ???. -->

Several issues with LiveData

- difficult observing without a lifecycle owner
- difficult interop with coroutine-first libraries

Before we go on bashing LiveData for all of its fallacies, we should highlight what was the intention of `LiveData` in the first place and what `LiveData` does right.

## Strong points

<!-- https://developer.android.com/topic/libraries/architecture/livedata -->

### Lifecycle awareness

It is virtually impossible to crash the application because some part of the View was accessed at the wrong moment.
Since `onCreateView` is always called before `onStarted` and `onDestroyView` is always called after `onStop`,
LiveData makes sure that all the emissions are performed only when the lifecycle owner is at least in the `STARTED` state, as can be observed in `LiveData.LifecycleBoundObserver`:

```java
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {

    // Other members omitted for brevity

    @Override
    boolean shouldBeActive() {
        return mOwner.getLifecycle().getCurrentState().isAtLeast(STARTED);
    }
}
```

You still have to understand the difference between Fragment LifecycleOwner and View LifecycleOwner.
Using the former in place of the latter might be unnecessarily allocating multiple observers, wasting resources and potentially leading to unexpected behavior.

### Automatic disposal

In RxJava it was very simple to leak an `Observable`. The design of LiveData makes it much more difficult.
<!-- Technically, nothing prevents you from using `liveData.observeForever(observer)` that you have to remove manually. -->
Technically, you can use `liveData.observeForever(observer)` and never release the observer,
but it is more than obvious that in majority of situations you should use `liveData.observe(owner, observer)` instead.
This way, the observer is removed automatically when the Lifecycle moves to the `DESTROYED` state as seen in `LiveData.LifecycleBoundObserver` again:

```java
class LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver {

    // Other members omitted for brevity

    @Override
    public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
        Lifecycle.State currentState = mOwner.getLifecycle().getCurrentState();
        if (currentState == DESTROYED) {
            removeObserver(mObserver);
            return;
        }
        // Omitted for brevity
    }
}
```

### Two-way data binding

It is possible to utilize data binding and
<!-- data-binding is super brittle, very difficult or impossible to unit test, and brings Gradle to its knees (and isn't even supported by Buck). -->

If properly taken care of and everyone in the team understands Separation of concerns principle, I imagine it can have its merits.
But from my personal experience, DataBinding was creating more issues that it was solving.

`BindingAdapters` make code fragmented. People do not shy away from putting complex logic into XML layouts that makes it untestable. When sometimes goes awry, the compiler will produce useless error message unless you run the gradle command with `--stacktrace` option.

- cryptic error messages

> Note: In many cases, view binding can provide the same benefits as data binding with simpler implementation and better performance. If you are using data binding primarily to replace findViewById() calls, consider using view binding instead.

### Main thread safety

## Weak points

<!-- There are few points that cripple the usability of LiveData. -->

### Null-safety

LiveData is written in Java that has no way of distinguishing nullable parameter types from non-nullable ones.
Since we cannot mark parameter types with annotations like `class LiveData<@NonNull T>`, there is no way to get compile-time null safety.

The following code will compile and run just fine:

```kotlin
class MainFragment : Fragment(R.layout.main_fragment) {

    private val viewModel by viewModels<MainViewModel>()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewModel.text.observe(viewLifecycleOwner) {
            textViewMessage.text = it
        }
        buttonClear.setOnClickListener {
            viewModel.clear()
        }
        buttonRandom.setOnClickListener {
            viewModel.random()
        }
    }
}

class MainViewModel : ViewModel() {

    private val _text = MutableLiveData<String>()
    val text: LiveData<String> = _text

    fun clear() {
        _text.value = null
    }

    fun random() {
        _text.value = "Text"
    }
}
```

To alleviate the issue, `LiveData` authors created a lint check that will warn you that assigning `null` to `LiveData` with a non-nullable parameter type is not a good idea:
> Cannot set non-nullable LiveData value to null

<!-- or when trying to assign nullable data type:
> Expected non-nullable value -->

This prevents majority of mishaps, but still falls short when we observe LiveData originating in some poorly written library whose code we don't have control over. If we update the code to explicitly define observed type like this:

```kotlin
viewModel.text.observe(viewLifecycleOwner) { text: String ->
    textViewMessage.text = text
}
```

we will receive a runtime crash when a `null` value is emitted.
From the consumer perspective this is unexpected and non-transparent behavior.

> java.lang.NullPointerException: Parameter specified as non-null is null: method kotlin.jvm.internal.Intrinsics.checkNotNullParameter, parameter text

This is because, by default, a platform type is assumed that does not have any guarantees about nullability. For the explicit kotlin type compiler generates an additional a guard to accept only non-null values:

{% capture platform_type %}

```java
this.getViewModel().getText().observe(this.getViewLifecycleOwner(), (Observer)(new Observer() {
    // $FF: synthetic method
    // $FF: bridge method
    public void onChanged(Object var1) {
        this.onChanged((String)var1);
    }

    public final void onChanged(String it) {
        TextView var10000 = (TextView)MainFragment.this._$_findCachedViewById(id.textViewMessage);
        Intrinsics.checkNotNullExpressionValue(var10000, "textViewMessage");
        var10000.setText((CharSequence)it);
    }
}));

```

{% endcapture %}

{% capture kotlin_type %}

```java
this.getViewModel().getText().observe(this.getViewLifecycleOwner(), (Observer)(new Observer() {
    // $FF: synthetic method
    // $FF: bridge method
    public void onChanged(Object var1) {
        this.onChanged((String)var1);
    }

    public final void onChanged(@NotNull String text) {
        Intrinsics.checkNotNullParameter(text, "text");
        TextView var10000 = (TextView)MainFragment.this._$_findCachedViewById(id.textViewMessage);
        Intrinsics.checkNotNullExpressionValue(var10000, "textViewMessage");
        var10000.setText((CharSequence)text);
    }
}));
```

{% endcapture %}

<table class="code_samples" style="table-layout: fixed; width: 100%;">
<tr>
<td> {{ platform_type | markdownify }} </td>
<td> {{ kotlin_type | markdownify }} </td>
</tr>
</table>

## Events

The `LiveData` class, as provided by the library, is good at managing the state but fails miserably when used for events.
This is because upon subscription `LiveData` always emits the latest value it has received. This behavior does not make sense for events, which should be emitted once and only once.

There have been many attempts to bend LiveData to support events. Every now and then someone comes up with a new, subjectively better solution like
`class SingleLiveEvent<T>>`[<sup>[1]</sup>](https://github.com/android/architecture-samples/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java),
`class Event<out T>`[<sup>[2]</sup>](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150),
`class SingleLiveEvent2<T>`[<sup>[3]</sup>](https://medium.com/@star_zero/singleliveevent-livedata-with-multi-observers-384e17c60a16),
`class LiveEvent<T>`[<sup>[4]</sup>](https://medium.com/@hadilq.dev/rxjava-instead-of-livedata-in-mvvm-f95e8fe0aa41)[<sup>[5]</sup>](https://proandroiddev.com/livedata-with-single-events-2395dea972a8),
etc.

<!--
- [GitHub - android / architecture-samples / SingleLiveEvent.java](https://github.com/android/architecture-samples/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java)
- [Medium - LiveData with SnackBar, Navigation and other events (the SingleLiveEvent case)](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150)
- [Medium - SingleLiveEvent (LiveData) with multi observers](https://medium.com/@star_zero/singleliveevent-livedata-with-multi-observers-384e17c60a16)
-->

Some solutions don't take into account multiple observers. Other solutions require you to handle it yourself with `getContentIfNotHandled()` or `peekContent()` that is not bullet-proof either.

To make sure events are handled once and only once, the observer has to do an additional boilerplate check. You can use `EventObserver` that encapsulates this boilerplate instead of `Observer` but this should not be the responsibility of the observer nonetheless.

No solution that I am aware of supports buffering of events when no observers are currently attached and replaying all of them upon resubscription.
<!-- Only one backpressure strategy -->

To add insult to injury, when multiple observers are in play, solutions often don't take thread safety into account.

<!-- https://medium.com/@hadilq.dev/rxjava-instead-of-livedata-in-mvvm-f95e8fe0aa41 -->

<!-- Even Google sometimes produces shitcode https://developer.android.com/topic/libraries/architecture/livedata#extend_livedata StockLiveData.get(symbol) -->

## Observing without lifecycle

## Transformations

With the limited aresnal of transformation operators consisting only of `map`, `switchMap` and `distinctUntilChanged`, `LiveData` is useful only for the transmiting the value from ViewModel to View.

Try anything ever slightly more complex, like `debounce` or `combineLatest`, and you are bound to have a hard time coming up with a robust solution.

Observing database

## Interop with coroutines

<!--
Neither `map` nor `switchMap` operator supports asynchronous operations like calling suspending functions.

val userId: LiveData<String> = ...
val user: LiveData<User> = userId.switchMap { id ->
    liveData {
        emit(repository.fetchUser(id))
    }
}
-->

Since `LiveData` is tied to a `Lifecycle` of a lifecycle-driven object, it would not 
`CoroutineLiveData.kt`
```kotlin
@MainThread
fun cancel() {
    if (cancellationJob != null) {
        error("Cancel call cannot happen without a maybeRun")
    }
    cancellationJob = scope.launch(Dispatchers.Main.immediate) {
        delay(timeoutInMs)
        if (!liveData.hasActiveObservers()) {
            // one last check on active observers to avoid any race condition between starting
            // a running coroutine and cancelation
            runningJob?.cancel()
            runningJob = null
        }
    }
}
```

Coroutines don't need `flatMap`, since `map` is suspending.

## No backpressure

<!-- ## Creating LiveData in ViewModel

We've received a LiveData Coroutine builder where we can call suspending functions
This makes LiveData more than just a value holder.
LiveData created this way will continue running for 5 more seconds before cancelling the coroutine if there are no active observers. -->

### Other notes

The simple design of LiveData made solutions complex. - Nerubia "engine" liveData
promotes unsafe code
