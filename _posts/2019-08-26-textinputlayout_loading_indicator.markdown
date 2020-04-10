---
layout: post
title:  "TextInputLayout with a loading indicator"
date:   2019-08-04 15:40:10 +0200
tags: android
---

We have a screen where the user enters their address through a list of dropdown menus: **State**, **City**, and **District**.
Contents of each dropdown menu depend on the item selected in the previous dropdown because the data is dynamically loaded from the backend.
The user must select the **State** first before proceeding to select **City** and likewise, they must select **City** before selecting **District**.
The request may take a considerable time to finish so we would like to show a loading indicator to the user.

We use `TextInputLayout` throughout the app which supports a wide variety of end icon modes:

<p align="center">
  <img src="/assets/images/text_input_layout/none.png" width="40%" height="40%" />
  <img src="/assets/images/text_input_layout/password_toggle.png" width="40%" height="40%" /> 
  <img src="/assets/images/text_input_layout/error.png" width="40%" height="40%" />
  <img src="/assets/images/text_input_layout/clear_text.png" width="40%" height="40%" />
  <img src="/assets/images/text_input_layout/dropdown_menu.png" width="40%" height="40%" />
  <img src="/assets/images/text_input_layout/custom.png" width="40%" height="40%" />
</p>

Naturally, using the end icon to display our loading indicator seems like a perfect fit.
It is straightforward to use a custom drawable in place of end icon like this:

```kotlin
textInputLayout.endIconMode = TextInputLayout.END_ICON_CUSTOM
textInputLayout.endIconDrawable = customDrawable
```

But it turns out that with a loading indicator it is not as simple as one would think...
	
## Drawable

Right away, the first problem is getting hold of a loading indicator drawable. There are several ways how to tackle the problem. Some better than the others, let's have a look.

### Bad option 1

There is no public drawable resource we can use for a loading indicator.

There is `android.R.drawable.progress_medium_material` but it is marked private and cannot be resolved in code. Copying the resource and all of its dependent private resources totals into about 6 files (2 drawables + 2 animators + 2 interpolators). That could work but feels quite like a hack. 

### Bad option 2

We can use `ProgressBar` to retrieve its `indeterminateDrawable`. The problem with this approach is that the drawable is closely tied to the `ProgressBar`. The indicator is animated only when the `ProgressBar` is visible, tinting one View will also tint the indicator in the other View and probably additional weird behavior.

In similar situations, we can use `Drawable.mutate()` to get a new copy of the drawable. Unfortunately, the `indeterminateDrawable` is already mutated and thus `mutate()` does nothing.

What actually worked to decouple the drawable from the `ProgressBar` was a call to `indeterminateDrawable.constantState.newDrawable()`. See [documentation][1] for more insight.

Anyway, this still feels like a hack.

### Good option

Although the drawable resource is marked private we can resolve certain theme attributes to get the system's default loading indicator drawable. The theme defines `progressBarStyle` attribute that references style for `ProgressBar`. Inside of this style is `indeterminateDrawable` attribute that references themed drawable. In code we can resolve the drawable like this:

```kotlin
fun Context.getProgressBarDrawable(): Drawable {
    val value = TypedValue()
    theme.resolveAttribute(android.R.attr.progressBarStyleSmall, value, false)
    val progressBarStyle = value.data
    val attributes = intArrayOf(android.R.attr.indeterminateDrawable)
    val array = obtainStyledAttributes(progressBarStyle, attributes)
    val drawable = array.getDrawableOrThrow(0)
    array.recycle()
    return drawable
}
```
Great, now we have a native loading indicator drawable without hacks!

## Animation

Now if you plug in the drawable into this code 
```kotlin
textInputLayout.endIconMode = TextInputLayout.END_ICON_CUSTOM 
textInputLayout.endIconDrawable = progressDrawable
``` 
you will find out that it does not display anything.

Actually, it does display the drawable correctly but the real problem is that it is not being animated. It just happens that at the beginning of the animation the drawable is collapsed into an invisible point.

Unfortunately for us, we cannot convert the drawable to its real type `AnimationScaleListDrawable` because it is in `com.android.internal.graphics.drawable` package.
Fortunately for us, we can type it as `Animatable` and `start()` it:
```kotlin
(drawable as? Animatable)?.start()
```

## Colors

Another unexpected behavior happens when `TextInputLayout` receives/loses focus. At such moments it will tint the drawable according to colors defined by `layout.setEndIconTintList()`. If you don't explicitly specify a tint list, it will tint the drawable to `?colorPrimary`. But at the moment when we set the drawable, it is still tinted to `?colorAccent` and at a seemingly random moment it will change color.

For that reason I recommend to tint both `layout.endIconTintList` and `drawable.tintList` with the same `ColorStateList`. Such as:
```kotlin
fun Context.fetchPrimaryColor(): Int {
    val array = obtainStyledAttributes(intArrayOf(android.R.attr.colorPrimary))
    val color = array.getColorOrThrow(0)
    array.recycle()
    return color
}

...

val states = ColorStateList(arrayOf(intArrayOf()), intArrayOf(fetchPrimaryColor()))
layout.setEndIconTintList(states)
drawable.setTintList(states)
```

## Result

Ultimately, we are presented with a nice little spinning indicator. Hey there, buddy!

<p align="center">
  <img src="/assets/images/text_input_layout/loading_indicator.png" width="90%" height="90%" />
</p>

See [this project][github-repo] for full implementation.

[1]: https://developer.android.com/reference/android/graphics/drawable/Drawable.ConstantState
[github-repo]: https://github.com/Antimonit/Loading-Indicator
