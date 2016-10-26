---
layout: post
title: "Making browserify.js working with Meteor application"
date: 2015-06-16 17:33:52 +0300
comments: true
categories: meteorjs, reactjs, browserify
---

* Update * This article is completely outdated as Meteor supports npm packages out of the box now.

In previous [post](http://www.loskutoff.com/blog/using-reactjs-as-a-meteorjs-views/)
I described how to wrap existing npm (especially React-related) packages into Meteor packages to use them in your application.

This way was quite hacky and was involving manual build modification, as well as forking existing libraries and committing build distributions in those forks.

I choose this way as there wasn't a way to [browserify](http://browserify.org/) packages on meteor application build step.

That really delighted me when I found that Eli Doran (https://github.com/elidoran) recently wrote custom [build plugin](https://github.com/elidoran/cosmos-browserify)
that finally enable us to include browserifyed React packages without writing those terrible wrappers!

From now I deprecate my libraries that I ported for [Atmosphere](https://atmospherejs.com/) package manager and recommend to those who still use wrapped packages move to Browserify with cosmos-browserify library help.

Below I'll describe how I moved my project from wrapped atmosphere packages to browserifyed components with [cosmos-browserify](https://github.com/elidoran/cosmos-browserify).

<!--more-->

## Migrating to 'right way' of libraries usage

Let's presume we have `firfi:meteor-react-bootstrap` package installed. It exports `ReactBootstrap` global for us.

Only one line of code that makes that possible is `firfi:meteor-react-bootstrap` in .meteor/packages. It was added either manuall or with `meteor add firfi:meteor-react-bootstrap` command.

Now, let's remove it and substitute with browserifyed version.

### Adding browserifyed library to our Meteor application

I'll go with local package way. This way, all browserifyed libraries will be available in all parts of your code as local packages being loaded before all other app code.

First let's create a package in folder packages. Let's call it `client-deps`, so final created path would be `packages/client-deps`.

To make Meteor identify it as a package, we should add file package.js there:

~~~
// packages/client-deps/package.js

Package.describe({
  name: 'client-deps'
});

Npm.depends({
  'react-bootstrap': '0.23.3', // this library will be browserifyed later
  react : '0.13.3' // we'll also add react itself so react-bootstrap doesn't feel loneliness
});

Package.onUse(function(api) {
  api.use(['cosmos:browserify@0.3.0']); // insert the latest version here
  api.addFiles(['app.browserify.js']); // we'll create this file below
  api.export(['ReactBootstrap', 'React']); // it's exported in app.browserify.js
});

~~~
{:lang="js"}

To tell which libraries we'll browserify we need a .browserify file which will be handled by cosmos-browserify build plugin.

Cosmos-browserify will unwrap all require()'d libraries in browserify bundles.

~~~
// packages/client-deps/app.browserify.js
React = require('react/addons');
ReactBootstrap = require('react-bootstrap'); // export it 'globally' so Meteor is happy and can `api.export` it in package.js
~~~
{:lang="js"}

And, to make it all work, you should finally install cosmos-browserify itself: `meteor add cosmos:browserify`
and point your meteor app that it should use your local package: `meteor add client-deps`

Don't forget to delete old libraries that were exporting your `ReactBootstrap` and `React`. In my case it was
`firfi:meteor-react-bootstrap` (that was depending on old version of https://github.com/grovelabs/meteor-react with React exported).

If after this you still get some react warnings it is quite possible you still have some libraries that exporting or using internally it's own React versions. In that case just browserify it too,
as I had to do with `reactjs:react`, `firfi:griddle-react`, `dgellow:react-loader` and `firfi:tcomb-form`.

With definitely more boilerplate you get latest versions of all npm libraries existing without third-party wrappers and also have it included 'right way'. Happy meteor coding!
