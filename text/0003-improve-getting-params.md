- RFC PR: https://github.com/react-navigation/react-navigation/pull/3510

# Summary

- Create `getParam` helper method: `this.props.navigation.getParam`
- `getParam` will consume paramName and fallback: `getParam(paramName, fallback`
- if paramName is not found in object, return fallback value

# Basic example

```
const name = this.props.navigation.getParam('name', 'Peter')
```

# Motivation

I often find myself running into this situation. It can be frustrating when you forget to access the vars safely.

# Detailed design

Add `getParam` navigationHelper

# Drawbacks

Why should we *not* do this? Please consider:

-

# Alternatives

- Make `params` default to an empty object

# Adoption strategy

- Documentation. Everything will continue to work as intended

# How we teach this

- Add documentation explaining new change

# Unresolved questions

- 
