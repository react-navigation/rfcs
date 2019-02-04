* Start Date: 2018-01-24
* RFC PR: (leave this empty)
* React Navigation Issue: (leave this empty)

# Summary

Right now when you press back in a tabs navigator, it goes back to home tab. I'd expect it to go to the previous tab I had visited instead. It's a bit tricky for tabs, and different apps handle it differently.

Currently two back behaviour options are implemented, `initialRoute` and `none`.

Here's a summary of different patterns used in various apps. I think we need to implement the behaviour used by instagram app.

# Existing patterns

NOTE: all code here are just for explaining the intent and have nothing to do with the actual router code.

## Don't handle back button

I don't like this pattern very much, but it's useful if you have a huge number of tabs. Though a huge number of tabs is not a good UX either.

**Used by:** YouTube

## Go to the home tab

This is the current behaviour.

```js
handleBack() {
  if (currentIndex !== 0) {
    const previousTab = getTabWithindex(0);
    return getStateForTab(previousTab);
  }
}
```

**Used by:** Facebook, Twitter

## Go to the tab which comes before the current

It's very simple, but probably not very intuitive.


```js
handleBack() {
  if (currentIndex > 0) {
    const previousTab = getTabWithindex(currentIndex - 1);
    return getStateForTab(previousTab);
  }
}
```

**Used by:** NA

## Keep a history of tabs

Sounds ok, but probably not the best UX. User can keep tapping tabs again and again, so you have to keep pressing back a lot of times before the parent navigator gets to handle the back button.


```js
onNavigate(route) {
  history.push(route);
}

handleBack() {
  if (history.length) {
    const previousTab = history.pop();
    return getStateForTab(previousTab);
  }
}
```

**Used by:** NA

## Keep a history of tabs, and de-duplicate

A modified version of the previous approach. We keep an array of previous tabs, but remove the duplicate tab from the stack if it existed previously. So you never visit the same tab twice when pressing back.

It's not very accurate because you don't go back to the exact tabs you came from. But nobody really remembers more than last 1-2 screens, so it should be fine.

```js
onNavigate(route) {
  const currentIndex = history.findIndex(r => r.key === route.key);
  if (currentIndex > -1) {
    history.splice(currentIndex, 1);
  }
  history.push(route);
}

handleBack() {
  if (history.length) {
    const previousTab = history.pop();
    return getStateForTab(previousTab);
  }
}
```

**Used by:** Instagram

# Proposal

It makes sense to implement the following behaviours:

```js
type backBehaviour =
  | 'none' // don't handle the back button
  | 'history' // go back to previous route in history (with dedupe)
  | 'order' // go back to the tab with previous index (corressponds to the `order` config option)
  | 'initialRoute' // go back to the intial route
```
