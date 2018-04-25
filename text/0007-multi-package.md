- Start Date: 2018-04-25
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Export react-navigation as several packages on npm, such that the exact dependencies can be required. Navigation containers will be explicitly used by app developers.

# Basic example

Currently, only the following API is available:

```js
import { createStackNavigator } from 'react-navigation';

const App = createStackNavigator(...);
```

We can continue to support this for the beginner use-case. But we would change the best practice, and publish packages to support the following:

```js
import { createStackNavigator } from 'react-navigation-stack-navigator';
import { createAppContainer } from 'react-navigation-native-container';

const AppNavigator = createStackNavigator(...);
const App = createAppContainer(AppNavigator);
```

Instead of depending on `react-navigation`, the app would explicitly depend on just `stack-navigator` and `native-container`. (Both of which would then depend on `react-navigation-core`) By moving the container into userland, we help people understand how react-nav works, and let them use an alternate container if they wish. Everyone on redux, for example, does't need a container at all!

We can also expose an API like this, for people who use react-nav on the web:

```js
import { createSidebarNavigator } from 'react-navigation-sidebar-navigator';
import { createAppContainer } from 'react-navigation-web-container';

const AppNavigator = createSidebarNavigator(...);
const App = createAppContainer(AppNavigator);
```

# Motivation

Right now, anybody building on top of react-navigation needs to import the full thing. This leads to a lack of innovation in the community because this pattern discourages from creating their own navigators, routers, views, and containers.

Plus, the single package makes `react-navigation` quite closely tied to `react-native`. Using `react-navigation` on the web is difficult right now beacuse it pulls in a lot of react-native specific dependencies.

# Detailed design

First we need to decide on the approach to manage this from an organization and tooling perspective. Generally we need to decide between a monorepo vs a multirepo approach. Once we decide how to do it, the package decoupling section describes the proposed structure of the new packages.

## Monorepo vs. Multirepo

Should the code for all of these new packages live in the main `react-navigation` repo, or should we create a new repo for each package? Both of these approaches have tradeoffs:

### Advantages of Monorepo

- Tooling is all in one place. Only one playground/test/lint/codecov/CI/githubbot to maintain
- Far easier to make API changes across the library
- Easier to make package versioning consistent
- Changelog for the whole library is in one place


### Advantages of Multirepo

- Tooling will be simpler for smaller repos than for a monorepo
- Pull requests and issues are scoped to the specific area of the codebase
- The react-navigation org may get bigger and more diverse because we are no longer tied to a single repo, web support can flourish
- Users can refer to git repo directly in package.json dependencies
- GitHub organization will match the npm organization and reality more closely


## Package Decoupling

The code in the main repo would be broken up into the following packages:

```
react-navigation-core

react-navigation-native-container
react-navigation-web-container

react-navigation-switch-router
react-navigation-switch-navigator

react-navigation-transitioner
react-navigation-stack-router
react-navigation-stack-actions
react-navigation-stack-view
react-navigation-stack-navigator
```

The official tab packages would be exported by `react-navigation-tabs`, and would live in the separate repo:

```
react-navigation-tabs-view
react-navigation-tabs-router
react-navigation-tabs-navigator
```

For beginners, we can continue to provide the main `react-navigation` package, which bundles basically all of the above. 

# Drawbacks

The tooling and maintenence burden will be increased when our package count increases. This is true of either the monorepo or multirepo approach.

Also, we will need to carefully consider version numbers across our universe of packages to avoid API-incompatible breakages.

# Alternatives

If we want to keep exporting a monolithic package, and properly support web, we would have to go update JS tooling like `create-react-app` and `razzle` in order to support all of the wacky react-native things that get imported.

# Adoption strategy

Keep supporting the monolithic `react-navigation` package to avoid distruption to current users. Encourage new users to import from the independent packages.

# How we teach this

Roll out as part of the next major release, update all documentation. We can blog and tweet about the development in order to excite the developer community to build custom navigators/views/containers/routers, or identify the area of the react-navigation codebase that they would like to contribute to.

# Unresolved questions

Help is needed with the implementation of either the monorepo or the multirepo approach! Our repo has a lot of tooling that we'd like to not break: prettier/lint, expo-deploys on each PR, tests with codecoverage, flow something, rn-cli.config.js hacks, multiple playground apps. For web support we should add a babel build step. It would be great to configure CI to auto-deploy to NPM when the version bumps and tests pass. The multi-package goal is clear, but there will be a lot of work to get this done, and we need support from JS tooling enthusiasts to get this set up.
