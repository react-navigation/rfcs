- Start Date: 2023-06-16
- RFC PR:
- React Navigation Issue:

# Summary

Change the API of `react-native-tab-view` to better suit react navigation's API. This will allow preventing a lot of re-renders and performance issues when using TabView together with `material-top-tabs`.

The idea is to make TabView's API leverage the descriptor pattern which is heavily used in react navigation.

```ts
type MaterialTopTabDescriptorMap = Record<string, MaterialTopTabDescriptor>;
```

# Basic example

TabView will accept the `scenes` array which contains route descriptors.

```tsx
export default function TabViewExample() {
  const [index, setIndex] = React.useState(0);
  const [routes] = React.useState([
    { key: "first", title: "First" },
    { key: "second", title: "Second" },
  ]);

  const renderTabBar = (props) => (
    <TabBar
      {...props}
      indicatorStyle={{ backgroundColor: "white" }}
      style={{ backgroundColor: "pink" }}
      indicator={IndicatorComponent}
      options={{
        articles: {
          accessibilityLabel: "Article",
          testID: "article",
          icon: IconComponent,
          label: LabelComponent,
          badge: BadgeComponent,
        },
        contacts: {
          accessibilityLabel: "Contacts",
          testID: "article",
          icon: IconComponent,
          label: LabelComponent,
          badge: BadgeComponent,
        },
        albums: {
          accessibilityLabel: "Albums",
          testID: "article",
          icon: IconComponent,
          label: LabelComponent,
          badge: BadgeComponent,
        },
      }}
    />
  );

  return (
    <TabView
      navigationState={{ index, routes }}
      onIndexChange={onIndexChange}
      renderTabBar={renderTabBar}
      scenes={{
        articles: FirstRoute,
        contacts: SecondRoute,
        albums: SecondRoute,
      }}
    />
  );
}
```

# Motivation

Currently, MaterialTopTabs are very slow when compared with using tab-view itself. We are changing TabBar props with every route change because we are looking up options by the currently focused route:

```ts
const focusedOptions = descriptors[state.routes[state.index].key].options;
```

This pattern prevents users to apply different options to tab screens (like different styles for each tab).

This API change will also make tab-view fit more into react navigation _ecosystem_.

There are multiple issues flagging material-top-tabs being slow: [#11047](https://github.com/react-navigation/react-navigation/issues/11047)

# Adoption strategy

This is a breaking change that would be released together with React Navigation v7.
