---
title: "Telegram bot integrate with Chapa Payment API(Python)"
description: "If you are looking for a way to integrate your telegram bot with Chapa Payment API, you are in the right place."
meta_title: ""
date: 2022-11-19 10:23:39
image: "/images/blogs/programming/Chapa-and-Telegram.png"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["programming", "python", "telegram"]
draft: false
---

If you are looking for a way to integrate your telegram bot with Chapa Payment API, you are in the right place.

Hey guys, It's me again. In this post, I will be talking about how to integrate telegram bot with Chapa Payment API. In this blog we are going to build Shopping Channel and integrate with Bot to make payment. So, let's get started.

Today topics are:

- [Introduction](#introduction)
- [Creating a Telegram Channel](#creating-a-telegram-channel)
- [Creating a Telegram Bot](#creating-a-telegram-bot)
- [Creating a Marketplace](#creating-a-marketplace)
- [Integrating with Chapa Payment API](#integrating-with-chapa-payment-api)
- [Conclusion](#conclusion)

## Introduction

If you are reading this i hope you already know what is Chapa and Telegram Bot. If you don't know what is Chapa and Telegram Bot, you can read my

    - [Chapa Payment API](https://chapa.co/)
    - [Telegram Bot](https://core.telegram.org/bots)

So the goal of this blog is to integrate Chapa Payment API with Telegram Bot. We are going to build a Shopping Channel. The flow will be like below

    - Item will be posted in the channel
    - Someone will click buy button
    - They will be redirected to our bot
    - They bot will create checkout in Chapa
    - They bot will give the user a link to pay
    - After successful payment from Chapa the user will be redirected to our bot

## Creating a Telegram Channel

To create a telegram channel you go to your telegram account and click the plus button in your phone

    - Give the channel name
    - Choose if you want to public or private(It's up to you but for now i will make it public)
    - Click **skip** when it let's you to add subscriber

Now your **Channel** is created. One more thing we need to do is get the **channel_id**. To get that

1. Post something in the channel(anything text preferable)
2. Forward that message to @username_to_id_bot
3. Grab the id and put it somewhere we will use it later (make sure the id is negative)

## Creating a Telegram Bot

To create a bot we have to use other bot made by telegram which is a father to all bot (don't ask me the mother i don't know that either 😃)

Step to create telegram bot

1. Goto @BotFather
2. click start and send /newbot
3. Fill out the info and finish creating the bot.
4. Grab the bot token and store it somewhere safe(don't export this by any means)

## Creating a Marketplace

So now the idea is the admin will post the message in the channel and the bot will edit it and adds the buy button.

Now Let's write some code. I suggest you to use Github and Git to manage the code for future.

We are going to use python telegram bot library.

First create python environment variable (optional)

```bash
python -m venv env # or use virtualenv like below if u have it
# virtualenv env
source env/bin/activate # activate the environment

pip install python-telegram-bot chapa python-dotenv
```

Now Create `.env` file and put the token that you copied from bot father earlier.

```bash
TOKEN="YOUR BOT TOKEN FROM BOT FATHER"
```

Also create file called `main.py` and write the below code

```python
import os
import logging # use logging rather than print.

from telegram.ext import Updater, CommandHandler, MessageHandler, Filters
from telegram import InlineKeyboardButton, InlineKeyboardMarkup

# to export the environment variable
from dotenv import load_dotenv
load_dotenv()

# you can obviously edit the logging parameter as you want
# i personally like this pattern. If you don't know what i am saying just ignore it or read more about logging.
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO
)

TOKEN = os.getenv("TOKEN")


def add_buy_button(update, context):
    '''
    This will add a button to any post in the channel

    Args:
        update: telegram.Update object
        context: telegram.Context object

    Return:
        None
    '''
    bot = context.bot.get_me()
    post = update.channel_post
    url = 'https://t.me/{bot.username}/?start={post.message_id}'

    keyboard = [[
        InlineKeyboardButton('Buy', url=url)
    ]]
    reply_markup = InlineKeyboardMarkup(keyboard)

    context.bot.edit_message_reply_markup(
        chat_id=post.chat.id,
        message_id=post.message_id,
        reply_markup=reply_markup
    )

    return


def main():
    '''
    This is main method to control the bot events and add the handlers
    '''
    updater = Updater(TOKEN)
    dispatcher = updater.dispatcher

    dispatcher.add_handler(MessageHandler(
        Filters.chat_type.channel, add_buy_button)
    )

    updater.start_polling()

    updater.idle()

if __name__ == '__main__':
    main()
```

Now let's explain what is going on.

- on the first couple of line above the `add_buy_button` method is setting up the logging and importing modules to use.
- `add_buy_button`:
  - it will get the bot details
  - get the post details
  - build url to redirect the user to the bot
  - make a telegram button
  - edit the post to add the button
- `main`:

  - initializer updater and dispatcher
  - register our method to dispatcher
  - start the bot

- run the `main` function.

Now run the bot and post anything to the channel it will gonna edit it and add one button.

Now we have achieved to make the button let's move to the bot and build it.

## Integrating with Chapa Payment API

Log in to login into your dashboard and generate API Key and put the secret key into `.env` file

```bash
TOKEN="..."
CHAPA_SECRET_KEY="...."
```

We are going to use `chapa` python SDK which is made by myself but if you don't wanna use that you can do it. If you wanna know more about the SDK click [here](https://github.com/chapimenge3/chapa).

Let's add new method to out `main.py`

```python
# update the telegram import
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, WebAppInfo

# add this imports
import random
import string
from chapa import Chapa

CHAPA_SECRET_KEY = os.getenv('CHAPA_SECRET_KEY')
chapa = Chapa(CHAPA_SECRET_KEY, response_format='obj')

CHANNEL_ID = "ID we copied earlier (negative value)"

# new method
def create_chapa_checkout(data):
    '''
    Create chapa checkout url and return the result.

    Args:
        data(dict): data to send to chapa api to generate checkout

    Return:
        data(dict): api response
    '''
    response = chapa.initialize(**data)
    if response.status != 'success':
        return None

    return response.data

def start(update, context):
    '''
    Handle any /start message to bot.

    Args:
        update: telegram.Update object
        context: telegram.Context object

    Return:
        None
    '''
    if update.message.text != '/start':
        cmd = context.args[0] # to identify if the user comes from the channel or not.
    else:
        cmd = None

    bot = context.bot
    bot_username = bot.get_me().username
    user = update.effective_user

    # if the user is coming from the channel not from other place
    if cmd and not cmd.startswith('tx_'):
        try:
            # generate transaction number, random email and random domain. hope you will use the correct value here
            random_tx = "".join(random.choices(string.ascii_letters, k=5))
            random_email = "".join(random.choices(string.ascii_letters, k=5))
            random_domain = "".join(random.choices(string.ascii_letters, k=5))

            data = {
                'email': f"{random_email}@{random_domain}.com",
                'amount': 1000,
                'tx_ref': random_tx,
                'first_name': user.first_name,
                'last_name': user.last_name or ' None',
                'callback_url': f'https://t.me/{bot_username}?start=tx_{random_tx}',
                'customization': {
                    'title': 'Amazing Bot',
                    'description': 'This is a test project for chapa python sdk'
                }
            }

            checkout = create_chapa_checkout(data)
            bot.copy_message(
                chat_id=update.effective_user.id,
                from_chat_id=CHANNEL_ID,
                message_id=cmd
            )

            text = 'You want to buy this item? \n Click the button below to continue'

            keyboard = [[
                InlineKeyboardButton(
                    "Chapa Pay", web_app=WebAppInfo(checkout.checkout_url))
            ]]

            reply_markup = InlineKeyboardMarkup(keyboard)

            bot.send_message(
                chat_id=update.effective_user.id,
                text=text,
                reply_markup=reply_markup
            )

            context.user_data[random_tx] = 'PENDING'

            return

        except Exception as e:
            logger.error('Error in Generating Checkout', e)

    elif cmd and cmd.startswith('tx_'):
        if context.user_data.get(cmd[3:]) == 'PENDING':
            response = chapa.verify(cmd[3:])
            if response.status == 'success':
                context.user_data[cmd[3:]] = 'SUCCESS'
                bot.send_message(
                    chat_id=update.effective_user.id,
                    text='Your payment was successful'
                )
                return

            else:
                context.user_data[cmd[3:]] = 'FAILED'
                bot.send_message(
                    chat_id=update.effective_user.id,
                    text='Your payment failed'
                )
                return

    update.message.reply_text('Well come to Chapa Shopping! \n\nuse this channel @chapashoping or https://t.me/chapashoping')

```

Let's Explain what we did there:

- We imported some module to help us with Telegram New Feature `WebApp` and also some helper modules
- We initialize chapa object which is advised to initialize it globally
- we created `create_chapa_checkout` to create chapa checkout page for our item
- we created `start` method

  - First portion is for the link which the user will if they click the buy button in the channel. Remember we added url to the button when we create it is for redirecting the user with additional parameter added to the `start` query parameter

    - It then generate some random strings like email, id and domain name.
    - build up the data parameter needed by Chapa API instruction. check that out [here](https://developer.chapa.co/docs/accept-payments/)
    - initialize the checkout by sending request to Chapa and get the checkout url.
    - Send to the user to finish payment if they wanna buy by clicking the button

  - Second portion is for the user which will come after they finish payment.

    - It first verify if they have already finished the checkout process by checking the status of their payment.
    - If not then, verify the payment using Chapa API.
    - If the payment is Successful it will send success message or failed message

That's all we need now Let's test it.

For better experience please test it on mobile version.

So let's restart our bot

Go to the telegram channel

Post some Item (**for now we set 1000 ETB for all item for demo purpose**)

Then click buy button and it will take you to the bot and click start again.

The bot will open Chapa Checkout page and after you finish that please close the opened page and click start button once again.

If everything okay the bot will send you success message.

## Conclusion

This blog is to show you how you can set up telegram with chapa api. All the code can be found in in my github page or click [here](https://github.com/chapimenge3/Chapa-Python-SDK-Examples).

https://github.com/chapimenge3/Chapa-Python-SDK-Examples

There are many flaws in the above code I know but my intention is to give you a better idea how you can achieve a simplest integration to Chapa API.

There are some best practices i wanna mention

1. Make sure to keep safe your telegram bot api key and Chapa secret key. Always use environment variable.

2. If you are making Webhook always verify the request is coming from Chapa by using Chapa webhook secret key

3. Make sure to double verify the transaction in your code with your database and in the chapa api too.

if you have any best practice mention it in the comment section.

Until next time, wish you the best.

Chapi Menge.
