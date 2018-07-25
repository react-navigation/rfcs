- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Navigation containers should become mandatory for end users. Navigators will no longer automatically manage navigation state when there is no container.

`createNavigationContainer` had been an existing API for a long time, but it had been automatically applied to the top level navigator. Now users must use it explicitly.

This change enables web users to avoid depending on react-native, and helps users on react-native understand the functionality of their application.

# Basic example

Previously, a simple app might look like this:

```js
import { createStackNavigator } from 'react-navigation';

const App = createStackNavigator({ ...routeConfigs });
```

In this case, the App was automatically wrapped with a container by `createStackNavigator`. After the proposed change, a simple app would look like this:

```js
import {
  createNavigationContainer,
  createStackNavigator
} from 'react-navigation';

const AppNavigator = createStackNavigator({ ...routeConfigs });

const App = createNavigationContainer(AppNavigator);
```

# Motivation

When we automatically wrap every navigator with a state machine, it can be unintuitive to explicitly use a different container.

The biggest motivation is to support web users who do not want to depend on `react-native`. When the container API becomes explicit, it will be possible to create custom navigators that 

# Detailed design

The navigation container API (createNavigationContainer) will not change. We will change each navigator in v2 to warn against uncontained usage. In v3, the uncontained navigators will not work.

Every app must be manually wrap with a container before rendering in react native.

```js
import {
  createNavigationContainer,
} from 'react-navigation';

const App = createNavigationContainer(AppNavigator);
```

We will also export react-navigation modules as independent npm packages. For example, the web container could be imported with:

`import { createBrowserContainer } from '@react-navigation/web'`


# Drawbacks

The only drawback is the increased boilerplate. Most apps will need to add two lines of code to have an explicit container.

# Alternatives

We could explore a fully-implicit container design that works on multiple platforms, but it may break badly in some fringe environments like server rendering or browfield apps.

# Adoption strategy

We will make a blog post to announce this and introduce the web container usage.

# How we teach this

Docs and example apps will be updated, in addition to the annoucement blog post.
