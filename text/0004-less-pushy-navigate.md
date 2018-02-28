- Start Date: 2018-02-23
- RFC PR: https://github.com/react-navigation/rfcs/pull/18
- React Navigation Issue: (leave this empty)

# Summary

This RFC suggests making the `navigate` action for `StackRouter` focus the given `routeName` if it already exists in `StackRouter` state, rather than the current behavior of pushing it. If it does not exist in the state but the `StackRouter` handles the route, then it will push it.

In other words, this proposed change is the same as how `StackRouter` behaves if you specify a `key` to the `navigate` action but without requiring that the user specify a `key` to take advantage of the behavior. If no `key` is provided, `StackRouter` assumes the user just wants to either jump to the route by the given name if it exists in state already or push it if not.

In addition to this change, `push` and other `StackRouter` related navigation helpers will be changed from non-bubbling & specific to the deepest `StackRouter` to be global in the same way that `navigate` is -- when you push the new route it can be handled by any `StackRouter` in the state tree. The reason for this is that there is now a clear use case for why this matters. Take for example the flow of navigating from a user profile -> comment -> user profile. It is conceivable that you may visit user profile for "Jane", then in a comment visit the profile for "Bob", then in a comment visit "Jane" again. In this situation it is expected that the second time you visit "Jane" it pushes a new profile screen on top, rather than jumping back to the first one. For these situations, users should use `push`.

# Basic example

In the "Moving between screens" section of the documentation, we have the following example:

```
class DetailsScreen extends React.Component {
  render() {
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>Details Screen</Text>
        <Button
          title="Go to Details... again"
          onPress={() => this.props.navigation.navigate('Details')}
        />
      </View>
    );
  }
}
```

This would no longer produce the advertised result. If you want to be able to push an endless number of screens here, you would need to either 1) call `push` explicitly, or 2) pass in a unique key for each screen.

```
// Example 1
// This would be preferred
class DetailsScreen extends React.Component {
  render() {
    const { navigation } = this.props;
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>Details Screen</Text>
        <Button
          title="Go to Details... again"
          onPress={() => {
            navigation.push('Details');
          }}
        />
      </View>
    );
  }
}
```

```
// Example 2
// This is contrived, it would make no sense to push a new key like this
class DetailsScreen extends React.Component {
  render() {
    const { navigation } = this.props;
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>Details Screen</Text>
        <Button
          title="Go to Details... again"
          onPress={() => {
            navigation.navigate({
              routeName: 'Details',
              key: navigation.getParam('n', 0) + 1
            });
          }}
        />
      </View>
    );
  }
}
```


# Motivation

- Getting the key for the initial route in a stack is causing people problems (https://github.com/react-navigation/react-navigation/pull/3540). It's easy to push duplicate screens because of this (https://github.com/react-navigation/react-navigation/issues/3265).
- This also helps mitigate the longstanding issue of double-pushing routes in a `StackNavigator`. This is because you need to explicitly provide a key if you would like to push a route that you expect can have multiple instances in the stack. Many routes aren't like this, for example in the iOS settings app it only makes sense to have one instance of any of the settings screens. Routes that can be pushed multiple times are almost always provided with some params that are used to populate the data on the screen, and some id from that data can be used as the route key (for example, you might navigate to a profile screen with a `userId` param, and you can use that as the key). (https://github.com/react-navigation/rfcs/issues/16)
- I suspect that this new behavior is more intuitive. `navigate` is more clearly distinguished from `push` -- it tries to go to an existing route and only pushes one if it's not already there. `push` always pushes a new route to state. It makes `navigate` for `StackRouter` behave more similarly to `TabRouter` also.

# Detailed design

We need to change [this branch](https://github.com/react-navigation/react-navigation/blob/2bb91a6740e2f1ade012906dc03c972ba6375cc3/src/routers/StackRouter.js#L185-L190) of StackRouter. The logic will be similar to the [`if (action.key)`](https://github.com/react-navigation/react-navigation/blob/2bb91a6740e2f1ade012906dc03c972ba6375cc3/src/routers/StackRouter.js#L200) branch, but rather than simply match for key we'll match for `routeName` within `state.routes`.

# Drawbacks

- This is a significant breaking change. `navigate`, until the recent introduction of specific helpers/actions like `push` for `StackRouter`, has been the primary way to move between the app.
- A small breaking change to make `StackRouter` actions `push`, `pop`, `popToTop`, and `replace` bubble in the same way that `navigate` does. This will likely have a minimal impact if we move on this change soon, as we just introduced these actions. In the worst case, users will have some action be performed when they expected a noop (and probably should not have called it in the first place) and it should not be difficult to fix.

# Alternatives

- We could introduce an entirely new action and keep `navigate` as-is.
- We could leave the current behavior unchanged.

# Adoption strategy

- We should give users the ability to migrate without needing to manually update their apps by deprecating the old behavior for one major release. We can do this by keeping the old codepaths in place for an action with a remapped name, so instead of `NAVIGATE` it will be `NAVIGATE_DEPRECATED`, and the helper for this action creator would be `navigate_deprecated` rather than `navigate`.  
- This needs to be a major version change.
- I believe this is unlikely to break any libraries. It also should not impact custom routers unless they are re-implementations of `StackRouter`, in which case the users will want to update to match this behavior.

# How we teach this

- We need to explain what a `key` is in the documentation. There is [an existing issue for this](https://github.com/react-navigation/react-navigation.github.io/issues/50) but it is increasingly important as we make it a more central part of the development flow.
- We can encourage users to use `push` when they want to push an arbitrary number of screens for a single route.
- We'll need to do a pass over the documentation to ensure it is consistent with the changes.
- As mentioned previously, this must go with a major version change to clearly communicate the breaking nature of the change. The release notes should provide information necessary to update your app.

# Unresolved questions

- Existing tutorials in the wild might depend on the old behavior. It's unclear how we can work around this. Perhaps we can add a warning in dev mode if users try to navigate multiple times to the same route multiple times, without providing a key?
