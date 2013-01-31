---
layout: post
title: "How to find a model by any attribute in Ember.js"
date: 2013-01-31 23:13
comments: true
categories: Data
---

One of the common things people ask about Ember Data is how to find a
single record by it's attribute. This is because the current revision
(11) only offers three methods of fetching records

```javascript
App.User.find(1) // returns a single user record
App.User.find({ username: "wycats" }) // returns a ManyArray
App.User.findQuery({ username: "wycats" }) // same as the above
```

If you want to search for a user by his username, you have two options

## Using .find with _smart_ server side

The way `App.User.find(1)` works is that it does a request to
`/users/1`, which is expected to return just one record.

You could modify your server to accept both `username` and `id` on the
`/users/1` path, which would allow to do `App.User.find("wycats")`. 

There's an issue with this though. If you load the same user via his
`username` and `id`, you'll end up with two records stored in the Ember
identity map.

Which basically means that if you try to retrieve all of the user
records, you will end up with that one user twice.

If you want to read more about this, [checkout this GitHub
issue](https://github.com/emberjs/data/issues/571)

## Using a findQuery

This might not seem like the right solution at first, since it returns a
`DS.ManyArray` instead of just one record, but hang on.

`DS.ManyArray` is a subclass of `DS.RecordArray`, which includes a
`DS.LoadPromise`.

To understand how `DS.LoadPromise` works, we need to understand what
promises are. [There's a great article about
that](https://gist.github.com/3889970), so I won't go into much detail. 

Promise is basically an async monad (I guess that doesn't help, let's
try again).

Promise is something which allows you to return an object which wraps
around a value, even if you don't have the value yet. For example if
you're doing `App.User.findQuery`, you'll get back an empty
`DS.ManyArray` instantly.

It doesn't wait until the AJAX request is finished, it just returns the
empty array, which is populated with the data once the request finishes.

This works because Ember uses data bindings and will automagically
update all of the views once the data is loaded. And also because the
router will wait if it's model has a state `isLoading`. That way you
won't display a page which is half loaded.

## Implementation

Now that we know we're getting a `DS.ManyArray`, we need to figure out a
way to make it represent only the value of it's first element, because
that's what we care about.

```javascript
var users = App.User.findQuery({ username: username });

users.one("didLoad", function() {
  users.resolve(users.get("firstObject"));
});

return users;
```

You can see that we are returning the result of the `findQuery`
instantly, but we're also setting an asynchronous callback which
**resolves the promise** to the `firstObject` once it is loaded.

Another way you could read the `resolve(x)` is _from now you're
representing value `x`_. Using this technique will work in all Ember,
because the data bindings will take care of everything. Always remember
that you don't need to worry about re-rendering your views, just change
the data and Ember will take care of the rest.
