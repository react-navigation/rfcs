- Start Date: 2018-07-16
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Allow to define a handler to be executed upon long pressing a tab item.

# Basic example

It is already possible to define a `tabBarOnPress` in the navigationOptions of a tab bar.
My proposal is to duplicate this exact handler, but call it `tabBarOnLongPress` and use instead of using the `onPress` method from react-native, use the `onLongPress` method instead. You could use this to for example show a drawer or more navigation items (see Motivation section).

# Motivation

Consider the pattern that Instagram uses for account switching.
It is possible to long press the last tab item, and then switch to another account from a drawer.

![Instagram account switching](https://i.imgur.com/f4TYJrC.gif)

Here is another example: You can long-press on a tab bar item in Tweetbot to reveal more sections of the app that do not fit in the tab bar.

![Tweetbot](https://i.imgur.com/JkXuUyc.gif)

The motivation for this proposal is to enable more UI patterns for developers such as the ones shown above.
Instagram is a mainstream app with over a billion users and includes this gesture. This means that users are quickly getting more familiar with this gesture, which is why `react-navigation` should support it as well.

# Detailed design

The underlying logic of the `tabBarOnPress` event handler should be duplicated, but it should listen to the `onLongPress` event from `react-native` instead. For consistency, the event handler in `react-navigation` should be called `tabBarOnLongPress`.

# Drawbacks

I don't see any drawbacks of supporting this. However see the following section:

# Alternatives

When considering to add support for `tabBarOnLongPress`, it also brings up the question what to do with the other events like `onPressIn`, `onMagicTap`, `onAccessibilityTap` etc. Instead of explicitly adding support for `tabBarOnLongPress`, you may want to opt for a more generic design.

# Adoption strategy

No breaking change, no codemod or coordination with other projects needed. It should be added to documentation and mentioned in the release notes to spread the word.

# How we teach this

The new feature is analogue to the `tabBarOnPress` event handler and does not introduce a new paradigm. `react-navigation` can still be teached in the exact same way.

# Unresolved questions

See alternatives.