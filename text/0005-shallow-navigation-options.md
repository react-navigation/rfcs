- Start Date: 2018-02-28
- RFC PR: https://github.com/react-navigation/rfcs/pull/26
- React Navigation Issue: (leave this empty)

# Summary

The current behavior on the 1.x branch for `navigationOptions` resolution for a route is different depending on whether the 'screen' component for the route is a normal React component or a nested navigator.

1. If it is a normal React component, the `navigationOptions` static is merged on top of the `navigationOptions` defined in the navigator config, then the `navigationOptions` defined in the route config (`{screen: MyScreen, navigationOptions: { /* etc */ }`) is merged on top of that.
2. If it is a nested navigator, we crawl into the navigator's active index recursively until we hit a leaf node (a normal React component) then use that as our base config. We then follow the same process, including grabbing `navigationOptions` off of the screen component, which is in this case a navigator (`MyNavigator.navigationOptions`). This is important to point out because the `navigationOptions` in the navigator config refer to the default `navigationOptions` for routes in the navigator, not for the navigator itself.

This RFC is to suggest the following changes:

- No longer recursively extract `navigationOptions` from nested navigators. All `navigationOptions` will be resolved based on rule number 1 described above.
- Change `navigationOptions` on navigator config to `defaultChildNavigationOptions` or something else with a name that more accurately describes the fact that it is not used by the navigator itself but instead by the routes inside of the navigator.
- Now that what was once `navigationOptions` will be called `defaultChildNavigationOptions` or something similar, we can use `navigationOptions` in navigator config to define the `navigationOptions` for the navigator itself.

# Basic example

```js
const ScreenA1 = () => <View />;
const ScreenA2 = () => <View />;
const ScreenB1 = () => <View />;
const ScreenB2 = () => <View />;

const StackA = StackNavigator(
  { ScreenA1, ScreenA2 },
  {
    navigationOptions: {
      tabBarLabel: 'Stack A',
    },
    defaultChildNavigationOptions: {
      title: 'Screen in Stack B',
    },
  }
);

const StackB = StackNavigator(
  { ScreenB1, ScreenB2 },
  {
    navigationOptions: {
      tabBarLabel: 'Stack B',
    },
    defaultChildNavigationOptions: {
      title: 'Screen in Stack B',
    },
  }
);

const RootNavigator = TabNavigator({
  StackA,
  StackB,
});
```

In this example we specify the `tabBarLabel` on the `navigationOptions` on `StackNavigator`. This will become a static property on the component that is returned from the `StackNavigator` function. If we were to specify the `tabBarLabel` on the `defaultChildNavigationOptions` then this would have no impact on the tab bar label.

# Motivation

The behavior for case number 1 described in the summary is predictable and intuitive (except perhaps with the exception of the `navigationOptions` from the route config having precedence over the `navigationOptions` defined on the component itself, but that is another discussion). The behavior for case number 2 is not quite as clear &mdash; it can lead to confusing behavior like these issues [1](https://github.com/react-navigation/react-navigation/issues/3571), [2](https://github.com/react-navigation/react-navigation/issues/3111), [3](https://github.com/react-navigation/react-navigation/issues/3090), and [4](https://github.com/react-navigation/react-navigation/issues/2378), to name only a few. This change would lead to less unnecessary "magic" in `navigationOptions`, making it easier to understand the library. It also makes it clearer that navigators can also have `navigationOptions` and gives an obvious way to specify them.

# Detailed design

- While implementing the [Navigation View API RFC](https://github.com/react-navigation/rfcs/blob/master/text/0002-navigator-view-api.md) we deleted the code responsible for crawling into nested navigators to extract `navigationOptions` ([see the createConfigGetter diff](https://github.com/react-navigation/react-navigation/commit/e27ad22c57444fadb827bf3bd3a75e3b2ca38041#diff-d7b38f711172bb2cb9393d3f1ca361a9)). This was accidental, but it led us to realize that this behavior was actually likely desirable, and we created this RFC. So no additional design work is needed for this.
- Using `navigationOptions` in navigator config as the `navigationOptions` on the navigator component itself rather than the default for child routes is trivial to implement. Similar for moving the current `navigationOptions` behavior over to `defaultChildNavigationOptions`.

# Drawbacks

- It is a breaking change in two ways: 1) people will need to rename `navigationOptions` to `defaultChildNavigationOptions` or something similar 2) they will need to move some of the config from `defaultChildNavigationOptions` or `navigationOptions` in static on child screens or route configs onto `navigationOptions` for the navigator.

# Alternatives

- Opt-in or out of this behavior. This adds complexity for maintainers and users.
- Don't change the existing behavior. If we don't do this then people will continue to be confused about why the configuration on some screen deep inside your app can impact a navigator that is its distant grandparent, like a stack -> drawer -> stack -> screen where the screen impacts the header of the outer stack.

# Adoption strategy

- This is a breaking change.
- A codemod for this would be difficult to write, we need to explain update steps to people. We would include it as part of the same major release as the `Navigation View API` refactor.

# How we teach this

- We should be explicit in the documentation about how `navigationOptions` are resolved. This becomes much easier due to this RFC.
- We should explain that nested navigators can have their own `navigationOptions` which are used by their parent navigator.

# Unresolved questions

It's unknown whether there are legitimate use cases for extracting `navigationOptions` from nested navigators that would be difficult to solve without it.
