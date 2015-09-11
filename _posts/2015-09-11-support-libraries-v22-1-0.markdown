---
layout: post
title: Support Libraries v22.1.0
date: '2015-04-22'
cover_image: '/content/images/2015/4/scaffolding.jpg'
---

It's been a while since my last post so here we are. You may have seen that the 22.1.0 support libraries were released yesterday, which is probably the biggest non-platform release we've done with the support library.

Before we go any further, have a read of Ian's official release [blog post](http://android-developers.blogspot.co.uk/2015/04/android-support-library-221.html). It outlines all of the new features in this release, for all of the libraries.

In this post I will concentrate more on the _how_ and _why_ certain things were done, particularly on the things I worked on (since I actually know the how and why on those).

## AppCompat

Lets start with AppCompat which has had a large update in this release. First, its refactoring...

### Refactoring

Previously the only entry point into AppCompat was through the now deprecated ActionBarActivity class. Unfortunately this forced you into using a set Activity hierarchy which made things like using PreferenceActivity impossible.

We've now extracted all of the internal stuff and exposed it as a single delegate API, [AppCompatDelegate](https://developer.android.com/reference/android/support/v7/app/AppCompatDelegate.html). AppCompatDelegate can be created by any Android object which exposes a [Window.Callback](https://developer.android.com/reference/android/view/Window.Callback.html), such as *any* Activity or Dialog subclass. You create one via its static [create()](https://developer.android.com/reference/android/support/v7/app/AppCompatDelegate.html#create(android.app.Activity, android.support.v7.app.AppCompatCallback)) methods.

There is a contract to maintain when you create a delegate. You *must* callback to it at every call it exposes (for instance `onCreate()`), but it's really simple and can be extracted into a base class.

The end result is that you can attach all of AppCompat's functionality to any Activity sub-class, as long as you call it as it wants.

If you plan on playing with AppCompatDelegate, I urge you to have a look at the AppCompatActivity source when you get a chance. It's an (extreme) example of how to integrate AppCompatDelegate.

Most people though won't need this level of customization though and can just use [AppCompatActivity](https://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html) (as you used to use ActionBarActivity).

### Dialogs

I just mentioned Dialogs in the refactoring step which should give you an idea of what else we've added. After completing the refactoring work, Dialogs were a natural next step. There is actually very little difference between an Activity and Dialog from a decor-setup point of view.

This means that we can finally close one of the biggest requests for AppCompat since v21: material styled dialogs.

