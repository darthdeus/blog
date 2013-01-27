---
layout: post
title: "Concatenated Properties"
date: 2013-01-27 18:18
comments: true
categories: core
---

As some of you might now, Ember provides you with something called
*concatenated property*. Their main use case is internal, which means
you are unlikely to have the need to use them in your own application.
There are some places in Ember where you might be surprised by how
things behave and this might be one of those. Let's start with an
example.

```javascript
App.UserView = Ember.View.extend({
  classNames: ["user"]
});

App.UserView.create().get("classNames") // => ["ember-view", "user"]
```

Now you might be asking, where is the `"ember-view"` coming from? Time
for another example

```javascript
App.DetailUserView = App.User.extend({
  classNames: ["more", "detail"]
});

App.DetailUserView.create().get("classNames") // => ["ember-view", "user", "more", "detail"]
```

This must be some sorcery! It seems that `classNames` aren't overwritten
in the subclass, but rather concatenated to the superclass' value of
that property. This works even when you overwrite it in an instance.

```javascript
Ember.View.create({ classNames: ["cat"] }).get("classNames") // => ["ember-view", "cat"]
```

A simple glance at the [`Ember.View`](https://github.com/emberjs/ember.js/blob/master/packages/ember-views/lib/views/view.js#L756) source code reveals it's secrets

```javascript
Ember.View = Ember.CoreView.extend({

  concatenatedProperties: ['classNames', 'classNameBindings', 'attributeBindings'],

  // more stuff
```

If this still doesn't make any sense to you, just go take a look at [the
tests for concatenated properties](https://github.com/emberjs/ember.js/blob/master/packages/ember-metal/tests/mixin/concatenatedProperties_test.js).
