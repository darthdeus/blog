---
layout: post
title: "State Manager and Friends - part 1"
date: 2013-01-27 19:23
comments: true
categories: Ember
---

Since state management is such a huge part of Ember.js it desrves a
dedicated article. I'm not going to explain the old router which used
`Ember.StateManager` to do it's bidding. Those days are over and we
should all be moving towards the v2 router (or v2.2 so to speak).
Instead we're going to go deep into the `Ember.StateManager`.

In the general concept, state manager is basically some object which
manages states and the transitions between them, thus representing a
finite state machine.

Let's say we have a `Post` which can be in two states, `draft` and
`published`. It begins it's life as a `draft` and when we `publish` it,
it should send out a notification email. The way Ember would handle this
is that it would assign a `Ember.StateManager` instance to the `Post`
instance and have that manage it's state (that's not exactly true in
Ember Data, but we'll get into that).

For now let's just say that this is the code we have

```javascript
PostManager = Ember.StateManager.extend({
  states: {
    draft: Ember.State.create(),
    published: Ember.State.create()
  }
});

Post = Ember.Object.extend({
  title: null,
  init: function() {
    this.set("stateManager", PostManager.create());
    this._super();
  }
});
```

This gives us a really basic implementation. I'm setting the
`stateManager` property in the `init` function to avoid sharing the
instance across multiple `Post` instances. I'll explain this in a
followup article, for now just remember that if you need to set a
property to an object instance, you have to do that in the `init`
function, not directly like `stateManager: PostManager.create()`.

OK, we are now ready to list all of the states a `Post` can have.

```javascript
post = Post.create();
post.get("stateManager.states"); // => { draft: ..., published: ... }

post.get("stateManager.currentState"); // => null
```

We forgot to say which of the states should be the default. Let's
do that.

```javascript
PostManager = Ember.StateManager.extend({
  initialState: "draft",
  states: {
    draft: Ember.State.create(),
    published: Ember.State.create()
  }
});
```

From now every single post we create will be a `draft`

```javascript
post = Post.create();
post.get("stateManager.currentState.name"); // => "draft"
```

And we can also make it transition into another state

```javascript
post = Post.create();
post.get("stateManager").transitionTo("published");
post.get("stateManager.currentState.name"); // => "published"
```

But `Ember.StateManager` can do more than that. We can hook into both
`enter` and `exit` events on each state and do some magic! Let's
redefine our state manager as this

```javascript
PostManager = Ember.StateManager.extend({
  initialState: "draft",
  states: {
    draft: Ember.State.create(),
    published: Ember.State.create({
      enter: function() {
        console.log("post was published");
      }
    })
  }
});

post = Post.create();
post.get("stateManager").transitionTo("published");
// console prints "post was published"
```

Understanding how this class works is essential for any Ember developer,
as it is being used in almost every part of the framework. We'll take at
some specific examples in the second part of this artcile.
