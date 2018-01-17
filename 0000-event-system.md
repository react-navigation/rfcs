- Start Date: 2018-01-16
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Add an event emitter to the navigation prop that emits focus/blur events for the route.

This proposal has been pioneered by @satya164: https://github.com/react-navigation/react-navigation/issues/1363 https://github.com/react-navigation/react-navigation/issues/51

# Basic example

In this example, we focus the first text input when the screen has completed transitioning:

```
componentDidMount() {
	this._sub = this.props.navigation.events.addListener(
		'DidFocus',
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
Screens must know when to request live data or resource-intensive APIs. Media should be able to auto-pause at correct times.

### Unmounting
Tabs and cards may need to unmount and re-mount contents when they are focused.

### Back button handling
Screens may want to prevent back actions based on internal state

### Keyboard dismissal
KB must dismiss when scene *starts* to transition back (mayBlur)

## Input autofocus
Inputs want to open the keyboard after the screen finishes transition

### “Refocus”
On extra tab press, navigate to top of stack, then scroll to top on next press


# Detailed design

An event emitter will be provided as part of each screen navigation prop. The events will be emitted by the navigator as changes happen to the navigation state.

`this.props.navigation.events` will be a `NavigationEventEmitter`:

```
export type NavigationEventEmitter = {
  +addEventListener: (
    eventName: string,
    eventHandler: (input: mixed) => void
  ) => {
    +remove: () => void,
  },
  +of: (childRoute: string) => NavigationEventEmitter,
};
```

The following events will be emitted:

### WillFocus
Called when a screen will become focused, after it is mounted. Will be re-called when returning to the screen. Useful for starting to subscribe to new data.

WillFocus may be called extra times, which is considered 'refocusing', which is most commonly tapping on a tab which is already selected. On iOS this extra focus may result in scroll-to-top behavior.

### DidFocus
Called after the transition of focusing the screen. May be useful for input autofocus that waits for the transition to complete before focusing, or any other animation that should happen when the screen finishes becoming active.

### MayBlur
Called when the user begins to swipe back and the screen may be blurred. Useful for dismissing the keyboard. Also may be useful to cancel the back event in cases where changes need to be saved or cancelled.

### WillBlur
The screen will be popping or another screen may become active. At this point screens may 

### DidBlur
The screen is fully invisible now. This will get called before a screen is unmounted.

# Drawbacks

Code complexity, difficulty of flow-typing the events properly

# Alternatives

We had discussed an imperative strategy in #51, (componentWillFocus) but there are several drawbacks with that approach, including HOC breakage.

# Adoption strategy

Apps can incrementally opt in to this feature, as this was a missing feature until now

# How we teach this

We would create an events doc for the main navigation section, and add events throughout the cookbooks. A big change like this would also merit a blog post.

# Unresolved questions

- How should the implementation work?
- How should back button cancellation work?
- Refocus event vs extra dispatches of 'Focus'
- Should there be a general 'route change' event, or will that simply be handled via props.navigation.state?
