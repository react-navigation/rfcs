- Start Date: 2018/07/13
- RFC PR:
- React Navigation Issue:

# Summary

Add a `navigateToURL` method to the API. Allows users to navigate just like `Linking.openURL` does internally without using `Linking.openURL`.

# Basic example

```
compose(
  withNavigation,
  withHandlers({
    onDynamicLinkReceived: ({
      navigation,
    }) => link => {
      navigation.navigateToURL(link) // navigates within the app as if `Linking.openURL` was called
    }
  })
)
```

# Motivation

Currently the [recommended approach](https://github.com/react-navigation/react-navigation/issues/266#issuecomment-363899405) to navigating with a full URL is to use [`Linking.openURL`](https://facebook.github.io/react-native/docs/linking.html).

There are downsides to using this approach. If we are already in the app, calling `Linking.openURL` may trigger user
approval to continue using the app to handle the link:

<img alt="android open with dialog" src="https://codelabs.developers.google.com/codelabs/android-deep-linking/img/48c972de20b04439760ad17d9b57d5fe.png" width="439" height="287" />

Consider the following use case using [Firebase dynamic links](https://firebase.google.com/docs/dynamic-links/):

- User receives an SMS with a dynamic link like `https://example.page.link/xxx` and clicks it
- The user has never clicked a link for `https://example.page.link` that before and Android prompts them which application to open the link with
- The user chooses to open `https://example.page.link/xxx` in our app
- The application received the dynamic link and using Firebase API can retrieve the link `https://example.com/xxx`. The application now must route the user to the appropriate path `/xxx` in the application. Using the recommended approach the application calls `Linking.openURL('https://example.com/xxx')`
- The user has never clicked a link for `https://example.com` before and Android prompts them which application to open the link with (again)
- The user is probably confused why they were asked twice after they clicked only one link. They may feel uncertain about what to do

It is also more performant to directly route to a URL instead of using `Linking.openURL`.

# Detailed design

I think adding we should add `navigateToURL` function to the [navigation property API](https://reactnavigation.org/docs/en/navigation-prop.html). This would not be a breaking change--simply exposing existing functionality that is implemented [here](https://github.com/react-navigation/react-navigation/blob/1fe11c100e72a632258e749d76bf542d425584b7/src/createNavigationContainer.js#L132).

Proposed implementation [here](https://github.com/s-nel/react-navigation/commit/9a9431b033fece4370309dead53b3cf82a090699)

# Drawbacks

- More API surface area to maintain
- Encourages using deeplinking/paths which already have issues logged against them

# Alternatives

To avoid the situation described above, the user (of `react-navigation`) can try to parse the URL themselves to determine the route and navigate via existing API calls.

# Adoption strategy

We will document the new API on the website. This should not be a breaking change.

# How we teach this

We should recommend users use this API to navigate to deep links. We should recomment `Linking.openURL` only for external URLs. We will document the new API on the website.

# Unresolved questions

- Should there be a navigation action for this?