---
title: "Crafting a Telegram-Based Shoe Shop with Chapa & Django"
description: "Explore 'Sole Steps,' where we fuse Django's power with Chapa Payments in Telegram to revolutionize online shoe shopping—one click at a time"
meta_title: ""
date: 2024-03-22 00:00:00
image: "/images/blogs/python/django-chapa-telegram.png"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["django"]
draft: true
---

Explore 'Sole Steps,' where we fuse Django's power with Chapa Payments in Telegram to revolutionize online shoe shopping—one click at a time

{{< toc >}}

Wooop!  Been a while since i write blog here. Like every dev execuse, I have been swamped with work. But today, I am back with a bang! We are going to build a Telegram-based shoe shop using Django and Chapa Payments.

This Blog is not your tipical Django related blog, We are going to really implement Fully Online Shoe Shop with Telegram Bot. So, let's get started.

## What is Chapa?

Chapa is the leading online payment gateway that enables businesses in Ethiopia to accept digital payments from anyone and anywhere at any time. For more information, visit [Chapa](https://chapa.co/).

## What is Telegram?

Telegram is a cloud-based instant messaging and voice over IP service. Telegram client apps are available for Android, iOS, Windows Phone, Windows, macOS, and Linux. Users can send messages and exchange photos, videos, stickers, audio, and files of any type. For more information, visit [Telegram](https://telegram.org/).

## What is Django?

Django is a high-level Python web framework that enables rapid development of secure and maintainable websites. Built by experienced developers, Django takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. For more information, visit [Django](https://www.djangoproject.com/).


Now that we got the basics out of the way, let's dive into the project.

## Project overview

We are going to build a Telegram Bot that will Post Shoes to our Telegram Channel, Allow User to Buy Shoes and Pay using Chapa Payment Gateway. The Bot will also Add Orders to our Django Admin Panel. It will also notify the user whenever there is an update on their order.

## Project Setup

You can find the complete code for this project on [GitHub](https://github.com/chapimenge3/chapa)

### Create a Django Project

First, we need to create a Django project. If you don't have Django installed, you can install it using pip:

```bash
pip install django
pip install httpx # We will use httpx to make http request to Telegram
```

Am not gonna go throgh the setup for python environment, I am assuming you know how to setup python environment.

Now, let's create a Django project:

```bash
django-admin startproject solesteps
```

Now the plan is we will gonna have 2 Django Apps, One for the Bot and One for the Shoe Shop.

### Create a Django App for the Bot

```bash
python manage.py startapp bot
python manage.py startapp shop
```

Let's register the apps in the `INSTALLED_APPS` list in the `settings.py` file:

```python
...
INSTALLED_APPS = [
    ...
    'bot',
    'shop',
]
...
```

So Before we move on further, Let's See what our Database Models will look like in the below image.

{{< image src="images/blogs/python/django-chapa-telegram-db.png" caption="" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Upwork Profile"  webp="false" >}}

Now we will define our models in the `shop/models.py` and `bot/models.py` files.

```python

# bot/models.py <-- Add the below code to this file

class User(models.Model):
    ...

class Channel(models.Model):
    ...

# shop/models.py  <-- Add the below code to this file

from django.db import models
from bot.models import User

class Product(models.Model):
    ...

class ShippingAddresses(models.Model):
    ...

class ShippingOptions(models.Model):
    ...

class Order(models.Model):
    ...

```

{{< accordion "Click to see the Full Code" >}}

{{< button label="Get the full Code" link="https://gist.github.com/chapimenge3/af426615d9396c8d9c7f64ee059967ce#file-bot-models-py" style="solid" >}}

{{< /accordion >}}


Before Migration we need to register our new User model to be used as the default user model. So let's add the below code to the `settings.py` file.

```python
...

AUTH_USER_MODEL = 'bot.User'
...
```

Now we will create the database tables by running the following command:

```bash
python manage.py makemigrations
python manage.py migrate
```

{{< accordion "Why should you need to do this?" >}}

The reason we need to run the `makemigrations` is to prepare the changes to be applied to the database. This command will have a lot of use to track the changes for database just like git will track the changes for your code. The `migrate` command will apply the changes to the database.

{{< /accordion >}}

The above command will create the database tables for the models we defined. Since we are not planning to build our own Admin Panel, we will use Django's built-in Admin Panel. So let's register our models in the `admin.py` file.

```python

# bot/admin.py <-- Add the below code to this file
from bot.models import User, Channel
from django.contrib import admin

admin.site.register(User)
admin.site.register(Channel)

# shop/admin.py  <-- Add the below code to this file

from shop.models import Product, ShippingAddresses, ShippingOptions, Order
from django.contrib import admin

admin.site.register(Product)
admin.site.register(ShippingAddresses)
admin.site.register(ShippingOptions)
admin.site.register(Order)

```

{{< notice "tip" >}}
For Advanced devs, you can customize the admin pannel with custom admin libraries like `django-admin-interface` or `django-jet` or customize the admin panel with `ModelAdmin` class.
{{< /notice >}}

Okay, Now let's run the server and see if the admin panel is working.

```bash
python manage.py runserver
```

Now open your browser and go to `http://localhost:8000/admin/`. Now if everything goes well, you should see the Django Admin Panel. It asks you to login right, so let's create a superuser.

```bash
python manage.py createsuperuser
# Fill the details as required
```

Login with the superuser credentials and you should see the admin panel. Try to add some products and see if it's working.

Moving 