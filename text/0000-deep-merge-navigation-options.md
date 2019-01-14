- Start Date: 2019-01-14
- RFC PR:
- React Navigation Issue:

# Summary

- Deep merge `navigationOptions` between the screen and `defaultNavigationOptions`.
- Refer: [#5468](https://github.com/react-navigation/react-navigation/issues/5468)

# Motivation

`navigationOptions` support multiple nested properties, like `headerStyle`.

With the current behaviour,
  if a style is defined in `defaultNavigationOptions.headerStyle` and another style is defined in screen's `navigationOptions.headerStyle`, the styles defined in `defaultNavigationOptions` are overwritten.

Hiding bottom border for the navigation bar using `borderBottomWidth` is one of the best use case. I added `borderBottomWidth` in `defaultNavigationOptions`. But, when I changed the background color of the bar in a screen, `borderBottomWidth` was no longer working. I had to add `borderBottomWidth` in screen's `headerStyle` also.

# Detailed design

Merge navigation options recursively just how lodash's merge works.

# Drawbacks

- This is a breaking change.

# Alternatives

- We can add an extra property in navigation options to control merge (shallow / deep)

# Adoption strategy

- This will be an internal change. No APIs will be affected.

# How we teach this

I think a little explanation in the [docs](https://reactnavigation.org/docs/en/headers.html#sharing-common-navigationoptions-across-screens) will be enough. As, docs are not really telling developers that they are shallow merged.
