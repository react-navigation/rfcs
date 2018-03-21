- Start Date: 2018-03-21
- RFC PR: (leave this empty)
- React Navigation Issue: (leave this empty)

# Summary

Support the horizontal side transition in Android settings.

I see we took from android core these transitions - https://github.com/react-navigation/react-navigation/blob/3c3668c95295eb34e52ccdc46824753924be1396/src/views/CardStack/TransitionConfigs.js#L55


* Standard Android navigation transition when opening an Activity - http://androidxref.com/7.1.1_r6/xref/frameworks/base/core/res/res/anim/activity_open_enter.xml

        transitionSpec: {
            duration: 350,
            easing: Easing.out(Easing.poly(5)), // decelerate
            timing: Animated.timing,
        },

* Standard Android navigation transition when closing an Activity - http://androidxref.com/7.1.1_r6/xref/frameworks/base/core/res/res/anim/activity_close_exit.xml

        transitionSpec: {
            duration: 230,
            easing: Easing.in(Easing.poly(4)), // accelerate
            timing: Animated.timing,
        },


However I cannot find the "Standard Android More Settings" transition config which we see in settings page as a horizontal translate. And previous screen gets a black dimmer on it.

### Here is a screen recording of it:

* Here is HD - https://gfycat.com/ShyFlusteredChanticleer
* Here is gif:

     ![](https://thumbs.gfycat.com/ShyFlusteredChanticleer-size_restricted.gif)

### Here it is slowed down by 4x:

* Here is HD - https://gfycat.com/SecondhandIcyIberianemeraldlizard
* Here is gif:
    
    ![](https://thumbs.gfycat.com/SecondhandIcyIberianemeraldlizard-size_restricted.gif)


We should support this please.

# Basic example

Please see screen recordings.

# Motivation

True to native, as the core animations are.

# Detailed design

Transition config. And options to support it.

# Drawbacks

Why should we *not* do this? Please consider:

- No reasons not to match Android.

There are tradeoffs to choosing any path. Attempt to identify them here.

# Alternatives

Continue using the vertical transition.

# Adoption strategy

Add to docs.

# How we teach this

Docs.

# Unresolved questions

