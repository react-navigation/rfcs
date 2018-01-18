* Start Date: 2018-01-16
* RFC PR: (leave this empty)
* React Navigation Issue: (leave this empty)

# Summary

Add an event emitter to the navigation prop that emits focus/blur events for the route.

This proposal has been pioneered by @satya164: https://github.com/react-navigation/react-navigation/issues/1363 https://github.com/react-navigation/react-navigation/issues/51

# Basic example

In this example, we focus the first text input when the screen has completed transitioning:

```
componentDidMount() {
  this._sub = this.props.navigation.addEventListener(
    'onDidFocus',
    this._focusFirstTextInput
  );
}
componentWillUnmount() {
  this._sub.remove();
}
```

# Motivation

Screens need to know about navigation changes while they are mounted. Consider the following use cases:

### Data and Media subscription

Screens must know when to request live data or when to enable/disable resource-intensive APIs. Media should be able to auto-pause at correct times.

### Unmounting

Tabs and cards may need to unmount and re-mount contents when they are focused.

May be useful in conjunction with removeClippedSubviews and utilities like https://github.com/SoftwareMansion/react-native-resource-saving-container

### Back button handling

Screens may want to prevent back actions based on internal state

### Keyboard dismissal

KB must dismiss when scene _starts_ to transition back (mayBlur)

### StatusBar Re-configuration

Screens may want to re-configure the status bar when the become active or blurred

### Input autofocus

Inputs want to open the keyboard after the screen finishes transition

### “Refocus”

On extra tab press, navigate to top of stack, then scroll to top on next press

# Detailed design

## Events

An event emitter will be provided as part of each screen navigation prop. The events will be emitted by the navigator as changes happen to the navigation state.

`this.props.navigation.addEventListener` will be a `NavigationEventSubscriber`:

```
export type NavigationEventSubscriber = (
  eventName: string,
  eventHandler: () => void
) => {
  +remove: () => void,
};
```

The following event will be emitted for _every_ navigation prop:

### onAction(action, newState)

This fires for every action that may affect the navigation state.

In some cases, the navigation state has not changed and `props.navigation.state === newState`. One example of this is the 'onMaySwipeBack' action fired by the stack navigator.

The following events will be emitted _inside of navigators_. Navigators will implement these additional actions by subscribing to its own onAction event from `props.navigation.addEventListener`.

### onWillFocus

Called when a screen will become focused, after it is mounted. Will be re-called when returning to the screen. Useful for starting to subscribe to new data.

onWillFocus may be called extra times, which is considered 'refocusing', which is most commonly tapping on a tab that is already selected. On iOS this extra focus may result in scroll-to-top behavior.

### onDidFocus

Called after the transition of focusing the screen. May be useful for input autofocus that waits for the transition to complete before focusing, or any other animation that should happen when the screen finishes becoming active.

### onWillBlur

The screen will be popping or another screen may become active. At this point screens may unsubscribe from heavy data subscriptions or APIs.

### onDidBlur

The screen is fully invisible now. This will get called before a screen is unmounted.

## Navigation State

The nav state will be augmented with the `isNavigating` flag to notify subscribers when a navigation animation is happening. The new structure of a navigation state (/route) is:

```
type NavigationState = {
  index: number,
  routes: Array<NavigationRoute>,
  isNavigating: boolean,
};
```

When an animated transition begins, the index/routes change, and isNavigating is toggled to true. If it does not switch to true, the navigation happens immediately without animation.

There will be a new navigation event for navigation completion: 'CompleteNavigation', which toggles the boolean to false.

This augmented state and new completion action is useful for navigators to provide focus events on this changing state.

# Drawbacks

Code complexity, difficulty of flow-typing the events properly

# Alternatives

We had discussed an imperative strategy in #51, (componentWillFocus) but there are several drawbacks with that approach, including HOC breakage.

# Adoption strategy

Apps can incrementally opt in to this feature, as this was a missing feature until now.

# How we teach this

We would create an events doc for the main navigation section, and add events throughout the cookbooks. A big change like this would also merit a blog post.

# Unresolved questions

* How does this proposal help with Android back button _cancellation_? As proposed, this will work to notify a screen when going back, but we need additioanl changes if this were to affect back-button behavior.
