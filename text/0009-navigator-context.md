- Start Date: 2019-05-13
- RFC PR:
- React Navigation Issue:

# Summary

This RFC adds the option to bind a navigator and its flow to a Context provider.

# Basic example

Given a Context and its provider, one option is to pass it as a `navigationOption` to the navigator:

```js
import { Provider } from 'MyContext';
...
const navigator = createStackNavigator(
  routes,
  {
    contextProvider: Provider,
  },
);
```

# Motivation

Currently, there's no way to wrap a navigator into a Context provider. This forces us to wrap a much
larger part of the code than necessary in the provider to achieve the same result. It'd be very
helpful to only bind a navigator and its flow into a Context.

# Detailed design

I honestly don't know enough of react-navigation's implementation to suggest how this chould be
implemented.

If there's a container component that wraps the entire navigator, this component could be wrapped by
the given context provider. This is the simplest approach I can think of.

Another alternative would be to externalize the navigator container's render as a way to customize
the whole container.

We'll probably also need a way to update the context provider's value. This would already be
encompassed by the externalized render suggested above, since the function could be called when the
props update.

# Drawbacks

The only drawback I see for this is the implementation cost, partly because I have little knowledge
of the library's architecture.

This is not a breaking change and is only possible to be implemented in the user space with workaround
or by breaking basic responsibilities principles since it can be achieved in a more global manner.

# Alternatives

As mentioned, the alternative is to wrap a larger part of the code in the context provider. This can
cause multiple unexpected errors like unnecessary rendering of components that are not related to the
context.

Also this apprach goes against good code practices like restrainig modules responsibilities.

# Adoption strategy

Since this is an extension feature and not a breaking change, I don't see the need to coordinate with
other libraries. This would be an optional feature that people wouldn't be required to use if they
don't need.

# How we teach this

Extending the documentation should be enough to spread the word and teach developers how to use this
feature.

# Unresolved questions

How to update the context provider's `value` prop is a big question that can be solved in numerous
ways.