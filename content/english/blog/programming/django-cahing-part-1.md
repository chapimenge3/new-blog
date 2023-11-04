---
title: "Caching in Django part 1"
description: "Does caching really improve the performance of your application? Let's find out."
meta_title: ""
date: 2022-10-28 19:23:39
image: "/images/blogs/python/django_cache.webp"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["python", "django"]
draft: false
---

Does caching really improve the performance of your application? Let's find out

This blog is high level overview of caching in Django. I will be using Django 3.0.5 and Python 3.8.2.

Most website today are dynamic and they need to fetch data from database. This is a very expensive operation and it can be slow. Caching is a way to store data in memory so that it can be retrieved faster. Caching is a very important part of web development and it can be used to improve performance of your website. In this blog, I will be using Django as an example to explain how caching works.

## What is Caching?

Cache is a temporary storage of data. It is used to store data so that future requests for that data can be served faster. It is a technique to improve the performance of the application.

For example, if you have a website that has a lot of users, you can cache the data that is frequently used by the users. This will reduce the load on the database and improve the performance of the application.

In simple pseudo code, caching can be implemented as follows:

```python
def get_data_from_cache(key):
    if key in cache:
        return cache[key]
    else:
        data = fetch_data_from_database(key)
        cache[key] = data
        return data
```

## How to use Caching in Django?

Django provides a caching framework that can be used to cache the data. It is a very simple and easy to use framework. There are multiple caching backends that can be used with Django. The most common caching backend is the in-memory cache and redis cache. In this blog, we are going to see both of them.

## Types of Caching

1. Memory Caching(Memcached)

2. Redis(New in Django 4.0)

3. Database Caching (Not going to cover this)

4. File Caching (Not going to cover this)

## Memory Caching or Memcached

Memcached is a entirely memory based cache server. It is a very fast and efficient cache server. Memcached is a very popular caching backend and used by many websites like Facebook, Wikipedia, etc. It runs as a daemon process and it is very easy to setup. It is a very simple and easy to use caching backends.

### Installing Memcached

Memcached can be installed using the following command:

```bash
sudo apt update
sudo apt-get install memcached
# additional tool which can use to examine, test, and manage your Memcached server
sudo apt install libmemcached-tools
sudo systemctl start memcached # start memcached service
```

### Set up Django

After installing Memcached you’ll need to install a Memcached binding. Django support

- [pylibmc](https://pypi.org/project/pylibmc/)
- [pymemcache](https://pypi.org/project/pymemcache/)

To use Memcached with Django you need to set Cache backend. Here is how to do it

**settings.py**

```python
...

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
        # LOCATION can also be unix:path where path is the path to a Memcached Unix socket file
        # 'LOCATION': 'unix:/tmp/memcached.sock',
    }
}

...

```

Additionally Memchached can also have multiple cache server. This means you can run Memcached daemons on multiple machines, and the program will treat the group of machines as a single cache. To implement this on your Django project, you should include all server addresses in the `LOCATION`, either by **comma(,)** or **semicolon(;)** delimited string, or provide the values as a python list.

So let's say we have 3 Memchached instances running on

- 145.216.61.211:11211
- 246.195.4.49:11211
- 66.33.28.146:11211

we can use these servers like below

```python

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': [
            '145.216.61.211:11211',
            '246.195.4.49:11211',
            '66.33.28.146:11211',
        ]
    }
}

```

### Disadvantage of Memchached

Memchached is a memory based caching the the cached data will be lost when the server is restarted. This is a major disadvantage of Memchached. Of course any of caching backend should not be used as a permanent storage of data. But still it is a major disadvantage.

## Redis

Redis is in-memory database that can be used as a caching backend. To use Redis as a caching backend, you need to install Redis and set it up. Redis is one of the most popular caching backend and it is used by many websites like StackOverflow, GitHub, etc. Redis is a very fast and efficient caching backend. It is a very simple and easy to use caching backend.

### Installing Redis

Redis can be installed using the following command:

we can install redis using apt package manager as follows

```bash
sudo apt update
sudo apt install redis-server
sudo systemctl start redis # start redis service
```

we can also install redis using docker(recommended) as follows

```bash
# assuming you have docker installed
docker run --name redis -p 6379:6379 -d redis
```

The above command will start a redis container and it will be accessible on port 6379.

### Set up Django

After installing Redis you’ll need to install a Redis binding. Django support

- [redis-py](https://pypi.org/project/redis/)

Installing the additional [hiredis-py](https://pypi.org/project/hiredis/) package is also recommended

**settings.py**

```python
...
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379'
    }
}
...
```

Most of the time Redis will be protected by a password. To use Redis with password, you need to set up like below

```python
...

# RECOMMENDED: read password from environment variable

REDIS_HOST = 'localhost' # or your redis host
REDIS_PORT = 6379 # or your redis port
REDIS_USERNAME = 'redis_username' # or your redis username
REDIS_PASSWORD = 'your_redis_password'

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': f'redis://{REDIS_USERNAME}:{REDIS_PASSWORD}@{REDIS_HOST}:{REDIS_PORT}',
    }
}

...
```

As your redis cache grows in size or you want to use multiple redis instances, you can use Redis either as a **comma(,)** or **semicolon(;)** delimited string, or as a list. While using multiple redis instances, django will use the first redis instance for writing and the rest for reading.

```python
...

# RECOMMENDED: read password from environment variable
REDIS_HOSTS = [
    ('246.24.85.60', 6379),
    ('109.103.190.255', 6380),
    ('113.89.138.152', 6381),
]

# result of list(map(lambda x: f'redis://{x[0]}:{x[1]}', REDIS_HOSTS))
# ['redis://246.24.85.60:6379', 'redis://109.103.190.255:6380', 'redis://109.103.190.255:6381']

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': list(map(lambda x: f'redis://{x[0]}:{x[1]}', REDIS_HOSTS)),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

...
```

Resource to use free redis server: [redislabs](https://redislabs.com/).

In the next blog we will see how to use caching inside django view.

This is it for this article. I hope you enjoyed it. If you have any questions or suggestions, please feel free to comment below.

Thank you for reading.

Chapi Menge
