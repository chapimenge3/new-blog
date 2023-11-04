---
title: "Caching in Django part 2"
description: "Does caching really improve the performance of your application? Let's find out."
meta_title: ""
date: 2022-10-28 19:23:39
image: "/images/blogs/python/django_cache.webp"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["career"]
draft: false
---

Does caching really improve the performance of your application? Let's find out

This is the second part of the blog on [caching in Django](https://blog.chapimenge.com/posts/django/cahing/). In this blog, we will be looking at how to use Redis as a caching backend in Django view.

## Project Overview

In this blog we are going to build a simple Django application that will use Redis as a caching backend. We will be using Django 4.0.6 and Python 3.10.4.

So the project is a very simple event management application. We are going to have two models in our application. One is the Event model and the other is the User model.

Without further ado, let's get started.

## Creating Django Project

We are going to create a new Django project using the following command:

```bash
django-admin startproject django_caching
```

This will create a new Django project called django_caching. We will be using this project for the rest of the blog.

## Creating Django App

We are going to create a new Django app called events using the following command:

```bash
python manage.py startapp events
```

This will create a new Django app called events. we are going create a model inside this app.

go to `/events/models.py` and add the following code:

```python
from django.db import models

class Event(models.Model):
    name = models.CharField(max_length=255)
    date = models.DateTimeField()
    location = models.CharField(max_length=255)
    description = models.TextField()

    def __str__(self):
        return self.name

class Guest(models.Model):
    event = models.ForeignKey(Event, on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    email = models.EmailField()

    def __str__(self):
        return self.name
```


For this project we are also going to add django rest framework. To add this we need to install the django rest framework using the following command:

```bash
pip install djangorestframework
```

Now let's add the django apps to our project. Go to `/django_caching/settings.py` and add the following code:

```python
...

INSTALLED_APPS = [
    ...

    'rest_framework',
    'events',

    ...
]

# Reddis Cache config

REDIS_CONFIG = {
    # please use python-dotenv for real world application
    'host': 'REDIS_HOST',
    'port': 'REDIS_PORT',
    'password': 'REDIS_PASSWORD',
    'username': 'REDIS_USERNAME',
}

if all(i for i in REDIS_CONFIG.values()):
    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.redis.RedisCache',
            'LOCATION': f'redis://{REDIS_CONFIG["username"]}:{REDIS_CONFIG["password"]}@{REDIS_CONFIG["host"]}:{REDIS_CONFIG["port"]}/0',
        }
    }


# Rest framework configuration(optional)
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 100
}

...
```

Now let's create the database for our project. To do this we need to run the following command:

```bash
python manage.py make migrations
python manage.py migrate
```

One last thing before moving to the next step is to add serializers for our models. create a file called `/events/serializers.py` and add the following code:

```python
from rest_framework.serializers import ModelSerializer

from event.models import Event, Guest

class EventSerializer(ModelSerializer):
    class Meta:
        model = Event
        fields = ('id', 'name', 'date', 'location', 'description')
        read_only_fields = ('id',)

class GuestSerializer(ModelSerializer):
    event = EventSerializer(read_only=True)
    class Meta:
        model = Guest
        fields = ('id', 'event', 'name', 'email')
        read_only_fields = ('id',)
        extra_kwargs = {
            'event': {'required': True},
        }
```

Now we are done with the database related stuff. Let's move to the next step. 

## Creating Django Views

To save time we are going to use django rest framework to create our views. To do this we need to add the following code to `/events/views.py`:

```python
from django.shortcuts import HttpResponse
from django.http import Http404
from django.core.cache import cache
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from event.models import Event, Guest

from event.serializers import EventSerializer, GuestSerializer

CACHE_TTL = 60 * 15
USE_CACHE = True
INVALIDATE_CACHE = True
USE_IMPROVED_Query = True

class EventList(APIView):
    def get(self, request):

        if USE_CACHE:
            events = cache.get('events')
            if events is not None:
                return Response(events)
            event = Event.objects.all()
            serializer = EventSerializer(event, many=True)
            cache.set('events', serializer.data, CACHE_TTL)
            return Response(serializer.data)

        events = Event.objects.all()
        serializer = EventSerializer(events, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = EventSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            if INVALIDATE_CACHE:
                cache.delete('events')
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

So let's break down the above code.

we first import the necessary modules. Then we define some constants that we will be using in our view. The constants are:
- `CACHE_TTL`: This is the time to live for our cache. In this case we are setting it to 15 minutes.
- `USE_CACHE`: This is a boolean that we will be using to determine if we want to use the cache or not.
- `INVALIDATE_CACHE`: This is a boolean that we will be using to determine if we want to invalidate the cache or not.
- `USE_IMPROVED_Query`: This is a boolean that we will be using to determine if we want to use the improved query or not.

The above constants are just for demonstration purposes. In a real world application you already know which constants you want to use.

```python
class EventList(APIView):
    ...
```
Then we create a class called `EventList` that inherits from `APIView`. This class will be used to list all the events and create a new event. We then define the `get` and `post` methods for this class. The `get` method will be used to list all the events and the `post` method will be used to create a new event.

```python
    ...
    def get(self, request):
        ...
```
The `get` method first checks if we want to use the cache or not. If we want to use the cache then it will check if the cache exists or not. If the cache exists then it will return the cache. If the cache does not exist then it will query the database and return the data. If we do not want to use the cache then it will query the database and return the data.

```python
    ...
    def post(self, request):
        ...
```
The `post` method first checks if the data sent is valid. If the data is valid then it will save the data to the database. If the data is not valid then it will return the errors. After saving the data to the database it will check if we want to invalidate the cache or not. If we want to invalidate the cache then it will delete the cache. If we do not want to invalidate the cache then it will not delete the cache. 

Now Let's create the url for our view. To do this we need to add the following code to `/events/urls.py`:

```python
from django.urls import path

from event import views

urlpatterns = [
    path('events/', views.EventList.as_view(), name='event_list'),
]
```
and add the following code to `/django_caching/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('events.urls')),
]
```

Now all we need to do is to run the server and test our view. To do this we need to run the following command:

```bash
python manage.py runserver
```

Now open your API client and test the view. You should get the following response:

First let's create a new event. To do this we need to send a `POST` request to `http://localhost:8000/events/` with the following data:

