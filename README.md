[![Build Status](https://travis-ci.org/absent1706/sqlalchemy-mixins.svg?branch=master)](https://travis-ci.org/absent1706/sqlalchemy-mixins)
[![Test Coverage](https://codeclimate.com/github/absent1706/sqlalchemy-mixins/badges/coverage.svg)](https://codeclimate.com/github/absent1706/sqlalchemy-mixins/coverage)
[![Code Health](https://landscape.io/github/absent1706/sqlalchemy-mixins/master/landscape.svg?style=flat)](https://landscape.io/github/absent1706/sqlalchemy-mixins/master)
[![PyPI version](https://img.shields.io/pypi/v/sqlalchemy_mixins.svg)](https://pypi.python.org/pypi/sqlalchemy_mixins)
[![Python versions](https://img.shields.io/pypi/pyversions/sqlalchemy_mixins.svg)](https://travis-ci.org/absent1706/sqlalchemy-mixins)

# SQLAlchemy mixins
A pack of framework-agnostic, easy-to-integrate and well tested mixins for SQLAlchemy ORM.

Heavily inspired by [Django ORM](https://docs.djangoproject.com/en/1.10/topics/db/queries/)
and [Eloquent ORM](https://laravel.com/docs/5.4/eloquent)

Why it's cool:
 * framework-agnostic
 * easy integration:
   ```python
    from sqlalchemy_mixins import ActiveRecordMixin

    class User(Base, ActiveRecordMixin):
         pass
    ```
 * clean code, splitted by modules
 * follows best practices of
    [Django ORM](https://docs.djangoproject.com/en/1.10/topics/db/queries/),
    [Peewee](http://docs.peewee-orm.com/)
    and [Eloquent ORM](https://laravel.com/docs/5.4/eloquent#retrieving-single-models),
 * 95%+ test coverage
 * already powers a big project

## Table of Contents

1. [Installation](#installation)
1. [Quick Start](#quick-start)
1. [Features](#features)
    1. [Active Record](#active-record)
        1. [CRUD](#crud)
        1. [Querying](#querying)
    1. [Eager Load](#eager-load)
    1. [Django-like queries](#django-like-queries)
        1. [Filter and sort by relations](#filter-and-sort-by-relations)
        1. [Automatic eager load relations](#automatic-eager-load-relations)
    1. [All-in-one: smart_query](#all-in-one-smart_query)
    1. [Beauty \_\_repr\_\_](#beauty-__repr__)
1. [Internal architecture notes](#internal-architecture-notes)
1. [Comparison with existing solutions](#comparison-with-existing-solutions)
1. [Changelog](#changelog)

## Installation

Use pip
```
pip install sqlalchemy_mixins
```

Run tests
```
python -m unittest discover sqlalchemy_mixins/
```

## Quick Start

Here's a quick demo of what out mixins can do.

```python
bob = User.create(name='Bob')
post1 = Post.create(body='Post 1', user=bob, rating=3)
post2 = Post.create(body='long-long-long-long-long body', rating=2,
                    user=User.create(name='Bill'),
                    comments=[Comment.create(body='cool!', user=bob)])

# filter using operators like 'in' and 'contains' and relations like 'user'
# will output this beauty: <Post #1 body:'Post1' user:'Bill'>
print(Post.where(rating__in=[2, 3, 4], user___name__like='%Bi%').all())
# joinedload post and user
print(Comment.with_joined('user', 'post', 'post.comments').first())
# subqueryload posts and their comments
print(User.with_subquery('posts', 'posts.comments').first())
# sort by rating DESC, user name ASC
print(Post.sort('-rating', 'user___name').all())
```

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/all_features.py)


# Features

Main features are [Active Record](#active-record), [Eager Load](#eager-load), [Django-like queries](#django-like-queries)
and [Beauty \_\_repr\_\_](#beauty-__repr__).

## Active Record
provided by [`sqlalchemy_mixins.ActiveRecordMixin`](sqlalchemy_mixins/activerecord.py)

SQLAlchemy's [Data Mapper](https://en.wikipedia.org/wiki/Data_mapper_pattern)
pattern is cool, but
[Active Record](https://en.wikipedia.org/wiki/Active_record_pattern)
pattern is easiest and more [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

Well, we implemented it on top of Data Mapper!
All we need is to just inject [session](http://docs.sqlalchemy.org/en/latest/orm/session.html) into ORM class while bootstrapping our app:

```python
BaseModel.set_session(session)
# now we have access to BaseOrmModel.session property
```

### CRUD
We all love SQLAlchemy, but doing [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)
is a bit tricky there.

For example, creating an object needs 3 lines of code:
```python
bob = User(name='Bobby', age=1)
session.add(bob)
session.flush()
```

Well, having access to session from model, we can just write
```python
bob = User.create(name='Bobby', age=1)
```
that's how it's done in [Django ORM](https://docs.djangoproject.com/en/1.10/ref/models/querysets/#create)
and [Peewee](http://docs.peewee-orm.com/en/latest/peewee/querying.html#creating-a-new-record)

update and delete methods are provided as well
```python
bob.update(name='Bob', age=21)
bob.delete()
```

And, as in [Django](https://docs.djangoproject.com/en/1.10/topics/db/queries/#retrieving-a-single-object-with-get)
and [Eloquent](https://laravel.com/docs/5.4/eloquent#retrieving-single-models),
we can quickly retrieve object by id
```python
User.find(1) # instead of session.query(User).get(1)
```

and fail if such id doesn't exist
```python
User.find_or_fail(123987) # will raise sqlalchemy_mixins.ModelNotFoundError
```

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/activerecord.py) and [tests](sqlalchemy_mixins/tests/test_activerecord.py)

### Querying
As in [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/quickstart/#a-minimal-application),
[Peewee](http://docs.peewee-orm.com/en/latest/peewee/api.html#Model.select)
and [Django ORM](https://docs.djangoproject.com/en/1.10/topics/db/queries/#retrieving-objects),
you can quickly query some class
```python
User.query # instead of session.query(User)
```

Also we can quickly retrieve first or all objects:
```python
User.first() # instead of session.query(User).first()
User.all() # instead of session.query(User).all()
```

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/activerecord.py) and [tests](sqlalchemy_mixins/tests/test_activerecord.py)

## Eager load
provided by [`sqlalchemy_mixins.EagerLoadMixin`](sqlalchemy_mixins/eagerload.py)

### Nested eager load
If you use SQLAlchemy's [eager loading](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html#eager-loading),
you may find it not very convenient, especially when we want, say,
load user, all his posts and comments to every his post in the same query.

Well, now you can easily set what ORM relations you want to eager load
```python
User.with_({
    User.posts: {
        Post.comments: {
            Comment.user: None
        }
    }
}.all()
```

or we can write strings instead of class properties:
```python
User.with_({
    'posts': {
        'comments': {
            'user': None
        }
    }
}.all()
```

### Subquery load
Sometimes we want to load relations in separate query, i.e. do [subqueryload](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.subqueryload).
For example, we load posts on page like [this](http://www.qopy.me/3V4Tsu_GTpCMJySzvVH1QQ),
and for each post we want to have user and all comments (to display their count).

To speed up query, we load posts in separate query, but, in this separate query, join user
```python
from sqlalchemy_mixins import SUBQUERYLOAD
Post.with_({
    'comments': (SUBQUERYLOAD, {  # load posts in separate query
        'user': None  # but, in this separate query, join user
    })
}}
```

Here, posts will be loaded on first query, and comments with users - in second one.
See [SQLAlchemy docs](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html)
for explaining relationship loading techniques.

> Default loading method is [joinedload](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.joinedload)
> (`None` in schema)
>
> Explicitly use `SUBQUERY` if you want it.

### Quick eager load
For simple cases, when you want to just 
[joinedload](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.joinedload)
or [subqueryload](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.subqueryload) 
a few relations, we have easier syntax for you:

```python
Comment.with_joined('user', 'post', 'post.comments').first()
User.with_subquery('posts', 'posts.comments').all()
```

> Note that you can split relations with dot like `post.comments`
> due to [this SQLAlchemy feature](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.subqueryload_all)


![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/eagerload.py) and [tests](sqlalchemy_mixins/tests/test_eagerload.py)

## Filter and sort by relations
provided by [`sqlalchemy_mixins.SmartQueryMixin`](sqlalchemy_mixins/smartquery.py)

### Django-like queries
We implement Django-like
[field lookups](https://docs.djangoproject.com/en/1.10/topics/db/queries/#field-lookups)
and
[automatic relation joins](https://docs.djangoproject.com/en/1.10/topics/db/queries/#lookups-that-span-relationships).

It means you can **filter and sort dynamically by attributes defined in strings!**

So, having defined `Post` model with `Post.user` relationship to `User` model,
you can write
```python
Post.where(rating__gt=2, user___name__like='%Bi%').all() # post rating > 2 and post user name like ...
Post.sort('-rating', 'user___name').all() # sort by rating DESC, user name ASC
```
(`___` splits relation and attribute, `__` splits attribute and operator)


> **All relations used in filtering/sorting should be _explicitly set_, not just being a backref**
>
> In our example, `Post.user` relationship should be defined in `Post` class even if `User.posts` is defined too.
>
> So, you can't type
> ```python
> class User(BaseModel):
>     # ...
>     user = sa.orm.relationship('User', backref='posts')
> ```
> and skip defining `Post.user` relationship. You must define it anyway:
>
> ```python
> class Post(BaseModel):
>     # ...
>     user = sa.orm.relationship('User') # define it anyway
> ```

For DRY-ifying your code and incapsulating business logic, you can use
SQLAlchemy's [hybrid attributes](http://docs.sqlalchemy.org/en/latest/orm/extensions/hybrid.html)
and [hybrid_methods](http://docs.sqlalchemy.org/en/latest/orm/extensions/hybrid.html?highlight=hybrid_method#sqlalchemy.ext.hybrid.hybrid_method).
Using them in our filtering/sorting is straightforward (see examples and tests).

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/smartquery.py) and [tests](sqlalchemy_mixins/tests/test_smartquery.py)

### Automatic eager load relations
Well, as [`sqlalchemy_mixins.SmartQueryMixin`](sqlalchemy_mixins/smartquery.py) does auto-joins for filtering/sorting,
there's a sense to tell sqlalchemy that we already joined that relation.

So that relations are automatically set to be joinedload if they were used for filtering/sorting.


So, if we write
```python
comments = Comment.where(post___public=True, post___user___name__like='Bi%').all()
```
then no additional query will be executed if we will access used relations
```python
comments[0].post
comments[0].post.user
```

Cool, isn't it? =)

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/smartquery.py) and [tests](sqlalchemy_mixins/tests/test_smartquery.py)

### All-in-one: smart_query
#### Filter, sort and eager load in one smartest method.
provided by [`sqlalchemy_mixins.SmartQueryMixin`](sqlalchemy_mixins/smartquery.py)

In real world, we want to filter, sort and also eager load some relations at once.
Well, if we use the same, say, `User.posts` relation in filtering and sorting,
it **should not be joined twice**.

That's why we combined filter, sort and eager load in one smartest method:
```python
Comment.smart_query(
    filters={
        'post___public': True,
        'user__isnull': False
    },
    sort_attrs=['user___name', '-created_at'],
    schema={
        'post': {
            'user': None
        }
    }).all()
```

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/smartquery.py) and [tests](sqlalchemy_mixins/tests/test_smartquery.py)

## Beauty \_\_repr\_\_
provided by [`sqlalchemy_mixins.ReprMixin`](sqalchemy_mixins/repr.py)

As developers, we need to debug things with convenient.
When we play in REPL, we can see this

```
>>> session.query(Post).all()
[<myapp.models.Post object at 0x04287A50>, <myapp.models.Post object at 0x04287A90>]
```

Well, using our mixin, we can have more readable output with post IDs:

```
>>> session.query(Post).all()
[<Post #11>, <Post #12>]
```

Even more, in `Post` model, we can define what else (except id) we want to see:

```python
class Post(BaseModel):
    __repr_attrs__ = ['user', 'body'] # body is just column, user is relationship
    # ...

```

Now we have
```
>>> session.query(Post).all()
[<Post #11 user:<User #1 'Bill'> body:'post 11'>,
 <Post #12 user:<User #2 'Bob'> body:'post 12'>]

```

![icon](http://i.piccy.info/i9/c7168c8821f9e7023e32fd784d0e2f54/1489489664/1113/1127895/rsz_18_256.png)
See [full example](examples/repr.py) and [tests](sqlalchemy_mixins/tests/test_repr.py)

# Internal architecture notes
Some mixins re-use the same functionality. It lives in [`sqlalchemy_mixins.SessionMixin`](sqlalchemy_mixins/session.py) (session access) and [`sqlalchemy_mixins.InspectionMixin`](sqlalchemy_mixins/inspection.py) (inspecting columns, relations etc.) and other mixins inherit them.

You can use these mixins standalone if you want.

Here's a UML diagram of mixin hierarchy:
![Mixin hierarchy](http://i.piccy.info/i9/5104592ba4b0b998e6ecf951b4c5b67f/1489403573/34714/1127895/diagram.png)

# Comparison with existing solutions
There're a lot of extensions for SQLAlchemy, but most of them are not so universal.

## Active record
We found several implementations of this pattern.

**[ActiveAlchemy](https://github.com/mardix/active-alchemy/)**

Cool, but it forces you to use [their own way](https://github.com/mardix/active-alchemy/#create-the-model) to instantiate SQLAlchemy
while to use [`sqlalchemy_mixins.ActiveRecordMixin`](sqlalchemy_mixins/activerecord.py) you should just make you model to inherit it.

**[Flask-ActiveRecord](https://github.com/kofrasa/flask-activerecord)**

Cool, but tightly coupled with Flask.

**[sqlalchemy-activerecord](https://github.com/deespater/sqlalchemy-activerecord)**

Framework-agnostic, but lacks of functionality (only `save` method is provided) and Readme.

## Django-like queries
There exists [sqlalchemy-django-query](https://github.com/mitsuhiko/sqlalchemy-django-query)
package which does similar things and it's really awesome.

But:
 * it doesn't [automatic eager load relations](#automatic-eager-load-relations)
 * it doesn't work with [hybrid attributes](http://docs.sqlalchemy.org/en/latest/orm/extensions/hybrid.html)
   and [hybrid_methods](http://docs.sqlalchemy.org/en/latest/orm/extensions/hybrid.html?highlight=hybrid_method#sqlalchemy.ext.hybrid.hybrid_method)

### Beauty \_\_repr\_\_
[sqlalchemy-repr](https://github.com/manicmaniac/sqlalchemy-repr) already does this,
but there you can't choose which columns to output. It simply prints all columns, which can lead to too big output.

# Changelog

## v0.2

More clear methods in [`sqlalchemy_mixins.EagerLoadMixin`](sqlalchemy_mixins/eagerload.py):

 * *added* `with_subquery` method: it's like `with_joined`, but for [subqueryload](http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html#sqlalchemy.orm.subqueryload).
   So you can now write:
   
   ```python
   User.with_subquery('posts', 'comments').all()
   ```  
 * `with_joined` method *arguments change*: now you should simply write
  
    ```python
    Comment.with_joined('user','post')
    ```
    
    instead of
    
    ```python
    Comment.with_joined(['user','post'])
    ```
 * `with_` method *arguments change*: it now accepts *only dict schemas*. If you want to quickly joinedload relations, use `with_joined`   
 * `with_dict` method *removed*. Instead, use `with_` method   

Other changes in [`sqlalchemy_mixins.EagerLoadMixin`](sqlalchemy_mixins/eagerload.py):

 * constants *rename*: use nicer `JOINED` and `SUBQUERY` instead of `JOINEDLOAD` and `SUBQUERYLOAD`
