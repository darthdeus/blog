---
layout: post
title: "Using Transactions in Ember Data - part 1"
date: 2013-02-02 14:15
comments: true
categories: Data Transactions
---

We talked about transactions in [one of the previous articles](http://darthdeus.github.com/blog/2013/01/27/ember-data-in-depth/)
(read it if you haven't already), but we didn't really touch on when to
use them in real world. One of the most common use cases for me is when
I just want to manage a single record while there are many changes
happening on the page.

Adding a record to a transaction is simple

```javascript
// say that we are in a controller
store = this.get("store");

// this ALWAYS returns a new transaction
transaction = store.transaction();

user = App.User.find(1);
transaction.add(user);

transaction.toString(); // => "<DS.Transaction:ember955>"
```

Now this is obvious, but what if we need to commit the transaction in a
completely different action? Do we need to store the instance somewhere
to use it later?

The answer is NO, we can always return the transaction in which the
record is by calling `.get("transaction")`. We can even do it if we
decide to fetch the user again in a completely different part of the
application.

```javascript
user = App.User.find(1);
user.get("transaction").toString(); // => "<DS.Transaction:ember955>"
```

It doesn't matter in which part of the application you add the record to
a transaction because you can always retrieve the correct instance
later.

Which allows us to do something like this:

```javascript
App.UsersNewRoute = Ember.Route.extend({
  model: function() {
    var transaction = this.get("store").transaction();

    var user = transaction.createRecord(App.User, {});
    return user;
  },

  events: {
    createUser: function(user) {
      user.get("transaction").commit();
    }
  }
});
```

Personally I use this when I only care about one record, but I know that
there might be other which are `dirty` and I don't want to commit those.
This happens almost every time you have two forms displayed at once.
