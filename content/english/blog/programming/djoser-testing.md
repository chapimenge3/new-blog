---
title: "How to Test Django, Django Rest-Framework, and Djoser with Email Verification"
description: "Does testing really improve the quality of your application or is it just a waste of time? Let's find out."
meta_title: ""
date: 2022-10-28 19:23:39
image: "/images/blogs/python/django_testing_cover.png"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["career"]
draft: false
---

Does testing really improve the quality of your application or is it just a waste of time? Let's find out


Happy new year(2023) everyone. I hope you all had a great holiday. I am back with a new blog post.

In this blog post we are gonna see, how to test djoser while you are using email activation for your users.

**Requirements**

- you are already using djoser inside your application.

So you are using django, django-rest-framework and djoser with email activation but having hard time to test the authentication ? well let's fix that problem in a bit.

First of all we need to do is separate our settings file into production, development and testing. The structure looks like this

```
settings/
      __init__.py
      base.py
      production.py
      development.py
      testing.py
```

The first thing we should do is create a folder inside your main project called `settings`. Careful with the name of the folder it should be exactly the same as `settings.py` but without the `.py`.

Next we create the below files inside the `settings` folder

```
__init__.py
base.py
production.py
development.py
testing.py
```

these files are just used as the name suggested in development the django settings use the `development.py` while testing it uses `testing.py` and so on.

Now let's copy the common settings for all of the project into the `base.py`. This is up to you to identify what is common for all of the project.

In my case, for `base.py` i put the following

```python
BASE_DIR
SECRET_KEY # read from environment variable
INSTALLED_APPS
MIDDLEWARE
ROOT_URLCONF
TEMPLATES
WSGI_APPLICATION
AUTH_PASSWORD_VALIDATORS
LANGUAGE_CODE
TIME_ZONE
USE_I18N
USE_TZ
STATIC_URL
DEFAULT_AUTO_FIELD
AUTH_USER_MODEL
# emial related parameter are read from env in my case
EMAIL_BACKEND
EMAIL_HOST
EMAIL_PORT
EMAIL_HOST_USER
EMAIL_HOST_PASSWORD
EMAIL_USE_TLS
REST_FRAMEWORK # depends on your choice of drf settings
DJOSER # common setting only we will extend it in other files
```

for `production.py`

```python
from .base import *

DEBUG = False
ALLOWED_HOSTS = ['yoursite.com'] # up to you choice what to allow
DATABASES # i use different db for prod
DJOSER.update({
....
}) # i use different conf for prod 
```

for `development.py`

```python
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['*'] # up to you choice what to allow
DATABASES # i use different db for dev
DJOSER.update({
....
}) # i use different conf for dev
```

for `testing.py` will see this file in detail but for now

```python
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['*'] # up to you choice what to allow
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
DATABASES # i use different db for test
DJOSER.update({
....
}) # i use different conf for test
```

Now the last file is the `__init__.py`
this file load dynamically the settings

```python
import os

from . import base

enviroment = os.environ.get('ENVIROMENT', 'development')

if enviroment == 'production':
    from .production import *
elif enviroment == 'testing':
    from .testing import *
else:
    from .development import *
```

