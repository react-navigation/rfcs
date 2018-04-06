- Start Date: 2018-02-09
- RFC PR: https://github.com/react-navigation/rfcs/pull/18
- React Navigation Issue: https://github.com/react-navigation/react-navigation/pull/3619

# Modular Actions Creators

Different routers should expose different actions. For example, it may make sense to 'pop' within a StackRouter, but an app with a drawer router should have `openDrawer` and `closeDrawer` actions available.

AddNavigationHelpers forces `createNavigator` to support every action that every router supports, which prevents custom routers from exposing new action types. This is a proposal to replace `addNavigationHelpers` with a more modular set of navigation actions, defined by each router. This also allows us to deprecate the global concept of "NavigationActions" entirely.

## Basic Example

```
// Only available in a Stack:
this.props.navigation.popToTop();

// Only available in a Drawer:
this.props.navigation.openDrawer();
```

## Detailed design

We will add a new function on the router, called "getActionCreators", which will return action creators. For example, in the StackRouter:

```
const stackRouter = {
    getActionCreators: route => ({
        popToTop: () => ({ type: 'PopToTop' }),
        setParams: (params) => ({ type: 'SetParams', key: route.key, params }),
        ...
    }),
    ...
}
```

Inside of createNavigator, when the child navigation prop gets prepared, it will use the router to get the action creators. It will make the actions available within `childNavigation.actions`, and will also merge in parent actions provided via `this.props.navigation.actions`. That will enable the following:

```
// this.props.navigation.actions will only have `pop` within stacks:
const action = this.props.navigation.actions.pop();
this.props.navigation.dispatch(action);
```

As a convenience, the actions are also exposed on the navigation prop. These will call `dispatch` under the hood automatically:

```
// Only available in a Stack:
this.props.navigation.popToTop();
// Only available in a Drawer:
this.props.navigation.openDrawer();
// Only available in Tabs or Drawer:
this.props.navigation.jumpTo();
```

This allows us to have the convenience of `addNavigationHelpers`, but each router can provide contextual helpers.

### Custom action support

Each router will support a new param, `getCustomActionCreators(route, navStateKey)`, which allows the user to provide additional action creators that will become helpers on the navigation prop.

```
const AppNavigator = createStackNavigator({...routeConfigs}, {
  getCustomActionCreators: (route, navStateKey) => ({
    goHome: () => NavigationActions.navigate('Home'),
  }),
});
```

Now, you will be able to call `props.navigation.goHome` within this navigator.

## Action Creator Definitions

Now is a good time to revisit the helpers that we have available, and what arguments they should support.

### Stack Router
    - Navigate
    - GoBack
    - Push
    - Pop
    - PopToTop
    - Reset

### Tab Router
    - Navigate
    - GoBack
    - JumpTo

### Drawer Router
    - Navigate
    - GoBack
    - JumpTo
    - OpenDrawer
    - CloweDrawer
    - ToggleDrawer


### Drawbacks

The action creators will override eachother in the same namespace. Which means that if you nest two stack navigators, you will only be able to access the action creators of the inner navigator when calling from inside it.

If two custom navigators both implement the same action, users may be confused about the behavior in their app.

Documentation may be easier when all of the actions are listed in the same place, as opposed to them being broken down per-router.

### Alternatives

We could leave things as-is, with a global set of NavigationActions, that may balloon in scope over time.

### Adoption Strategy

We would roll this out as a point-release of the library, because it shouldn't break any existing actions or normal use-cases.

If we decide to change the definitions of the helpers, it should be released in a major version and people can change their action creators at that time.


# Unresolved questions

Should we change the signature of these actions? If so, what to?

Is it possible for routers to have overlapping actions? How can we avoid unexpected behavior in these cases?
