---
layout: post
title: "Controller, ObjectController and ObjectProxy"
date: 2013-01-27 19:24
comments: true
categories: controller
---

When you first come to Ember, you'll soon stumble upon three things:

- `Ember.Controller`
- `Ember.ObjectController`
- `Ember.ArrayController`

For some people (including me) it is not very clear what's the
difference between the first two.

`Ember.Controller` is just a plain implementation of
`Ember.ControllerMixin`, while `Ember.ObjectController` is a subclass of
`Ember.ObjectProxy`. This is a huge difference! Let's take a look at how
`Ember.ObjectProxy` works, and as always starting with a code sample
([taken from the excellent source code documentation](https://github.com/emberjs/ember.js/blob/master/packages/ember-runtime/lib/system/object_proxy.js#L35-L50)).

```javascript
object = Ember.Object.create({
  name: "foo"
});

proxy = Ember.ObjectProxy.create({
  content: object
});

// Access and change existing properties
proxy.get("name") // => "foo"
proxy.set("name", "bar");
object.get("name") // => "bar"

// Create new "description" property on `object`
proxy.set("description", "baz");
object.get("description") // => "baz"
```

There is really no magic. In the basic usage, `Ember.ObjectProxy` will
delegate all of it's unknown properties to the `content` object, with
one exception.

If we try to set a new property on a proxy while it's content is
undefined, we will get an exception.

```javascript
proxy = Ember.ObjectProxy.create();
proxy.set("foo", "bar"); // raises the following exception
```

```
Cannot delegate set('foo', bar) to the 'content' property
of object proxy <Ember.ObjectProxy:ember420>: its 'content' is undefined.
```

I've stumbled upon this in one scenario, where I didn't set content for
my `ObjectController`, but I tried to modify one of it's properties.
Raising the exception is a good example of failing fast, rather than
silently swallowing errors.

This being said you should almost always use `Ember.ObjectController`
over `Ember.Controller`, unless you know what you're doing :)
