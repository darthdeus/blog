---
layout: post
title: "PostgreSQL Basics by Example"
date: 2013-08-19 01:19
comments: true
categories: postgresql
---

Connecting to a database

```
$ psql postgres     # the default database
$ psql database_name
```

Connecting as a specific user

```
$ psql postgres john
$ psql -U john postgres
```

Connecting to a host/port (by default `psql` uses a unix socket)

```
$ psql -h localhost -p 5432 postgres
```

You can also explicitly specify if you want to enter a password `-W` or not `-w`

```
$ psql -w postgres
$ psql -W postgres
Password:
```

Once you're inside `psql` you can control the database. Here's a couple of handy commands

```
postgres=# \h                 # help on SQL commands
postgres=# \?                 # help on psql commands, such as \? and \h
postgres=# \l                 # list databases
postgres=# \c database_name   # connect to a database
postgres=# \d                 # list of tables
postgres=# \d table_name      # schema of a given table
postgres=# \du                # list roles
postgres=# \e                 # edit in $EDITOR
```

At this point you can just type SQL statements and they'll be executed on the database you're currently
connected to.

## User Management

Once your application goes into production, or basically anywhere outside of your dev machine,
you're going to want to create some users and restrict access.

We have two options for creating users, either from the shell via `createuser` or via SQL `CREATE ROLE`

```
$ createuser john
postgres=# CREATE ROLE john;
```

One thing to note here is that by default users created with `CREATE ROLE` can't log in. To allow login you need to provide
the `LOGIN` attribute

```
postgres=# CREATE ROLE john LOGIN;
postgres=# CREATE ROLE john WITH LOGIN; # the same as above
postgres=# CREATE USER john;            # alternative to CREATE ROLE which adds the LOGIN attribute
```

You can also add the `LOGIN` attribute with `ALTER ROLE`

```
postgres=# ALTER ROLE john LOGIN;
postgres=# ALTER ROLE john NOLOGIN;   # remove login
```

You can also specify multiple attributes when using `CREATE ROLE` or `ALTER ROLE`, but bare in mind that `ALTER ROLE` doesn't change the permissions the role already has which you don't specify.

```
postgres=# CREATE ROLE deploy SUPERUSER LOGIN;
CREATE ROLE
postgres=# ALTER ROLE deploy NOSUPERUSER CREATEDB;  # the LOGIN privilege is not touched here
ALTER ROLE
postgres=# \du deploy
           List of roles
 Role name | Attributes | Member of
-----------+------------+-----------
 deploy    | Create DB  | {}
```

There's an alternative to `CREATE ROLE john WITH LOGIN`, and that's `CREATE USER` which automatically creates the `LOGIN` permission. It is important to understand that users and roles are the same thing. In fact there's no such thing as a user in PostgreSQL, only a role with LOGIN permission

```
postgres=# CREATE USER john;
CREATE ROLE
postgres=# CREATE ROLE kate;
CREATE ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 darth     | Superuser, Create role, Create DB, Replication | {}
 john      |                                                | {}
 kate      | Cannot login                                   | {}
```

You can also create groups via `CREATE GROUP` (which is now aliased to `CREATE ROLE`), and then grant or revoke
access to other roles.

```
postgres=# CREATE GROUP admin LOGIN;
CREATE ROLE
postgres=# GRANT admin TO john;
GRANT ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 admin     |                                                | {}
 darth     | Superuser, Create role, Create DB, Replication | {}
 john      |                                                | {admin}
 kate      | Cannot login                                   | {}
postgres=# REVOKE admin FROM john;
REVOKE ROLE
postgres=# \du
                             List of roles
 Role name |                   Attributes                   | Member of
-----------+------------------------------------------------+-----------
 admin     |                                                | {}
 darth     | Superuser, Create role, Create DB, Replication | {}
 john      |                                                | {}
 kate      | Cannot login                                   | {}
```