```json
{
    "name": "DjangoCon US 2020",
    "date": "2020-07-15",
    "location": "San Diego, CA",
    "description": "DjangoCon US is the annual conference for the Django community. It is a place to meet other users and developers of the framework, catch up on the latest news and developments, and learn about what's coming in the future."
}
```

the response to the above should return status `200` and respond with the object that was created.

Now create multiple time by changing the name of the event. Make sure to create at least 10 or higher is okay.

Now let's list all the events. To do this we need to send a `GET` request to `http://localhost:8000/events/`. The response should return status `200` and respond with all the events that were created.

Now to check the request again. The response should return status `200` and respond will be retrieved from the cache.

To check the object is coming from the cache or not. Remember those constants that we defined earlier. Let's change the `USE_CACHE` constant to `False`. Now send the request again. The response should return status `200` and respond will be retrieved from the database. Record the time it took to retrieve the data from the database. 

Now change the `USE_CACHE` constant back to `True`. Now send the request again. The response should return status `200` and respond will be retrieved from the cache. Record the time it took to retrieve the data from the cache. 

Now compare the time it took to retrieve the data from the database and the cache. You should notice that the time it took to retrieve the data from the cache is much faster than the time it took to retrieve the data from the database.

Or another way to check if the data is coming from the cache or not is to to use `django-debug-toolbar`. To do this we need to add the following code to `/django_caching/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'debug_toolbar',
]

```

and add the following code to `/django_caching/urls.py`:

```python
...

urlpatterns = [
    ...
    path('api-auth/', include('rest_framework.urls')), # add this too to use the browsable API
    path('__debug__/', include(debug_toolbar.urls)),
]
```

and make sure to install `django-debug-toolbar` by running the following command:

```bash
pip install django-debug-toolbar
```

Now open your browser and go to `http://localhost:8000/events/` and you should debug toolbar at the right side of the page. Click on the `Cache` tab and you should see the cache information.

![debug-toolbar](/screenshot-debugtoolbar.webp)

Try making the constant `USE_CACHE` to `False` and send the request again. You should see the cache information is empty.

Now this blog can go in more detail but for today it is enough. Make sure to implement the caching for the GuestList view, EventDetail view, and GuestDetail view.

## Conclusion

Today we learned how to implement caching in Django. We learned how to caching the data in redis server and retrieve it from the cache. We also learned how to use `django-debug-toolbar` to check the cache information.

if you want this project you can find it on [GitHub](https://github.com/chapimenge3/redis-cache). 

There is more example in the above repository. You can check it out.

## References

- [Django Caching](https://docs.djangoproject.com/en/3.0/topics/cache/)
- [Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Redis Lab](https://redislabs.com/)

Thank you for reading.

Chapi Menge