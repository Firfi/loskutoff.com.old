---
layout: post
title: "Writing a bot for online browser game with PhantomJS"
date: 2014-03-28 12:18:18 +0400
comments: true
categories: phantomjs, games
---

Phantomjs could be used for frontend functional testing.

Sometimes it could be used for [rich web application html snapshots for search engine crawlers](http://www.yearofmoo.com/2012/11/angularjs-and-seo.html)

Let's use it for cheating.

In this article I'll describe creating process of very simple html+js online game bot that running headlessly 24/7 on remote server (I'll use [Heroku](https://herokuapp.com/)).

<!--more-->

I will assume that reader is familiar with [phantomjs installation](http://phantomjs.org/download.html). In this example I'll use online [ZPG](http://ru.wikipedia.org/wiki/Zero_Player_Game) [Godville](http://godvillegame.com/).

## Login and main game logic

There's basically two lifecycle parts that need to be maintained: authentication and main lifecycle.

First and most easy way to do authentication is just fill login/pass fields on each run. It is not the best way because game could use captcha after several requests (and Godville is using it) so let's plug in cookies:

`phantomjs --cookies-file=cookies.txt bot.js`

Where bot.js is main script file. File cookies.txt will be created automatically and will remain in project folder after you stop application. It will be used on next run making your bot automatically authenticated and redirected to main game page by server.

Detecting if bot is logged in depends on game mechanic. For Godville it could be done by URL checking after main game page request (/superhero). If bot is not logged in, it will be redirected to /login page.

```javascript
// try to open /superhero main game page
page.open('http://godvillegame.com/superhero', function() {
  if (page.url == account.godvilleUrl + '/login') {
    page.evaluate(function() {
      $('#username').val('login');
      $('#password').val('password');
      $('.input_btn').click();
    });
  } else {
    // run main game logic
  }
}
```

And there we are! Bot is logged in and obtaining information, you could write game logic on javascript inside evaluate() callback like you're doing it in browser console. Everything depends on your imagination now. Phantomjs works with websockets so it see dynamic game DOM changes.

For debugging you could use page.render function that makes page screenshot:

`page.render('example_login.png');`

and also write console.log output. By default it will not be printed in application console so additional code needed:

```javascript
page.onConsoleMessage = function(msg) {
  console.log(msg); // or write it in file if you prefer as it done in my repository code
};
```

## Deploy on Heroku

Create application-specific [Procfile](https://devcenter.heroku.com/articles/procfile): `worker: phantomjs --cookies-file=cookies.txt bot.js`

Assuming you have [Heroku toolbelt](https://toolbelt.heroku.com/) installed already, do `heroku create`, then add and commit Procfile and script files to git initialized by this command git repo. Add necessary for phantomjs buildpack:

```bash
heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
heroku config:set PATH="/usr/local/bin:/usr/bin:/bin:/app/vendor/phantomjs/bin"
```

and then push your changes: `git push heroku master`.

Make sure that your worker is running: `heroku ps:scale worker=1`

That's it. And there's [my bot repository](https://github.com/Firfi/gvchatbot)


