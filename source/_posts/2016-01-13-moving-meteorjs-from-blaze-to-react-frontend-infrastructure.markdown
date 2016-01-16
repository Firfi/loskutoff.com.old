---
layout: post
title: "Moving MeteorJS from Blaze to React frontend infrastructure"
date: 2016-01-13 21:53:19 +0300
comments: true
categories: meteorjs, blaze, javascript, react
published: false
---

Making rich frontend SPA is always challenging task, especially when you don't know what right tools to choose.

In this post I'll propose one of the best at the moment tools and a way to integrate it with MeteorJS infrastructure.

One big project which I'm working on currently is [Fiddlequest](https://fiddlequest.com/), tool for musical students, their parents and teachers.

It have lot of dynamic frontend components with complicated requirements. They should be fast, reactive and real-time.

It was a real challenge to write this project on Blaze, when I didn't know how to integrate React there.

When I found a good way, it was already full of Blaze templates which was not easy to maintain.

Simply rewriting wasn't an option as it'd lead to stopping developing new features, so I found it very handy to rewrite it component by component from time to time, not stopping introducing new features.

My objective here is to show reader why and how to move Meteor project from Blaze to React.

<!--more-->

## Why should I do it

### 1. Blaze is deprecated

Main reason is simple: Meteor Development Group announced [abandonment of Blaze](https://forums.meteor.com/t/next-steps-on-blaze-and-the-view-layer/13561).

As a side note, it isn't the first time MDG changing template engine: before Blaze was [Spark](http://info.meteor.com/blog/meteor-080-introducing-blaze).

However, I'm pretty sure that this time new template engine knowledge will be really useful for you as React don't going anywhere for long time from now
(on contrary to Angular which in it's place being substituted by completely new and different [Angular 2](https://angular.io/), keeping only a name).

### 2. React is really better for you

- It working with state right way. You don't have piece of data lingering copied over in several different components.
It _makes_ you keep your master data in one place, handling to components only projection of this data. See [official React docs](https://facebook.github.io/react/docs/thinking-in-react.html)

- It have nice component system, making you write independent components which are really easy (I did it many times and happy so far) to move around your application UI should requirements change.

I.e. I have multi step signup flow with client+server-side validations. One day, I got a requirement from client to have it working not only on the /signin page, but also on landing page through modal dialog. It was as easy as just writing <SignFlow ... /> inside my modal component body!

- It have strong community support, meaning there's lots of good quality components which you can just plug in in your application and use. On contrary, Blaze components are rare and have poor code quality overall, crawling with bugs. Good riddance!



