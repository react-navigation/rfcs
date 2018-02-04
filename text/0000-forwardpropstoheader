# Reactive Header Control with `<this.props.navigation.ForwardPropsToHeader>`
## Problem
In the React lifecycle, all rendering should be done inside `render()`. But if a screen needs to control how a header is rendered, it cannot do so in the `render()` method. Instead, the Screen must use a third-party object to pass callbacks through. This does not scale well for complex headers.

It is possible to use 
```jsx
static navigationOptions = {
  header: (props: *) => {
    globalStateObject.setHeaderProps(props)
    return null
  },
}
render() {
  return (
    <View>
      <CloseUpGallery photos={photos}/>
      <ViewAlbumHeader
        onChangeText={console.warn}
        {...db.headerProps}
      />
      {/* Here is the body of the Screen */}
      <FooComponent/>
    </View>
  )
}
```
However, this leaves a lot to be desired. When using `headerMode: 'float'`, two headers will be displayed during the screen transition. The alternative on iOS is having the non-native look of `headerMode: 'screen'`.

## Desire
```jsx
class Screen extends React.Component {
  // ...
  render() {
    return (
      <View>
        <CustomHeader
          onChangeText={this.callback.bind(this)}
          editingMode={this.state.editingMode}
          headerLeft={this.props.navigation.defaultHeaderLeft}
          headerRight={this.props.navigation.defaultHeaderRight}
        />
        {/* Here is the body of the Screen */}
        <FooComponent/>
      </View>
    )
  }
}
```
## Without modifying the library
We can achieve this without modifying the library. Here's the new code:

```jsx
// index.js
const ModalStack = StackNavigator({
    Home: {
        screen: MainScreen,
    }, SlaveScreen: {
        screen: SlaveScreen
    }
}, {
    headerMode: 'float',
    onTransitionEnd: () => {
        db.setTransitionEnded(true)
    },
})
// state.js
export class State {
    transitionEnded = false
    headerProps = null
    setTransitionEnded(transitionEnded) {
        this.transitionEnded = transitionEnded
    }
    setCurrentHeader(header) {
        this.currentHeader = header
    }
    setHeaderProps(headerProps) {
        this.headerProps = headerProps
    }
    slaveScreen = {
        triggerEditMode: () => null,
        setEditModeCallback: function(callback) {
            this.triggerEditMode = callback
        },
    }
}
export let db: State = new State()
// screens/SlaveScreen.js
export default class CatchUpdateWrapper extends React.Component {
    constructor(props) {
        super(props)
        this.state = { __updated_props: {} }
    }

    componentDidMount() {
        db.setCurrentHeader(this)
    }

    componentWillUnmount() {
        db.setCurrentHeader(null)
    }

    render() {
        return <this.props.innerComponent {...this.props} {...this.state.__updated_props}/>
    }
}

class HeaderProxy extends React.Component {
    render() {
        return null
    }

    componentDidUpdate() {
        if (db.currentHeader !== null && db.transitionEnded) {
            db.currentHeader.setState({
                __updated_props: this.props,
            })
        }
    }
}
export class SlaveScreen extends React.Component {
    static navigationOptions = {
        header: (props: *) => {
            db.setHeaderProps(props)
            return <CatchUpdateWrapper innerComponent={CustomHeader}
                                       hideDuringTransition={true} {...props} />
        },
        headerBackTitle: null,
        headerTruncatedBackTitle: null,
        headerRight: <Button title={"Edit"} onPress={() => {
            db.slaveScreen.triggerEditMode()
        }}/>,
    }

    constructor(props) {
        super(props)
        this.state = { editMode: false }
        db.slaveScreen.setEditModeCallback(this.triggerEditMode.bind(this))
    }

    triggerEditMode() {
        this.setState({ editMode: true })
    }

    render() {
        return (
            <View style={{ backgroundColor: '#000', width: 1000, height: 1000 }}>
                <HeaderProxy
                    onChangeText={console.warn}
                    hideDuringTransition={true}  // residue of not cleaned up CustomHeader class
                    editStyle={this.state.editMode}
                    {...db.headerProps}
                />
                <Text style={{ color: "#FFF" }}>I'm the body!</Text>
            </View>
        )
    }
}
```

You can try a working example with this: (uses Expo, it works when ejected too)

```bash
git clone https://github.com/CrazyPython/reactive-navigation-demo.git
cd reactive-navigation-demo
yarn
react-native run-android
```

## The Proposal: Integrating this as a part of `react-navigation`

I want to be able to use `<HeaderProxy/>` in this way without maintaining my own global variables,  `<CatchUpdateWrapper/>`, and custom `navigationOptions`. It does not debounce

I propose a `<this.props.navigation.ForwardPropsToHeader/>`, managed by the Navigator that works out-of-the-box`<HeaderProxy/>`.

`<this.props.navigation.ForwardPropsToHeader/>` should:

- include the default props that react-navigation passes, unlike `<HeaderProxy/>`.

### Disadvantages

- The Header is not updated in the same render cycle as the screen.
- You have to specify the header class and initial props in `navigationOptions.header`.

## Notes

This is simply a draft. I would like to hear the community's ideas. 
