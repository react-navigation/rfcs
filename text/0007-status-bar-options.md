- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

The configuration of status bars is mostly dependent on the screen, so it maps well to `navigationOptions`. Currently people add a `StatusBar` component to every screen, but this isn't really ideal because if you go from A -> B -> C, and B uses a light status bar and C doesn't indicate any status bar, then C will have a light status bar rather than the default. Additionally, when you switch between tabs it doesn't update the status bar config, you need to subscribe to navigation events.

# Basic example

```
class MyScreen extends React.Component {
  static navigationOptions = {
     // default to 'default', possible values: 'default', 'dark-content', 'light-content'
    statusBarStyle: 'light-content',

    // default to true
    statusBarVisible: true,

    // default to 'fade', possible values: 'fade', 'slide', 'none'
    // if slide is specified and only style is changed, then use 'fade'
    statusBarAnimation: 'slide',
  };
}
```

# Motivation

It improves library ergonomics and is highly requested ([1](https://react-navigation.canny.io/feature-requests/p/control-statusbar-config-for-screens-in-navigationoptions), [2](https://github.com/react-navigation/react-navigation/issues/11)). There is no downside for users who do not use it.

# Detailed design

The status bar configuration for deepest active route will always be applied on navigation state change. We will use the imperative `StatusBar` API, in particular `StatusBar.setBarStyle` and `StatusBar.setHidden`. We are intentionally leaving out `translucent` and `backgroundColor` options for the Android status bar because they are much less commonly used and the expected semantics aren't as straightforward.

# Drawbacks

This could be implemented in user space, but most apps need this and users will benefit from having a solution built into the tool.

The main drawback is it may confuse people how it relates to the `StatusBar` component and imperative API from react-native. We should suggest that people use this API and not the `StatusBar` API when possible.

# Alternatives

- Use a decorator for screen components that subscribes to focus and updates config accordingly, this could be implemented easily in userspace.

# Adoption strategy

No coordination with other libraries is necessary. People do not need to use it if they do not want to. If they do, they should get rid of all explicit `StatusBar` API calls (except for `translucent` and `backgroundColor` on Android) and use this API instead.

# How we teach this

We should update the [status bar documentation](https://reactnavigation.org/docs/en/status-bar.html).


# Unresolved questions

What's the best approach for Android specific properties? We don't know if the app is configured to translucent or not because there is no API in react-native to determine this.