We now have the new [AppCompatDialog](https://developer.android.com/reference/android/support/v7/app/AppCompatDialog.html) class which you should now use any time you use reference `Theme.AppCompat.Dialog` (or related).

To cap this section off, AppCompat now also has its own AlertDialog implementation for material styled AlertDialog everywhere. To use it, just change your usage to [android.support.v7.app.AlertDialog](https://developer.android.com/reference/android/support/v7/app/AlertDialog.html). It handles the rest.

One thing to note is that AppCompat's AlertDialog does not implement everything that the framework version does. It only exposes things which are valuable in this 'material world' (groan).

### android:theme

_Before we go into this section, make sure you have read my [Theme vs Style]({% post_url 2014-11-12-theme-vs-style %}) post. It explains the basis of what we're about to talk about._

In AppCompat v21, we exposed a quick-hack-get-it-out way for you to be able to set a theme on a [Toolbar](https://developer.android.com/reference/android/support/v7/widget/Toolbar.html) using `app:theme`.

In 22.1.0 we now have expanded that functionality so that you can set a theme on *any* view in your layouts. We have also moved to using `android:theme` which allows seamless handover between the compat and framework functionality.

The best bit though, is that the automatic theme inheritance from a view's parent also works on all devices running API v11 and above. Here's a quick example:

{% highlight xml %}
<Toolbar
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

    <!-- This TextView inherits its theme from the parent Toolbar -->
    <TextView android:text="I'm light!" />

</Toolbar>
{% endhighlight %}

For devices running API v10 and older, you can still use `android:theme` but the parent theme inheritance will not work. This means that you should either rethink your layouts, or set `android:theme` on all of the children (this is really inefficient though).

For those that are interested, the thing that enables the parent theme inheritance is [LayoutInflater.Factory2](https://developer.android.com/reference/android/view/LayoutInflater.Factory2.html).

### Widgets

If you read Ian's post then you'll have seen that the tinting widgets have now been exposed publicly (there's even a few new ones).

That's great, but there's another change that happened related to this: we no longer change the platform theme's default widget styles. What this means is that you will only get the material styling on a pre-v21 device, if you're using the AppCompat implementation of that widget (either implicitly or explicitly). In practice you should not see a difference since we insert AppCompat's implementation in-place automatically.

This allows us to fix an issue where our material styles were being used, but not tinted. This happened when the platform implementation of the widget was used with our style, and showed up in various places such as Preferences.

Instead, you will now see the default platform style (Holo, etc). This may look a little bit weird but it's better than a black un-tinted drawable you can't actually see.


### Theme window features

AppCompat is now more strict on what it expects in theme window flags, more closely matching what you would get from the framework.

The main reason behind this is to support dialogs which we mentioned earlier. They make heavy use of the `windowNoTitle` flag, which AppCompat previously did not pay much attention to.

You might be seeing the following exception once you have updated to v22.1.0:

    IllegalArgumentException: AppCompat does not support the current theme features

Have a look at my StackOverflow answer on how to fix your theme: [http://stackoverflow.com/q/29790070/474997](http://stackoverflow.com/q/29790070/474997)

## v4

The grandfather of the support libraries, support-v4 continues to grow and have new stuff added.

### ColorUtils

[ColorUtils](https://developer.android.com/reference/android/support/v4/graphics/ColorUtils.html) has been moved out of Palette and into support-v4 as a public class. It contains some really nice stuff in there for working with colors. For instance, you can calculate the minimum color alpha for text over a background color:

{% highlight java %}
int backgroundColor = ...;
int textColor = Color.WHITE;
float minContrastRatio = 4.5f; // We want a minimum contrast ration of 1:4.5

int minAlpha = ColorUtils.calculateMinimumAlpha(
        textColor, backgroundColor, minContrastRatio);

if (minAlpha != -1) {
    // There is an alpha value which has enough contrast, use it!
    return ColorUtils.setAlphaComponent(textColor, minAlpha);
}
{% endhighlight %}

There's other goodies in the class to, like colour composition and luminance calculation utilities. Have a look in the javadoc for more info.

### Drawable tinting

The Drawable tinting methods added in Lollipop are super useful for letting you dynamically tint assets. AppCompat had its own baked in implementation in the v21 support library and we've now extracted that into [DrawableCompat](https://developer.android.com/reference/android/support/v4/graphics/drawable/DrawableCompat.html) in support-v4 for everyone to use. It's important to know how it works though.

{% highlight java %}
Drawable drawable = ...;

// Wrap the drawable so that future tinting calls work
// on pre-v21 devices. Always use the returned drawable.
drawable = DrawableCompat.wrap(drawable);

// We can now set a tint
DrawableCompat.setTint(drawable, Color.RED);
// ...or a tint list
DrawableCompat.setTintList(drawable, myColorStateList);
// ...and a different tint mode
DrawableCompat.setTintMode(drawable, PorterDuff.Mode.SRC_OVER);
{% endhighlight %}

The thing to remember is that after you call [DrawableCompat.wrap()](https://developer.android.com/reference/android/support/v4/graphics/drawable/DrawableCompat.html#wrap(android.graphics.drawable.Drawable)), you can not rely on the result being the same type as what you give it. Instead you should use [DrawableCompat.unwrap()](https://developer.android.com/reference/android/support/v4/graphics/drawable/DrawableCompat.html#unwrap(android.graphics.drawable.Drawable)) to retrieve the original Drawable.

Internally, we now wrap your Drawable in a special 'tint drawable' will automatically update your Drawable's color filter from the specified tint. This allows us to handle [ColorStateList](https://developer.android.com/reference/android/content/res/ColorStateList.html) instances.

## Palette

Palette has also recieved a bit of love in this release. The first thing is that we've added a new [Builder](http://developer.android.com/reference/android/support/v7/graphics/Palette.Builder.html) class to help with instantiation. We found that we were adding more and more 'knobs' and settings to Palette which was making the API convoluted. Builders are a nice way to make this less painful as an API.

The second (and more important) change is the large performance increase in generating Palettes. The most costly piece of work in Palette is the colour quantization step. This take all of the pixels in a image and reduces the colour depth down to a small number of colours (usually 16).

In this release, we went back to some old-style optimizations for the colour quantization. Things like less object allocations, more appropriate data structures and a reduction in algorithmic complexity. These have resulted in a massive increase in speed.

Here are some quick tests I did. As you can see the speed-up is roughly 5-6x on a device using ART, but on a Dalvik device the increase is greater.

| Device    | 22.0   | 22.1.0 | Speedup |
|-----------|--------|--------|---------|
| Nexus 6   | 47ms   | 8ms    | ~6x     |
| Nexus 5   | 55ms   | 11ms   | ~5x     |
| Nexus One | 1200ms | 120ms  | ~10x    |

_The results are not scientific and only give a rough indication, but you get the idea._

---

_Cover photo: [Scaffolding](https://flic.kr/p/GbAqX) by Brett Weinstein_