Djoser uses different serializer class for different purpose but we only focus on [Email serialzers](https://djoser.readthedocs.io/en/latest/settings.html#email) check out the docs

So now let's write the `testing.py` file. In this file we are gonna use sqlite3 for the database and we are gonna use the console backend for the email.

```python
from .base import *
DEBUG = True
ALLOWED_HOSTS = ['*']
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
DATABASES = {
    # 'default': {
    #     'ENGINE': 'django.db.backends.sqlite3',
    #     'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    # }
    # for postgresql: i use postgres 
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djoser',
        'USER': 'djoser',
        'PASSWORD': 'djoser',
        'HOST': 'localhost',
        'PORT': '5432',
    }

    # for mysql:
    # 'default': {
    #     'ENGINE': 'django.db.backends.mysql',
    #     'NAME': 'djoser',
    #     'USER': 'djoser',
    #     'PASSWORD': 'djoser',
    #     'HOST': 'localhost',
    #     'PORT': '3306',
    # }
}

DJOSER_EMAIL = {
    'activation': 'your-authentication-app.email.ActivationEmail'
}

if 'EMAIL' in DJOSER:
    DJOSER['EMAIL'].update(DJOSER_EMAIL)
else:
    DJOSER['EMAIL'] = DJOSER_EMAIL

```

if you notice in the above code we have the following line

```python
DJOSER_EMAIL = {
    'activation': 'your-authentication-app.email.ActivationEmail'
}
```

we override the email activation serializer with `your-authentication-app.email.ActivationEmail` but it doesn't exist yet so let's create that now.

in your authentication app create a file called `email.py` and put the below serializer and let's talk about it.

```python
from django.contrib.auth.tokens import default_token_generator

# djoser imports
from templated_mail.mail import BaseEmailMessage
from djoser import utils
from djoser.conf import settings

EMAILS = {}

class ActivationEmail(BaseEmailMessage):
    """Email Activation Token Generator
    """
    template_name = "email/activation.html"

    def get_context_data(self):
        # ActivationEmail can be deleted
        context = super().get_context_data()
        user = context.get("user")
        context["uid"] = utils.encode_uid(user.pk)
        context["token"] = default_token_generator.make_token(user)
        context["url"] = settings.ACTIVATION_URL.format(**context)
        uid, token = context['uid'], context['token']
        # here we store all the requested tokens in a dictionary for later use
        EMAILS[user.email] = {'uid': uid, 'token': token}
        return context
```

The only thing we add from the djoser serializer is the `EMAILS` dictionary which is used to store the email sent from our server.

```python
EMAIL = {}

...

EMAILS[user.email] = {'uid': uid, 'token': token}
```

These are used to store email sent from our server.

NOTE: we are using `EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'` because while testing we don't want django to send actual emails.

Now we are going to test our users. Here are some cases to test but in the mean time it is up yourself what to test

```python
from django.urls import reverse
from django.contrib.auth import get_user_model
from rest_framework import status
from rest_framework.test import APITestCase

User = get_user_model()

class UserViewSetTest(APITestCase):

    def setUp(self):
        """
        Set up method which is used to initialize before any test run.
        """
        self.user_info = self.generate_user_info()
    
    def generate_user_info(self):
        """Generate user data for new user.
        Returns:
            Dict: dictionary of the test user data.
        """
        return {
            "first_name": "fake.first_name()",
            "last_name": "fake.last_name()",
            "username": "fake.user_name()",
            "password": "fake.password()",
        }
    
    def test_create_user(self):
        """
        Test for creating users using API.
        """
        url = reverse("user-list")
        response = self.client.post(
            url,
            self.user_info,
        )
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        user = User.objects.get(id=response.data['id'])
        self.assertEqual(user.email, self.user_info["email"])
        self.assertEqual(user.username, self.user_info["username"])
        # self.assertEqual(user.ssn, self.user_info["ssn"])
        self.assertTrue(user.password is not self.user_info["password"])
        self.assertTrue(user.is_deleted is not True)
        self.assertTrue(user.father_first_name is None)
        self.assertTrue(user.mother_first_name is None)
        self.assertTrue(user.password is not None)
        self.assertTrue(user.birth_date is not None)
    
    def test_get_token(self):
        """
        This test is used to test the login API. getting token and testing the token.
        """
        # Create a new user to login
        user_info = self.generate_user_info()
        new_user = self.client.post(
            reverse("user-list"),
            user_info,
        )
        self.assertEqual(new_user.status_code, status.HTTP_201_CREATED)

        # Activation of User
        from authentications.email import EMAILS

        activation_url = reverse('user-activation')
        activation_data = EMAILS[user_info["email"]]
        self.client.post(activation_url, activation_data)

        url = reverse("jwt-create")
        data = {
            "username": user_info["username"],
            "password": user_info["password"],
        }
        response = self.client.post(url, data)

        self.assertTrue(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data["access"] is not None)
    
    def test_get_user(self):
        """
        This test for retrieving single user using API.
        """

        # Create a new user to login
        new_user = self.client.post(
            reverse("user-list"),
            self.user_info,
        )
        self.assertEqual(new_user.status_code, status.HTTP_201_CREATED)

        # Activate User
        from authentications.email import EMAILS

        activation_url = "http://127.0.0.1:8000/auth/users/activation/"
        activation_data = EMAILS[self.user_info["email"]]
        self.client.post(activation_url, activation_data)

        # Get token
        url = reverse("jwt-create")
        data = {
            "username": self.user_info["username"],
            "password": self.user_info["password"],
        }

        response = self.client.post(url, data)
        self.assertTrue(response.status_code, status.HTTP_200_OK)

        # Get user
        token = response.data["access"]
        self.client.credentials(HTTP_AUTHORIZATION=f"JWT {token}")

        url = reverse('user-list', kwargs={'id':new_user.data["id"]})
        get_user = self.client.get(url)

        self.assertEqual(get_user.status_code, status.HTTP_200_OK)
        self.assertEqual(get_user.data["id"], new_user.data["id"])
        self.assertEqual(get_user.data["email"], new_user.data["email"])

        test_user = self.client.post(
            reverse("user-list"),
            self.generate_user_info(),
        )
        url = url = reverse('user-list', kwargs={'id': test_user.data['id'] })
        get_user = self.client.get(url)
        self.assertEqual(get_user.status_code, status.HTTP_404_NOT_FOUND)
```

Finally run the test. Before that i wanna take the chance to introduce you to the `Makefile` which can be used to automate the process of running test and other commands. I am not gonna explain it here but it is worth to check it out by yourself.

For now just run the test by first activating the virtual environment, exporting test environment variables and then run the test.

```bash
source venv/bin/activate
export environment=test
python manage.py test
```

There is different kind of test in the above take what you prefer to test but the main purpose of this blog is to let you know how to test djoser with email activation on.

So i hope you have learned something new today. If you have any question or suggestion please let me know in the comment section.

Hope it helps you.

Enjoy.
