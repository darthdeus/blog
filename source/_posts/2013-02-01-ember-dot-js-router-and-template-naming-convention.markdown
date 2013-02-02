---
layout: post
title: "Ember.js Router and Template Naming Convention"
date: 2013-02-01 19:43
comments: true
categories: Router
---

Ever since the change to `resource` and `route` a lot of people are
confused about the meaning of the two and how they affect naming. Here's
the difference:

- `resource` - a thing
- `route` - something to do with the thing

Let's say we have a model `App.Post` and we want to show a list of posts
and a new post form. There are many ways you can go about this, so let's
start with the simplest.

```javascript
App.Router.map(function() {
  this.resource("posts", { path: "/" });
  this.route("new", { path: "/new" });
});
```

This would result in the following template structure

```html
<script type="text/x-handlebars" data-template-name="posts">
  ... list the posts
</script>

<script type="text/x-handlebars" data-template-name="new">
  ... new post template
</script>
```

With the following naming

```javascript
PostsRoute
PostsController
PostsView
NewRoute
NewController
NewView
```

[Here's a JSBin](http://jsbin.com/ogorab/33/edit)

This is almost never useful, since you might have many `/new` actions
and you'd need to scope them to the resource, which would be done as
follows

```javascript
App.Router.map(function() {
  this.resource("posts", { path: "/" }, function() {
    this.route("new", { path: "/new" });
  });
});
```

Here things get a little more complicated, since we're nesting something
inside the resource. This means that we'll end up with three templates
instead of two

```html
<script type="text/x-handlebars" data-template-name="posts">
  <h1>This is the outlet</h1>

  {{outlet}}
</script>

<script type="text/x-handlebars" data-template-name="posts/index">
  ... list the posts
</script>

<script type="text/x-handlebars" data-template-name="posts/new">
  ... new post template
</script>
```

With the following naming

```javascript
PostsRoute
PostsController
PostsView

PostsIndexRoute
PostsIndexController
PostsIndexView

PostsNewRoute
PostsNewController
PostsNewView
```

[Here's a JSBin](http://jsbin.com/ogorab/34/edit)

This means whenever you create a resource it will create a brand new
namespace. That namespace will have an `{{outlet}}` which is named after the
resource and all of the child routes will be inserted into it.

There are many reasons behind it, but let's try another example which
will make it more obvious. We will add a `/:post_id` and
`/:post_id/edit` routes.

```javascript
App.Router.map(function() {
  this.resource("posts", { path: "/" }, function() {
    this.resource("post", { path: "/:post_id" }, function() {
      this.route("edit", { path: "/edit" });
    });

    this.route("new", { path: "/new" });
  });
});
```

Additional to the routes in the previous example, this will give us

```javascript
// IMPORTANT - it's not PostsPostRoute, because `resource`
// always creates a new namespace
PostRoute 
PostController
PostView

PostIndexRoute
PostIndexController
PostIndexView

PostEditRoute
PostEditController
PostEditView
```

Templates are named accordingly `post`, `post.index` and `post.edit`,
**there is nothing like `posts.post.index` or `posts.post` or
`posts.post.edit`**.

[Here's a JSBin](http://jsbin.com/ogorab/35/edit)

But the problem is when we try to access the `App.Post` model from the
`post/index` or `post/edit` template. It is only available in the `post`
template with the outlet. Now why is that?

Since we are defining a `resource` it is expected that the child routes
will be related to that `resource`, that's why they don't need to load
it separately. They can access it from the parent `PostController` via
`needs` ([more about that can be found in this article](http://darthdeus.github.com/blog/2013/01/27/controllers-needs-explained/)

[Here's a JSBin](http://jsbin.com/ogorab/37/edit)

This is the general pattern you would be using if you want to nest
everything. But what if you don't want to render `post` into the
`posts` outlet? Well nothing prevents you from defining the routes as
this.

```javascript
App.Router.map(function() {
  this.resource("posts", { path: "/" }, function() {
    this.route("new", { path: "/new" });
  });

  this.resource("post", { path: "/:post_id" }, function() {
    this.route("edit", { path: "/edit" });
  });
});
```

What is the difference? The naming remains exactly the same as in the
previous example, even templates are named the same. But the `post`
template will be inserted into the `application` layout, not inside the
`posts` layout. This is the case when you want the detail `post` page to
replace the whole layout, instead of just showing it together with the
`posts` list.

I hope the examples will help you understanding how the v2 routes work,
since this is a completely essential part of Ember.js.

If you have any questions, leave them in the comments or tweet me
[@darthdeus](http://twitter.com/darthdeus).
