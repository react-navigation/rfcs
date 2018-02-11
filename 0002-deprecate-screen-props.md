# What is `screenProps`

The `screenProps` prop allows the user to pass a prop to a navigator's children screen. This prop propagates deep into the children screens.

```js
<RootNavigator screenProps={{ foo: 'bar' }} />
```

# Use case

I initially added this prop to be able to pass some data to the child screens in a navigator from the parent. There are several other ways people are using this:

1. State management, to pass state from root to some screen down the tree
2. Reference to parent component instance, methods etc.

# Problems

1. People misuse this as a way to solve state-management issues, which is not something a navigation library needs to solve
2. Apart from the misuse, it's too easy to de-optimize the whole tree with `screenProps`. For example, `screenProps={{ foo: 'bar' }}` creates a new object every render, which breaks `shouldComponentUpdate` in all children navigators and screens, no matter how deep the tree is, and will re-render everything for a simple change. And actually most of the usage I have seen in the bug reports use it this way. It can also cause issues like this https://github.com/react-navigation/react-navigation/issues/2625

Even if we can document to avoid the re-render issue, this prop was present due to lack of other ways to achieve the same functionality. Now that React's context API is finally going to be stable, it's no longer the case.

# Alternatives

## Dynamically passing params

My initial use case was passing some data to the child screens, which can be addressed by allowing to pass params dynamically.

```js
<Tabs initialParams={{ foo: 'bar' }} />
```

Above code also has the re-render issue, but localized to one navigator. And the name `params` makes it clear that it's for passing params and not arbitrary data.

## Context API

React's context is a more explicit way to achieve the same functionality which doesn't interfere with `shouldComponentUpdate`. When using context, only the components using the data passed from the top will re-render, and not unrelated screens.
