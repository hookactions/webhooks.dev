---
title: "Basics"
date: 2019-07-02T21:01:55+02:00
draft: false
weight: 1
---


Very simply, a webhook is a way for a foreign service to interact with your website. A foreign service may be a third party like Salesforce or Stripe. It may also be a service you control; for example you may have a service that emits events with a webhook to your main application.


## What makes a webhook different than an API call?
Let's say you have a web application built in two parts. The first part is the backend that is hosted at **api.example.com** and you have the frontend that is hosted at **example.com**. When the frontend wants to retreive data from the backend API, it can make an HTTP call to your API server. When you make this API call, you control the data you are sending to the API. As an example, let's say you have a `/create-user` endpoint that creates a user in your application.

```
POST /create-user

{
  "first_name": "John",
  "last_name": "Doe",
  "email_address": "john@example.com",
  "password1": "123",
  "password2": "123
}
```

Which returns a response like so:

```json
{
  "id": "123456"
}
```

In the `/create-user` HTTP call, you have full control over what the data looks like and how it is sent to your server. With a webhook you don't have control over this, and this is where the complexity of managing them start to become apparent.

## Webhook example

Let's take a simple example: everytime a user is updated in YetiCMS, we want to update them in our application. Luckily for us, YetiCMS provides webhooks so that we can get notified when a user is updated. In their website we set up a webhook to point at `/webhooks/update-user`. If we were building this, we would have the context of our business and any other technology context, such as using JSON or XML. In our application, we may already let users update themselves, so we have a `/api/update-user` endpoint that takes in the following data:

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "birth_date": "1986-07-02"
}
```

Most webhooks use a `POST` request to your server and if you're lucky, they use the same data format as you (JSON or XML). Generally, you won't be able to use your existing `/api/update-user` endpoint for a few reasons:

1. Authentication – When your application handles authentication, it might be done with a JWT HTTP header, session cookie, or even basic auth. 99% of the time, the service sending you a webhook will not use the same authentication as you do. Most of the time webhooks use a signing method called an HMAC signature to verify the webhook is coming from where you think it is.

2. Formatting – Most modern APIs are built with JSON, but if you're receiving a webhook from an older service it may send it in XML. It's up to you to parse it properly.

3. Structure – If you are able to receive JSON to your JSON api, then most likely the structure of the JSON will be different than you're expecting. In our above example, YetiCMS might send data in the structure like so:

```json
{
  "user": {
    "id": "224cbd28-9d05-11e9-b358-3c15c2eb7774",
    "name": {
      "first": "John",
      "infix": null,
      "last": "Doe",
      "suffix": "III"
    },
    "birthDate": "07/02/1986"
  }
}
```

Now your job is to translate that data into your own internal representation of a user and update accordingly. Sometimes it is not as trivial as taking camelCase keys and converting them to snake_case.

## Conclusion

This is what makes webhooks time consuming. Translating differently structured data than you are expecting takes time. If you have to integrate more than one service with webhooks, you will be extremely lucky if they work the same way.
