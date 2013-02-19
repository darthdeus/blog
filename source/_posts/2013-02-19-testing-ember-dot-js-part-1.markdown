---
layout: post
title: "Testing Ember.js - part 1"
date: 2013-02-19 19:05
comments: true
categories: Testing
---

Ever since I saw the _testing_ slides from EmberCamp I was thinking
about testing. Up until now I've been using Capybara which is really
really really slow.

But @joliss mentioned this thing called `Ember.testing` which should
automagically fix all of the async problems which make tests ugly, such
as waiting for the application to initialize and finish routing.

In its essence `Ember.testing = true` disables the automatic runloop,
which gives you the control to manually schedule asynchronous operations
to happen in a one-off runloop via `Ember.run`.

`Ember.run` will run the given function inside a runloop and flush all
of the bindings before it finishes, which means you can render a view
inside `Ember.run` and check the DOM right after that. Here's an example
from the `Ember.View` tests

```javascript
view = Ember.ContainerView.create({
  childViews: ["child"],

  child: Ember.View.create({
    tagName: 'aside'
  })
});

Ember.run(function(){
  view.createElement();
});

equal(view.$('aside').length, 1);
```

As you can see the `view.createElement()` happens inside the runloop
scheduled by `Ember.run` which will return only after the view was
completely rendered and all bindings flushed.

Let's take a look at a [complete example](http://jsbin.com/ixupad/59/edit)
and take it apart step by step

```javascript
// Testing mode disables automatic runloop
Ember.testing = true;

// Creating an application normally happens async,
// which is why we have to wrap it in Ember.run
Ember.run(function() {
  App = Ember.Application.create();
});

App.Router.map(function() {
  this.route("home", { path: "/" });
});
  
App.Store = DS.Store.extend({
  revision: 11,
  adapter: DS.FixtureAdapter.extend({
    // This will make the FixtureAdapter do everything synchronously
    // instead of using setTimeout, which is vital because setTimeout
    // happens outside of the runloop.
    simulateRemoteResponse: false
  })
});

App.User = DS.Model.extend({ name: DS.attr("string")});
App.User.FIXTURES = [ { id: 1, name: "brohuda" }];

App.HomeRoute = Ember.Route.extend({
  model: function() {
    return App.User.find(1);
  }
});

// Enabling Ember.testing will also disable automatic initialization,
// which forces us to initialize manually
Ember.run(function() {
  App.initialize();
});

// In real life this would be an assertion,
// here we'll just check if everything is rendered at this point in time.
$("p strong").append($("h2").text());
```

Take the example apart, play with it and try to figure out what works
and what doesn't :)

If you see

```
assertion failed: You have turned on testing mode, which disabled the run-loop's autorun.
You will need to wrap any code with asynchronous side-effects in an Ember.run
```

it means that you forgot to wrap something in `Ember.run`. I hope this
is a good enough introduction. In one of the upcoming articles we'll
take a look at simple Ember application and try testing it with a
full featured testing framework.
