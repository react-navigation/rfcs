* Start Date: 2018-01-16
* RFC PR: https://github.com/react-navigation/rfcs/pull/1
* React Navigation Issue: https://github.com/react-navigation/react-navigation/issues/1363

# Summary

Add an event emitter to the navigation prop that emits focus/blur events for the route.

This proposal has been pioneered by @satya164: https://github.com/react-navigation/react-navigation/issues/1363

This will resolve longstanding requests for such functionality, as seen here: https://github.com/react-navigation/react-navigation/issues/51

# Basic example

In this example, we focus the first text input when the screen has completed transitioning:

```
componentDidMount() {
  this._sub = this.props.navigation.addListener(
    'didFocus',
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

### Keyboard dismissal

KB must dismiss when scene _starts_ to transition back (mayBlur)

### StatusBar Re-configuration

Screens may want to re-configure the status bar when the become active or blurred

### Input autofocus

Inputs want to open the keyboard after the screen finishes transition

### Related Concepts for Future work and exploration:

_Back button handling_ - Screens may want to prevent back actions based on internal state

_Refocus_ - On extra tab press, navigate to top of stack, then scroll to top on next press

# Detailed design

## Event System

An event emitter will be provided as part of each screen navigation prop. The events will be emitted by the navigator as changes happen to the navigation state.

`this.props.navigation.addListener` will be a `NavigationEventSubscriber`:

```
export type NavigationEventSubscriber = (
  eventName: string,
  eventHandler: (payload: NavigationEventPayload) => void
) => {
  +remove: () => void,
};
```

## Event Payload

Each event will emit a payload of the same shape, with the following type:

```
type NavigationEventPayload = {|
  +state: NavigationState,
  +lastState: NavigationState,
  +action: NavigationAction,
|};
```

This shows the previous and current state of the navigation route, and the action that was fired.

## Top-Level "action" Event

The "action" event will be emitted for _every_ navigation prop, when an action attempts to change the navigation state. The event only fires for the navigation state of the particular screen/navigator.

In some cases, the navigation state has not changed and `props.navigation.state === newState`. One example of this is the 'maySwipeBack' action fired by the stack navigator, which will be implemented in a following proposal.

## Navigator Events

The following events will available to all screens inside of navigators. Navigators will implement these additional actions by subscribing to its own `onAction` event from `props.navigation.addListener`.

### willFocus

Called when a screen will become focused, after it is mounted. Will be re-called when returning to the screen. Useful for starting to subscribe to new data.

onWillFocus may be called extra times, which is considered 'refocusing', which is most commonly tapping on a tab that is already selected. On iOS this extra focus may result in scroll-to-top behavior.

### didFocus

Called after the transition of focusing the screen. May be useful for input autofocus that waits for the transition to complete before focusing, or any other animation that should happen when the screen finishes becoming active.

### willBlur

The screen will be popping or another screen may become active. At this point screens may unsubscribe from heavy data subscriptions or APIs.

### didBlur

The screen is fully invisible now. This action will NOT fire when a screen is unmounted, for example at the end of a pop transition.

## "Is Transitioning" State

The nav state will be augmented with the `isTransitioning` flag to notify subscribers when a navigation animation is happening. The new structure of a navigation state (/route) is:

```
type NavigationState = {
  index: number,
  routes: Array<NavigationRoute>,
  isTransitioning: boolean,
};
```

When an animated transition begins, the index/routes change, and isTransitioning is toggled to true. If it does not switch to true, the navigation happens immediately without animation.

There will be a new navigation action for transition completion: 'CompleteTransitionAction', which toggles the `isTransitioning` boolean to false. This is done by StackNavigator, or any navigator once the transition completes.

This augmented state and new completion action is useful for navigators to provide the correct focus/blur events to children.

## Immediate Navigation Actions

The "Navigate" and "Back" actions will automatically set `isTransitioning: true`. To override this behavior and request an immediate transition without animation, you can provide a new `immediate: true` option to the navigate and back actions, and the action will leave `isTransitioning` as false.

# Drawbacks

Code complexity, difficulty of flow-typing the events properly

# Alternatives

We had discussed an imperative strategy in [#51](https://github.com/react-navigation/react-navigation/issues/51), (componentWillFocus) but there are several drawbacks with that approach, including HOC breakage.

# Adoption strategy

Apps can incrementally start utilizing these events, as this was a missing feature until now.

# How we teach this

We would create an events doc for the main navigation section, and add events throughout the cookbooks. A big change like this would also merit a blog post.

# Unresolved questions

* Does this proposal help with Android back button _cancellation_? As proposed, this will work to notify a screen when going back, but we need additional changes if this were to affect back-button behavior. Lets answer this question in followup proposals.
