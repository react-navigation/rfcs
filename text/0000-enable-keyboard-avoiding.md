- Start Date: 2013-07-23
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Add enableKeyboardAvoiding to navigationConfig.

# Basic example

```
createStackNavigator({...}, { enableKeyboardAvoiding: true });
```

# Motivation

keyboardVerticalOffset is required if the KeyboardAvoidingView does not start from the top of the screen.
For createStackNavigator or createMaterialTopTabNavigator there will headers and tabs.
If developers need a KeyboardAvoidingView inside StackNavigator/TopTabNavigator, they cannot provide the offset.

I've seen most of the projects use magic numbers (like 64 for ios, 54+StatusBar.currentHeight for Android, and special case for iPhone X, iPad, horizontal iPhone X...) to set the keyboardVerticalOffset. Which is not safe.

If you are thinking why not put the KeyboardAvoidingView outside the StackNavigator. The answer sometimes we cannot, for example below structure:
```
<BottomTabNavigator>
  <StackNavigator />
</BottomTabNavigator>
```
We don't want bottom tabs to be "KeyboardAvoiding", we only want the StackNavigator to be "KeyboardAvoiding"
```
<BottomTabNavigator>
  <KeyboardAvoidingView>
    <StackNavigator />
  </KeyboardAvoidingView>
</BottomTabNavigator>
```

# Detailed design

The easiest way is to make StackNavigator/TopTabNavigator let them accept a parameter 'enableKeyboardAvoiding'. If enableKeyboardAvoiding is true, the StackNavigator and TopTabNavigator will warp themself with a KeyboardAvoidingView

An POC PR:
https://github.com/react-navigation/react-navigation/pull/4290

# Drawbacks

`behavior="padding"` in KeyboardAvoidingView might have to be hardcode.

# Alternatives

Let StackNavigator/TopTabNavigator provide real-time percise header height.

# Adoption strategy

No breaking change, enableKeyboardAvoiding will be false by default and no KeyboardAvoidingView will be created.

# How we teach this

For React Navigation users, they don't need to know details. We can simply say `enableKeyboardAvoiding` can make the Navigator "eKeyboardAvoiding"

# Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still TBD?