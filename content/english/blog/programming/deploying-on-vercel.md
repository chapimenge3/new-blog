---
title: "Deploy telegram bot on Vercel(Python)"
description: "Looking for a way to deploy your telegram bot on Vercel? Here's how to do it."
meta_title: ""
date: 2023-04-10 00:23:39
image: "/images/blogs/programming/vercel-deployment.png"
categories: ["Programming"]
author: "Chapi Menge"
tags: ["programming", "python", "telegram", "vercel"]
draft: false
---

Looking for a way to deploy your telegram bot on Vercel? Here's how to do it.

Hey Everyone, It has been a while since I have written a blog post. I have been busy moving to a new country and getting settled. I am still not settled but i was thinking i should drop this blog post.

Today Topics are:

- [Telegram Bot Long Polling and Webhook](#telegram-bot-long-polling-and-webhook)
- [Using FastAPI to create a Telegram Bot](#using-fastapi-to-create-a-telegram-bot)
- [Deploying FastAPI on Vercel](#deploying-fastapi-on-vercel)
- [Deploying Telegram Bot using FastAPI on Vercel](#deploying-fastapi-on-vercel)

Let's get started.

## Telegram Bot Long Polling and Webhook

Telegram Bot is a great way to interact with your users. You can create a bot and interact with your users. You can create a bot using [BotFather](https://t.me/botfather).

So what is Long Polling and Webhook?

Long Polling is a way to get updates from Telegram. When something happens on Telegram, Telegram stores the updates in their servers. When you send a request to Telegram, Telegram sends you the updates that happened since the last time you sent a request. This is called Getting Updates.

You can try this out by using the [getUpdates](https://core.telegram.org/bots/api#getupdates) method. You can use this method to get updates from Telegram.

```bash
curl -X GET "https://api.telegram.org/bot<token>/getUpdates"
```

So, Long pooling is a way to get the updates from telegram repeatedly. The Sample Python Code for Long Polling is:

```python
import requests

updates = []
update_id = 0

while True:
    response = requests.get(f"https://api.telegram.org/bot<token>/getUpdates", params={"offset": update_id})
    updates.extend(response.json()["result"])
    update_id = updates[-1]["update_id"] + 1
```

The above code will get the updates from Telegram and store it in the updates variable. The update_id is used to get the updates from the last update_id.

Webhook is a way to get updates from Telegram. When you create a webhook, Telegram sends you the updates when something happens on Telegram. You can create a webhook using the [setWebhook](https://core.telegram.org/bots/api#setwebhook) method.

```bash
curl -X POST "https://api.telegram.org/bot<token>/setWebhook" -d "url=https://example.com"
```

For this case you don't need to send a request to Telegram to get the updates. Telegram will send you the updates when something happens on Telegram.

This is actually a very efficient and simple way to get updates from Telegram. You can use this method to create a Telegram Bot.

We are going to use this method to create a Telegram Bot using FastAPI.

## Using FastAPI to create a Telegram Bot

FastAPI is a great framework to create APIs. It is very fast and easy to use. You can create a Telegram Bot using FastAPI.

My always approach is to create a Telegram Bot using Long Polling and then create a Telegram Bot using Webhook when I want to deploy it on Vercel.

Let's create a Telegram Bot using Long Polling. For this example we are going to use the [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) library.

```python
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters
import os

TOKEN = os.environ.get("TOKEN")

def start(update, context):
    context.bot.send_message(chat_id=update.effective_chat.id, text="I'm a bot, please talk to me!")

def register_handlers(dispatcher):
    start_handler = CommandHandler('start', start)
    dispatcher.add_handler(start_handler)

def main():
    updater = Updater(token=TOKEN, use_context=True)
    dispatcher = updater.dispatcher

    register_handlers(dispatcher)

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
```

If you run the above code and start the bot, you will see the following message when you send a message to the bot.

`I'm a bot, please talk to me!`

So the next thing is we want to deploy this on vercel. For this first we need to know if we can deploy a very simple hello world API on Vercel. So let's create a very simple FastAPI app.

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def index():
    return {"message": "Hello World"}
```

If you run the above code and go to `http://localhost:8000`, you will see the following message.

`{"message": "Hello World"}`

**NOW THE MOST IMPORTANT PART. We need to create a `vercel.json` file in the root directory of the project. This file is used to configure the deployment on Vercel.**

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/api/index" }]
}
```

You can read more about the `vercel.json` file [here](https://vercel.com/docs/configuration).

NOW PUT THE ABOVE CODE IN THE BELOW DIRECTORY STRUCTURE.

```
├── api
│   └── index.py
├── vercel.json
└── requirements.txt
```

Put all the requirements in the `requirements.txt` file.

```bash
fastapi==0.63.0
python-telegram-bot==13.4
```

anything else you need to add to the `requirements.txt` file.

Now you might ask if you have any file in the root directory of the project or anywhere it doesn't affect the deployment on Vercel. you can have something like below.

```
├── api
│   └── index.py
├── vercel.json
├── requirements.txt
├── README.md
└── LICENSE
└── test.py
```

So don't worry about the files in the root directory of the project except the `vercel.json`, `requirements.txt` and `api` directory.

- Now push the code to GitHub.
- Login to Vercel.
- Click on `New Project`.
- Select `Import Project`.
- Select the GitHub repository.
- Click on `Import and Deploy`.
- Wait for the deployment to finish.
- Click on `Visit` to see the deployed app.

After you deploy the app, you will see the following message.

`{"message": "Hello World"}`

If you don't see the above message, you might have done something wrong. Follow the steps again.

Now let's create a Telegram Bot using Webhook. For this we need to create a new file in the `api` directory.

```python
import os
from typing import Optional

from fastapi import FastAPI, Request
from pydantic import BaseModel

from telegram import Update, Bot
from telegram.ext import Dispatcher, MessageHandler, Filters, CommandHandler

TOKEN = os.environ.get("TOKEN")

app = FastAPI()

class TelegramWebhook(BaseModel):
    '''
    Telegram Webhook Model using Pydantic for request body validation
    '''
    update_id: int
    message: Optional[dict]
    edited_message: Optional[dict]
    channel_post: Optional[dict]
    edited_channel_post: Optional[dict]
    inline_query: Optional[dict]
    chosen_inline_result: Optional[dict]
    callback_query: Optional[dict]
    shipping_query: Optional[dict]
    pre_checkout_query: Optional[dict]
    poll: Optional[dict]
    poll_answer: Optional[dict]

def start(update, context):
    context.bot.send_message(chat_id=update.effective_chat.id, text="I'm a bot, please talk to me!")

def register_handlers(dispatcher):
    start_handler = CommandHandler('start', start)
    dispatcher.add_handler(start_handler)

@app.post("/webhook")
def webhook(webhook_data: TelegramWebhook):
    '''
    Telegram Webhook
    '''
    # Method 1
    bot = Bot(token=TOKEN)
    update = Update.de_json(webhook_data.__dict__, bot) # convert the Telegram Webhook class to dictionary using __dict__ dunder method
    dispatcher = Dispatcher(bot, None, workers=4)
    register_handlers(dispatcher)

    # handle webhook request
    dispatcher.process_update(update)

    # Method 2
    # you can just handle the webhook request here without using python-telegram-bot
    # if webhook_data.message:
    #     if webhook_data.message.text == '/start':
    #         send_message(webhook_data.message.chat.id, 'Hello World')

    return {"message": "ok"}

@app.get("/")
def index():
    return {"message": "Hello World"}
```

Make sure to add the index route because later we can test if the app is working or not using that.

## Deploying FastAPI on Vercel

Now commit the code and push it to GitHub. You can go to vercel and see the deployment is happening. After the deployment is finished, you can go to the `Settings` tab and click on `Environment Variables`. Add the `TOKEN` variable and add the token of your bot.

We are almost done hang in there.

Now we need to add the webhook to the bot. Copy the URL of the deployed app and add `/webhook` to the end of it and set it as the webhook of the bot. Below is the example.

Let's say https://telegram-bot.vercel.app is the url of the deployed app. Then the webhook url will be https://telegram-bot.vercel.app/webhook.

```bash
curl 'https://api.telegram.org/bot<token>/setWebhook?url=https://telegram-bot.vercel.app/webhook'
```

The above command will return the following message.

`{"ok":true,"result":true,"description":"Webhook was set"}`

Now go to the Telegram app and send a message to the bot. You will see the following message.

`I'm a bot, please talk to me!`

That's it. You have successfully deployed a Telegram Bot on Vercel.

Very simple right? If you have any questions, feel free to ask in the comments.

If you want to see the code that i deployed to vercel you can check out the [Chapi's GitHub](https://github.com/chapimenge3)

As always, Enjoy Coding!

Chapi Menge
