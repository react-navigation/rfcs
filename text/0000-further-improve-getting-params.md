- Start Date: 2018-09-19
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

- Create `getParams` helper method: `this.props.navigation.getParams`
- getParams takes optional `fallback` parameter: `getParams(fallback)`
- If `fallback` is not provided, returns empty object

# Basic example

```javascript
const { name } = this.props.navigation.getParams();
```

# Motivation

Same motivation as RFC `0003-improve-getting-params.md`.

People who just use plain JavaScript might correctly protest that the following is an alternative:

```javascript
const { name } = this.props.navigation.state.params || {};
```

But using that from TypeScript will require me to add typing to the empty object:

```typescript
interface NavigationParams {
  name: string;
}

const { name } = this.props.navigation.state.params || ({} as NavigationParams);
```

Which I think is a bit verbose. And it would be less readable (in my opinion) if the fallback-value is more than just an empty object.

Just using `getParam(paramName)` is feasible:

```typescript
const { name } = this.props.navigation.getParam("name");
```

But it's not very IDE/workflow-friendly.

If I try to "find usages" in VS Code with the `name` property on the `NavigationParams`-interface, then the above `getParam`-code will not show up in the results.

In my opinion, when using a typed language, it is a bit of an anti-pattern to use string-keys for retrieving values. Since it makes it more difficult to navigate around the codebase.

Since the `this.props.navigation` object normally has typing (in TypeScript-codebases):

```typescript
import { NavigationScreenProp, NavigationState } from "react-navigation";

interface NavigationParams {
  name: string;
}

interface Props {
  navigation: NavigationScreenProp<NavigationState, NavigationParams>;
}
```

Then adding `getParams` will make the code less verbose:

```typescript
const { name } = this.props.navigation.getParams();
// or
const { name } = this.props.navigation.getParams({ name: "Unknown name" });
```

While also supporting static typing and easier navigating around the code.

Will also be useful to devs using plain JavaScript.

# Detailed design

Much like `getParam` has a `createParamGetter` helper-function, `getParams` could easily be created like so:

```javascript
const createParamsGetter = route => (defaultValue = {}) => {
  return route.params || defaultValue;
};
```

Which can be added in the `getChildNavigation.js` file:

```javascript
// snipped code, getParams must be added two places, just like getParam:
if (children[childKey]) {
  children[childKey] = {
    // snipped away code
    getParam: createParamGetter(childRoute),
    getParams: createParamsGetter(childRoute)
  };
  return children[childKey];
}
```

I suggest returning an empty object as the default fallback value. This is to allow safe accessing properties of the return value when `getParams` is invoked without fallback.

# Drawbacks

One impact could be devs confused by naming, and not understanding the difference between `getParam` and `getParams`.

# Alternatives

You could also add a helper-function in the library instead, so people have to explicitly import it:

JavaScript-version:

```javascript
function getParams(navigation, fallback = {}) {
  return (
    (navigation && navigation.state && navigation.state.params) || fallback
  );
}
```

TypeScript-version:

```typescript
function getParams<S, P>(
  navigation: NavigationScreenProp<S, P>,
  fallback: P = {} as P
): P {
  return (
    (navigation && navigation.state && navigation.state.params) || fallback
  );
}
```

Usage:

```javascript
const { name } = getParams(this.props.navigation);
```

Personally I'd be annoyed having to import this everywhere, since I'd need to use it quite a few places. But it is an alternative.

# Adoption strategy

This is not a breaking change.

# How we teach this

Will probably require some docs change. Move towards not recommend using `this.props.navigation.state.params` any longer.

# Unresolved questions
