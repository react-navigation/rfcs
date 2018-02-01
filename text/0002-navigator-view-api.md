* Start Date: 2018-01-29
* RFC PR: https://github.com/react-navigation/rfcs/pull/7
* React Navigation Issue: (leave this empty)

# Summary

Introduce a simpler API for createNavigator and navigation views. By providing "descriptors" to the navigation view, it does not need to compute options or manually create the child navigation prop.

Down the line, this will enable us to deprecate `addNavigationHelpers`, which is an avoidable burden for redux apps and when implementing custom navigators. Eventually, each navigator may provide a different set of helpers.

# Basic example

Using a new `createNavigator` API and `descriptors` prop, this is simplest possible custom navigation view:

```js
const CustomNavigationView = ({navigation, descriptors}) => {
  const {routes, index} = navigation;
  const descriptor = descriptors[routes[index].key];
  const ScreenComponent = descriptor.getComponent();
  return (
    <ScreenComponent
      navigation={descriptor.navigation}
      options={descriptor.options}
    />
  );
};
const router = StackRouter({...}, {...});
const CustomNavigator = createNavigator(CustomNavigationView, router);
```

This `<CustomNavigator />` will have behave like a stack, only rendering the active scene. It can present screens as defined by the `router`.

# Motivation

This change prevents the router from being passed directly into the navigation view, which was causing coupling between the concepts. This change will allow for faster innovation on the navigation views, and allow them to be consumed by other navigation libraries.

This also makes it easier to implement navigation views, because the children navigation props have already been provided within `props.descriptors`

# Detailed design

Navigators will now be created via a simpler API:

`const Navigator = createNavigator(NavigationView, router, navigationConfig);`

In this example, navigationConfig is an object that will be passed as a prop to the navigtion view. The `NavigationView` takes the following props:

* `navigation` - Navigation prop with `dispatch`, `state`, and `addListener`
* `screenProps` - The navigator will pass this through, so that the navigation view can provide screenProps to each screen
* `navigationConfig` - Arbitrary config object from `createNavigator`
* `descriptors` - Object map, with keys corresponding to the `route.key`. Each scene descriptor has:
  * `navigation` - Child navigation, with events and route state
  * `state` - Shortcut to navigation state. Same as `sceneDescriptor.navigation.state`
  * `key` - Shortcut to route key. Same as `sceneDescriptor.state.key`
  * `getComponent()` - Function to get the required screen component
  * `options` - Navigation options for the screen, as defined with navigationOptions on the route config and screen component

Importantly, the navigation view no longer receives a router prop. Instead, it gets `descriptors`. Routers are no longer aware of the navigation options that a view may need, meaning that custom navigators are much easier to implement and understand.

This change will ensure that `addNavigationHelpers` happens automatically for each child navigation prop, and the view will no longer be responsible for it. This is not desirable because different navigatators may wish to expose different helpers. As a followup to this RFC, the helpers could be moved off of the navigation prop, which may reduce boilerplate and remove the need for `addNavigationHelpers`.

For more details, [see the experimental implementation here.](https://github.com/react-navigation/react-navigation/pull/3392)

# Drawbacks

This is a breaking API change, so it will incur changes on anybody currently implementing a custom navigator. Fortunately it also makes it easier to implement custom navigators.

Also, this change will result in a different API for redux users. They will no longer need to wrap the top-level navigation prop with `addNavigationHelpers`. This is a better API, but people will be forced to change.

# Alternatives

The alternative is to leave things the way they are, but the coupling causes pain for maintainers and navigation hackers.

# Adoption strategy

Implementors of custom navigators would switch to the new API, which is a forced change, but the new API is simpler.

Redux users can/may remove `addNavigationHelpers` from their integration.

# How we teach this

Improve documentation for custom navigators, and remove the section on `addNavigationHelpers`.

Also, move over the example custom navigators within navigation playground. [See the proposed changes here](https://github.com/react-navigation/react-navigation/pull/3392).

# Unresolved questions

What is the future of `addNavigationHelpers`, now that it is mixed up inside of createNavigator?

One suggestion: remove all helpers from `props.navigation`, so that it is simply `state`, `dispatch`, and `addListener`. Each navigator would provide custom helpers, _but they would not be on the navigation prop_. So, users would be asked to migrate from `props.navigation.navigate` to `props.navigate`. This will allow navigators to provide different helpers, depending on the use case. StackNavigator could provide `props.push` and `props.pop`, while TabNavigator may provide `jumpTo` instead.
