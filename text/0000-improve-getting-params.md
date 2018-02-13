- Start Date: (fill me in with today's date, 2018-02-12)
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

- Move `this.props.navigation.state.params` up to `this.props.params`
- Default `params` to an empty object
- Make it flow-able

# Basic example

```
const { name, id } = this.props.params
```

Since `params` will default to `{}` it will make the experience a bit more delightful.

# Motivation

I often find myself running into this situation. It can be frustrating when you forget to access the vars safely. I wish for `params` to either default to `{}` or for a convenience method to exist.

# Detailed design

TBD

# Drawbacks

Why should we *not* do this? Please consider:

- Implementation cost. There are a lot of places where `params` seem to interact with the codebase. I'm not sure how little or much time this would take me.
- Eventually users would have to migrate to the new API

# Alternatives

- Keeping the `params` where they currently live (navigation.state.params)
- Creating a `getParam` helper instead [(#3510)](https://github.com/react-navigation/react-navigation/pull/3510)
- Optionally making `params` default to an empty object

# Adoption strategy

- Deprecate `navigation.state.params` and show warning message with new API
- Remove in next minor? release

# How we teach this

- Add documentation explaining new change
- Add breaking changes in release docs

# Unresolved questions

There's a lot I don't know about the codebase. Anything I should look out for would be appreciated.
