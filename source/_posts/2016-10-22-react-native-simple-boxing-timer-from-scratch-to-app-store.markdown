---
layout: post
title: "React Native simple boxing timer - from scratch to the App Store"
date: 2016-10-22 14:44:35 +0700
comments: true
categories: reactjs, javascript, swift, appstore
hidden: true
---

Ever wondered how easy it is to create your first app for iOS? It isn't. Many pitfalls out there. 
I'll describe the process as it was actual at 2016 October. 

<!--more-->

# Develop with React Native

It is quite easy to [initialize React Native project](https://facebook.github.io/react-native/docs/getting-started.html) and start to tackle the code.
With provided CLI it will run nicely and have a hot refresh on the emulator, along with browser debug console.
Developing with React Native feels great in comparison with Phonegap I tried before 
([there is the link to the app I wrote as a result](https://play.google.com/store/apps/details?id=com.firfi.yobatimer))
In comparison with web React.js, instead of DOM components you use native components like <View/> and write a lot of inline styles. 
Otherwise, it's a familiar React JS app where you can use your favorite libraries like Bluebird, Moment.js, etc.
Of course, Redux/Mobx too. One exception - no libraries which are working with DOM. Because there's no DOM.
I've chosen Mobx for data handling and I think it was a very good solution for concrete application objectives.
It worked quite nice with settings screen. That's a topic that I'll write a different post about.

![Settings screen](/images/boxing-timer/settings.png "Boxing timer application settings screen")

React Native components which were used for this screen - View, Picker and Settings class.

## Version for old iOS

I wanted the app to be accessible as much as possible, including to third-world countries. 
By this reason, I included iOS 7 support to the app (iPhone 4s and later). It brings additional considerations: 
React Native version 0.33.1 was the last version where iOS 7 was officially supported. 
Gladly it wasn't too old (it was released just a month ago) so I've decided to go with it.
Of course, I've tried earlier versions initially, but they weren't building well in xCode. 

## Design, Material UI

All apps need a design. When you're a web developer without a visual design feeling, you just take a Bootstrap.
There's no Bootstrap in mobile apps world so I've turned to Google Material Design rulebook. 
First, I've decided with the color scheme. It had to be roughly the same I used for my previous timer app, so I've got 
an appropriate color combination from Material Design color guide. 

## Icons, buttons

Second, I needed some buttons. 
Library [React Native Material Kit](https://github.com/xinthink/react-native-material-kit) did the job.
Third, I've needed some icons. For the web, you just get fontawesome. 
Good news - they're available on [React Native too](https://github.com/oblador/react-native-vector-icons)

![Main screen](/images/boxing-timer/main.png "Boxing timer application main screen")

So now I had a design for my main screen.

## Sleep control, sounds  
  
Specifics of a timer application is that you need to play sounds and have a control over screen sleeping. 
For sounds, I've used [React Native Sound](https://github.com/zmxv/react-native-sound) library. 
It was quite old at the moment, but I haven't found anything better. It did its job though and let me play the sounds.

<div class="jumbotron">
    <p>Note that in order for some React Native libraries to work, you have to do additional settings in your iOS and Android projects.</p>
    <p>What is need to be done described in each library readme, but mostly running script `react-native link library-name` does the job automatically.</p>
    <p>This is actual for Material Kit, Icons lib, React Native Sound and screen sleep control lib.</p>
</div>

For screen sleep control I used [react-native-keep-awake](https://github.com/corbt/react-native-keep-awake). I do `KeepAwake.activate()` whenever timer is running and `KeepAwake.deactivate()` whenever timer is stopped.
This way, the user wouldn't experience screen sleep in the middle of training.

## Rotation control, responsive font size

The hard part was the reactive font size change, depending on screen rotation. 
Also, you can't use the pixels there unless you define the sizes for each Apple device.
Googling for screen rotation detection gave me a couple of deprecated and non-working libraries, 
and the official React Native docs were quite vague on this matter. Eventually, in depths of Stack Overflow I found The Answer and now I sharing this Secret with you:
use View's onLayout callback!

```
<View onLayout={(event) => {
    this.setState(this.calculateWH(event));
}}>
...
```

where calculateWH is defined this way:

```
calculateWH(event) {
    const l = event ? event.nativeEvent.layout : Dimensions.get('window'); // for some reason, it doesn't always give the event
    return {
        vw: l.width / 100,
        vh: l.height / 100
    };
}
```

so now you have vw and vh for reactively changing your fonts size and also have rerendered on screen rotation. 
Don't forget to import Dimensions from 'react-native' package.

# App store

I found this part most time-consuming, in comparison with development itself.

## App icons

First, you need a nice icon. 
I just fetched a couple of free ones from the Internet, glued it together in Photoshop and added a gradient background. 
Another thing you'd need is a lot of different sizes for the icon. 
I just uploaded my icon to one of free icon resizing services and got it in different sizes.

![App Icon](/images/boxing-timer/icon.png "App Icon")

## App screenshots, UX testing

Second, in order for your app published, you need screenshots. 
Usually, a couple is enough, but I wanted screenshots for all devices. 
As a result, with two app screens, I needed around 12 screenshots on 6 emulators. 
People who add internationalization in their app needs screenshot packs for each language. 
So if I will be supporting let's say 12 languages I'd need 144 screenshots. 
Better to prepare for that and automatize the process.
For this and for beta testing / deployment I used [fastlane](https://github.com/fastlane/fastlane).
Fastlane is a set of tools for automating several mobile development chores. 
It spared me a couple of clicks. Well, actually a lot of clicks.
I used it's `snapshot` tool for taking screenshots. Before really using it, I needed to set up the tests in xCode. 
I did everything by the manual but there was an issue: elements didn't recognize testId property. 
So I've decided not to go with it, and record test with manual recording feature in xCode.
 
Resulting test I've got:

```
func testExample() {
      
      let app = XCUIApplication()
      snapshot("0Timer")
      let startButton = app.buttons[" Start  "]
      startButton.tap()
      let stopButton = app.buttons[" Stop  "]
      stopButton.tap()
      app.tabBars.buttons["Settings"].tap()
      snapshot("1Settings")
      
}
```
      
After running it in snapshot tool, I've got nice 12 screenshots. Now I can use it for internationalization too.

## Beta, signing, release   
      
It was the time to check the app in a real environment. 
By default, fastlane is set up for TestFlight, 
but TestFlight doesn't work on old iOS that I had at the time, so I've reconfigured it to TestFairy service.
After checking on the device, I've released it with fastlane deploy. This process has its moments (i.e. signing) 
but generally, fastlane automatize most of the chores.