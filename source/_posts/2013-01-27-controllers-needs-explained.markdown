---
layout: post
title: "Controller's Needs Explained"
date: 2013-01-27 11:53
comments: true
categories: controller
---

Since the v2 router came it became clear that using global singleton
controllers like `App.userController = App.UserController.create()` is
not the way to go. This prevents us from doing a simple binding like

```javascript
App.UserController = Ember.ObjectController.extend({
  accountsBinding: "App.accountsController.content"
})
```

There is no need or even possibility to manage the controller instances
with the new router though. It will create the instance for us. One way
we can use this is with `this.controllerFor`, which can be used inside
of a route.

```javascript
App.UserRoute = Ember.Route.extend({
  setupController: function(controller, model) {
    // some magic with `this.controllerFor("user")`
  }
})
```

but since this method is only available on the route and not inside a
controller, it wasn't very pleasant to specify dependencies (or needs)
between controllers. Which is exactly where needs come in and solve the
issue

```javascript
App.UserController = Ember.ObjectController.extend({
  needs: ["foo"]
});
```

this will give you the opportunity to call `controllers.foo` on the
`App.UserController` instance and get back an instance of
`App.FooController`. You could even (ab)use that in the templates like
this

```html
<!-- inside `users` template -->
{% raw %}{{controllers.foo}}{% endraw %}
```

## Needs vs routing

Needs become incredibly useful when you have nested routes, for example

```javascript
App.Router.map(function() {
  this.resource("post", { path: "/posts/:post_id" }, function() {
    this.route("edit", { path: "/edit" });
  });
});
```

In this case we will get `post`, `post.index` and `post.edit`. If you go
to `/posts/1` you expect to get `post.index` template, which is true,
but the context (or model, or content) is being set on the
`PostController`, not on `PostIndexController`.

When you think about it it does make sense, because the `resource` is
basically shared between `post.index` and `post.edit`, that's why it is
fetched and stored in their parent. Let's go through this in detail:

- visit `/posts/1`
- router basically does `App.Post.find(1)` **and assigns that to the
  content of `PostController`**
- template `post` is rendered
- template `post.index` is rendered in `post`'s outlet

and when you transition to `/posts/1/edit`, the only thing that changes
is the leaf route, you still keep the same `App.Post` model, because it
belongs to the parent `PostRoute`, not to the leaf `PostIndexRoute`. But
this has a drawback. You're not able to directly access the content from
the `post.index` template, since it doesn't belong to it's controller.
That's where needs come in.

```javascript
App.PostIndexController = Ember.ObjectController.extend({
  needs: ["post"]
})
```

and in the `post/index` template, you can access the content like this

```html
{% raw %}{{controllers.post.content}}{% endraw %}
```

By specifying the need Ember will make sure that it gives you the right
`PostController` instance with it's content set to the right value.
