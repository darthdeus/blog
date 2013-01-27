---
layout: post
title: "Ember Data in Depth"
date: 2013-01-27 13:52
comments: true
categories: "Ember Data"
---

# Inside Ember Data

This is a guide explaining how Ember Data works internaly. My initial
motivation for writing this is to understand Ember better myself. I've
found that every time I understand something about how Ember works, it
improves my application code.

## Main parts

First we need to understand what are the main concepts. Let's start with
a simple example.


```javascript
App.User = DS.Model.extend({
  username: DS.attr("string")
});
```

Let's dive deep into this. There are four important concepts, two of
which are basic Ember.js and we're going to skip them

- `App.User` represents a `User` class in the `App` namespace
- `username` represents a property on the `User` class

These are the basics and you should be familiar with them to understand
the rest of this guide. Next we have `DS.Model` and `DS.attr`:

## DS.Model and DS.attr

`DS.Model` is one of the core concepts in Ember Data and it represents a
single _resource_. Models can have relationships with other models,
similar to how you'd model your data in a relational database. But let's
ignore that for now.

`DS.Model` is both a state machine and a promise. If you don't
understand what promises are, please take a look at [this awesome
article](https://gist.github.com/3889970) which explains them in depth.

State machines are used throughout Ember and they basically represent something _which can have multiple states and can transition between the states_. For example `DS.Model` can have the following states (*[taken from the official Ember guide](http://emberjs.com/guides/models/model-lifecycle/)*):

- `isLoaded` - The adapter has finished retrieving the current state of the record from its backend.
- `isDirty` - The record has local changes that have not yet been saved by the adapter. This includes records that have been created (but not yet saved) or deleted.
- `isSaving` - The record has been sent to the adapter to have its changes saved to the backend, but the adapter has not yet confirmed that the changes were successful.
- `isDeleted` - The record was marked for deletion. When `isDeleted` is true and `isDirty` is `true`, the record is deleted locally but the deletion was not yet persisted. When `isSaving` is true, the change is in-flight. When both `isDirty` and `isSaving` are `false`, the change has been saved.
- `isError` - The adapter reported that it was unable to save local changes to the backend. This may also result in the record having its `isValid` property become false if the adapter reported that server-side validations failed.
- `isNew` - The record was created locally and the adapter did not yet report that it was successfully saved.
`isValid` No client-side validations have failed and the adapter did not report any server-side validation failures.

We can also bind to these with event handlers, which will be explained later, but for now let's just list them:

- `didLoad`
- `didCreate`
- `didUpdate`
- `didDelete`
- `becameError`
- `becameInvalid`

_[I would also encourage you to go take a look at the source documentation on GitHub](https://github.com/emberjs/data/blob/f274153754cb8b629cd98fc6c590f18bc8ee3ff6/packages/ember-data/lib/system/model/states.js#L223-L245)_

It is important for us to understand what each state means, because they
can affect how our application behaves. For example if we try to modify
a record which is already being saved, we will get an exception saying
something like this

```
Attempted to handle event `willSetProperty` on <App.User:ember1144:null>
while in state rootState.loaded.created.inFlight. Called with
{reference: [object Object], store: <App.Store:ember313>, name: username}
```

The important part here is the `rootState.loaded.created.inFlight`. If
we look at [the source of `DirtyState`](https://github.com/emberjs/data/blob/f274153754cb8b629cd98fc6c590f18bc8ee3ff6/packages/ember-data/lib/system/model/states.js#L254-L261), we can see what this means

> Dirty states have three child states:
>
> - `uncommitted`: the store has not yet handed off the record to be saved.
> - `inFlight`: the store has handed off the record to be saved, but the adapter has not yet acknowledged success.
> - `invalid`: the record has invalid information and cannot be send to the adapter yet.

Let's go through the record lifecycle and observe it's state. We can do
this by doing `.get("stateManager.currentState.name")`

```javascript
user = App.User.find(1)
user.get("isLoaded") // => true
user.get("isDirty") // => false
user.get("stateManager.currentState.name") // => loaded

user.set("username", "wycats")
user.get("isLoaded") // => true
user.get("isDirty") // => true, which means comitting the transaction will save the record
user.get("stateManager.currentState.name") // => uncommitted

user.get("transaction").commit()
// while the record is being saved
user.get("stateManager.currentState.name") // => inFlight
user.get("isSaving") // => true
// after the record was saved
user.get("stateManager.currentState.name") // => saved
```

## Transactions and `commit()`

In the previous example, we've used `get("transaction").commit()` to
persist the changes to the server. `.commit()` will take all `dirty`
records in the transaction and persiste them to the server.

A record becomes dirty whenever one of it's attributes change. For
example

```javascript
user = App.User.find(1)
user.get("isDirty") // => false
user.set("username", "wycats")
user.get("isDirty") // => true
```

If we create a new record, it will be dirty by default

```javascript
user = App.User.createRecord()
user.get("isDirty") // => true
```

[Currently there's a regression](https://github.com/emberjs/data/pull/646)
that we change an attribute to something else, and then back to the
original value, the record will be marked as dirty.

```javascript
user = App.User.find(1)
originalUsername = user.get("username")

user.get("isDirty") // => false
user.set("username", "wycats")
user.get("isDirty") // => true
user.set("username", originalUsername)
user.get("isDirty") // => true, even though it should be false
```

But let's hope this will be fixed soon.

### Transactions

Until now we assumed that there is some *global* transaction which is
the same for every single model. But this doesn't have to be true. We
can create our own transactions and manage them at our will.

I recommend you take a look at [the tests for transactions in Ember Data
repository](https://github.com/emberjs/data/blob/master/packages/ember-data/tests/integration/transactions/basic_test.js).
They basically show all of the scenarios which you can encounter. For
example

```javascript
transaction = store.transaction();
record = transaction.createRecord(App.User, {});

transaction.commit(); // this will save the record to the server

record.set("foo", "bar");
transaction.commit(); // nothing is committed here, because the record
                      // is removed from the transaction when it is saved

store.commit(); // this will save the record properly
```

We can also add a record to a transaction, which will remove it from the
global transaction. Important thing to note here is that
[`store.transaction()`](https://github.com/emberjs/data/blob/master/packages/ember-data/lib/system/store.js#L127-L129)
**always returns a new transaction**.

```javascript
user = App.User.find(1);
transaction = store.transaction();
transaction.add(user);

user.set("username", "wycats");

store.commit(); // nothing happens
transaction.commit(); // user is saved
```

Same goes for deleting records

```javascript
user = App.User.find(1);
transaction = store.transaction();
transaction.add(user);

user.deleteRecord();

store.commit(); // nothing happens
transaction.commit(); // user is deleted
```

We can also remove a record from a transaction

```javascript
user = App.User.find(1);
transaction = store.transaction();

transaction.add(user);
transaction.remove(user);

user.set("name", "wycats");

transaction.commit(); // nothing happens
```

One scenario when transactions can be useful is when you just need to
change one record, without affecting changes to other records. You can
put that change in a separate transaction, instead of just doing
`store.commit()`.
