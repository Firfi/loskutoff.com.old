---
layout: post
title: "Using ReactJS as a MeteorJS view layer"
date: 2015-04-17 19:55:17 +0700
comments: true
categories: meteorjs, reactjs
---

* Update * Nowadays Meteor provides React support OOB so this article would be useful if you're moving from old Blaze system.

Here I'll share actual on this day way to use ReactJS with Meteor. The basic setup itself rather simple,
so after shortly describing it, I'll touch a matters of using third-party React packages.

<!--more-->

## Using React components in Meteor applications

To basic setup React and jsx transformation pipeline it is enough to install this package https://github.com/reactjs/react-meteor

Also, it offers you a special mixin for tying together Meteor reactive sources and your component state, but I found it problematical to use in current state
(i.e. your getMeteorState function will be executed everytime your main state is changed with setState).

It can sometimes be annoying, but in general, if you don't mind it, you can use it as well.

Another option can be using subscriptions on componentWillMount and unsubscribe it in componentWillUnmount like this:

~~~

componentWillMount: function() {
    this._cancelSubscription = Tracker.autorun(...);
}

componentWillUnmount: function() {
    this._cancelSubscription();
}

~~~
{:lang="js"}

Sometimes I find Tracker.autorun not enough, and currently use some service for data fetching, abstracting it and exposing reactive stream
backed up by BaconJS library (it could be rxJS as well). It looks almost the same as in example above; you get subscription in componentWillMount
and unsubscribe in componentWillUnmount.

Another possible solution is using any Flux flavours, and listen to Storages.

One another problem I faced with is using React in current project that written with Blaze already with some of Meteor routing libraries (I use iron router)
React components doesn't know anything about your route change or Blaze template destruction and stays on your screen.

To address this issue, I wrap components in Blaze templates, hooking template create/destroy livecycle callbacks:

~~~
// some code omitted, I left only relevant to issue one
_ReactUtils.createClass = function(opts) {
  var templateName = opts.templateName;
  var templateClass = new Template(
    templateName,
    function() {
      return new HTML.DIV;
    }
  );
  Template[templateName] = templateClass;
  var Component = React.createClass(opts);
  templateClass.onRendered(function() {
    var template = this;
    var data = this.data || {};
    var c = React.createElement(Component, data);
    c._meteorTemplate = template;
    template._reactComponent = React.render(c, template.firstNode);
  });
  templateClass.onDestroyed(function() {
    var template = this;
    React.unmountComponentAtNode(template._reactComponent.getDOMNode());
  });
  return Component;
};
~~~
{:lang="js"}

With this approach, I can just replace my Blaze components one by one, i.e.

~~~
<template name="indexPage">
    {{> someBlazeComponent1}}
    {{> someReactComponent2}}
</template>
~~~
{:lang="html"}

And when I'm ready to switch another component to React,

~~~
<template name="indexPage">
    {{> someReactComponent1}}
    {{> someReactComponent2}}
</template>
~~~
{:lang="html"}

If you start a new project, you can use some of React routers I suppose. I didn't investigated this matter yet.

## Using third-party React packages in Meteor

Huge problem I faced with was lack of pre-build step in packages as well as impossibility to use browserify on client-side.

Many React libraries use require() and doesn't have build distributive available in repository, so if there's no such Meteor package you have to build it yourself.

What I do usually is use webpack, excluding React from build (as we'll depend on react-meteor package in our package-wrapper).

~~~
module.exports = {
    entry: "./index.js",
    output: {
        path: __dirname,
        filename: "dist/meteor-dist.js",
        libraryTarget: "umd",
        library: "MyLibrary"
    },
    externals: {
        react: 'React'
    }
};
~~~
{:lang="js"}

Here, 'react' library would be excluded and will be referenced as a React on global object that we have already,
and library itself will be exported as MyLibrary.

Then you can run webpack and it'll create distributive that can be used in Meteor package description:

~~~
Package.onUse(function(api) {
  api.use('reactjs:react@0.2.1', ['client', 'server']);
  api.addFiles('dist/meteor-dist.js', ['client', 'server']);
  api.export('MyLibrary', ['client', 'server']);
});
~~~
{:lang="js"}

However, that wouldn't work. Reasons are described in my previous post, so there's just a solution: in your dist, manually
create an object with React from global scope, pass it into dist function, then extract MyLibrary from it back to the global scope.

It'll look like that:

~~~
var meteorHack = {
  React: React
};
(function webpackUniversalModuleDefinition(root, factory) {
	...
})(meteorHack, ...) // here, meteorHack instead of this

...

MyLibrary = meteorHack.MyLibrary;

~~~
{:lang="js"}

Pay attention that we pass `meteorHack` object instead of `this` as it wouldn't work as expected with `this`.

Now you can check the library locally with a symlink and publish to Meteor repository.

My last recommendation here will be making it a fork of original library rather than creating own repository, this way you can
pull updates and generally have things tracked better.