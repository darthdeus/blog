---
layout: post
title: "render, control, partial, view, template"
date: 2013-02-10 21:29
comments: true
categories: Templates
---

There are many ways one can DRY up templates when using Ember.js, it all
depends on what you're trying to achieve.

## partial && template

`{% raw %}{{partial "foo"}}{% endraw %}` will take a template
`foo.handlebars` and insert it without changing anything, which is
exactly the same as in Rails. There are no views created, no scope
changes, it just inserts the template right there.

`{% raw %}{{template}}{% endraw %}` isn't really meant to be used anymore, so use
`{% raw %}{{partial}}{% endraw %}` instead.

## view

`{% raw %}{{view App.FooView}}{% endraw %}` will create an instance of
`App.FooView` (with `foo.handlebars` template unless you override the
name) and insert it in place. You can bind on properties of the view,
such as `{% raw %}{{view App.FooView contentBinding="foobar"}}{% endraw %}`,
or just specify a property directly `{% raw %}{{view App.FooView class="foobar"}}{% endraw %}`.

This is a low level thing and is mostly used to instantiate simple
views, such as `{% raw %}{{view Ember.TextField valueBinding="name" class="username"}}{% endraw %}`

## render && control

Most of the time you're looking to use `{% raw %}{{render}}{% endraw %}` instead of
`{% raw %}{{view}}{% endraw %}` as it offers better means of
abstraction. `{% raw %}{{render "foo" bar}}{% endraw %}` will create a
`App.FooController` and bind it's content to `bar`.  It also creates a
`App.FooView` and renders a `foo` template.

One drawback is that `{% raw %}{{render}}{% endraw %}` **can not be called multiple times on
a single route**. If you need a self sustainable widget which can be
created any number of times you want, you're looking for `{% raw %}{{control}}{% endraw %}`
which has exactly the same effect as `{% raw %}{{render}}{% endraw %}`, but it will have a new
controller instance every time you call it, while `{% raw %}{{render}w{% endraw %}` uses a
singleton controller.

Please keep in mind that `{% raw %}{{control}}{% endraw %}` is currently under heavy
development and will probably change soon, because of the high number of
issues there are with it.
