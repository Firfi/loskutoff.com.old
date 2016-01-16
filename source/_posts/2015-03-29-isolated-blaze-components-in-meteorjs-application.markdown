---
layout: post
title: "Isolated Blaze components in MeteorJS application"
date: 2015-03-29 15:28:51 +0700
comments: true
categories: meteorjs, blaze, javascript
---
This article is an attempt to describe non-common in Meteor community approach of handling Blaze templates as isolated components.
There's great blank space on this topic I found trying to implement it so code below would be result of my experiments and observations.
I use this approach in my current project.

*Update 31 Marth* @Ben's Comment to this article proposed a nice @meteorhacks library [Flow-components](https://github.com/meteorhacks/flow-components)
that does thing by same principles I described here. 

<!--more-->

* * *

I implemented my last project with [MeteorJS](https://www.meteor.com/) framework.
Though it gives nice development speed, it also have many architectural drawbacks.
I wasn't particularly happy with [Blaze](https://www.meteor.com/blaze) template engine.
As I come from Angular world, I got used to wrap parts of my application frontend in isolated modules, bunch of code,
including js, html and (!) css.
Structure is something like described in [Johnpapa's Angular Styleguide](https://github.com/johnpapa/angular-styleguide#style-y152).
In Angular, I used directives for that so I can just drop some directive in any place of application, honouring it's inner API that expressed through directive parameters.
Trying to write something like that for Blaze I found it rather cumbersome, as I want from component next functionality:

- 1) Parameters with dual binding that are passed explicitly
- 2) Event system that gives possibility pop up events from components

I was able to resolve 1) and 2) like this:

- For 1), parameters with dual binding, I was just passing [ReactiveVars](https://atmospherejs.com/meteor/reactive-var) links into component like this:
- For 2), I used EventEmitter. Option is to just use callbacks.

~~~
// client/myTemplate.js
'use strict';

var templateClass = Template.myTemplate;

templateClass.onCreated(function() {
    // set initial state
    var template = this;
    template.data = template.data || {}; // sic! *1
    template.state = {
        myDualBindingVar: new ReactiveVar('default'), // sic! *2
        myEventEmitter: new EventEmitter(),
        parentEmitter: template.data.eventEmitter // now I can get parent events
    };
});

templateClass.helpers({
    myDualBindingVar: function() { // if I want dual binding
        return Template.instance().state.myDualBindingVar; // sic! *3
    },
    myDualBinding: function() { // if I want to just pass this variable in component on initialisation or if I want to draw in in Blaze template of current component
        return Template.instance().state.myDualBindingVar.get();
    },
    eventEmitter: function() {
        return Template.instance().state.myEventEmitter;
    }
});
~~~
{:lang="js"}

~~~
<!-- client/myTemplate.html -->
<template name="myTemplate">
    <div class="my-template"> <!-- root element with class corresponding to component name -->
        {{> mySecondTemplate eventEmitter=eventEmitter myDualBindingVar=myDualBindingVar}}
    </div>
</template>
~~~
{:lang="html"}

~~~
// client/mySecondTemplate.js
'use strict';

var templateClass = Template.mySecondTemplate;

mySecondTemplate.onCreated(function() {
    // set initial state
    var template = this;
    template.data = template.data || {};
    template.state = {
        myDualBindingVar: template.data.myDualBindingVar,
        parentEmitter: template.data.eventEmitter
    };
    template.autorun(function() {
        var myDualBinding = template.state.myDualBindingVar.get();
        // do stuff with parent var usage
    });
});

mySecondTemplate.events({
    'click .someButton': function() {
        var emitter = Template.instance().state.parentEmitter;
        emitter.emit('someButtonClicked');
    }
});

~~~
{:lang="js"}

Here I'll explain decisions that I made in code above.

*1: I write `template.data = template.data || {};`{:lang="js"} in each component, as current Blaze approach is just left it undefined if there's no parameters given.
I'm sure they have the reasons doing this. But these reasons make me do additional 'if/else'-like check like that.

*2: `ReactiveVar` is a package that you have to install additionally. Most of tutorials I found use just Session API. Though it would work, I strongly disapprove this approach as we're going back to 90'th global state with that.
Additionally, it you have several components on page, i.e. login widget on page itself and login widget in popup, using Session will be a problem as it will cause names clash issue.

*3: To get current template instance that we set up in onCreate I use Template.instance() call. That's right, it's call on singleton that returns you current template instance. I didn't found any better way doing this.
Also, getting current template instance doesn't seems very popular thing too (because meteor people store everything in Session I presume) so before I started investigating this issue there wasn't real good answer 'how to get current template instance' on StackOverflow (It was documented officially though).

* * *

In next article that about to be written I'll describe another solutions for component-like frontend architecture, including Angular and ReactJS.
Also, as I started to use ReactJS instead of pure Blaze lately, I want to describe how to set up and use it, as there seems to not enough information in community how to do it.