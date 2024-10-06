---
title: "Python Firebase Notifications"
date: 2024-10-06
draft: false
tags: ["python", "firebase", "notifications"]
categories: ["Tutorial"]
---

## Introduction

Firebase Cloud Messaging (FCM) is a powerful tool for sending notifications across platforms like Android, iOS, and the Web. In this tutorial, I will guide you through sending Firebase notifications using Python.

## What is Firebase Cloud Messaging (FCM)?

Firebase Cloud Messaging allows you to send notifications and data messages to your users’ devices. This is commonly used in mobile apps to re-engage users and provide real-time updates.

## Setting Up Firebase for Python Notifications

To send notifications via Firebase, we need to set up Firebase in your project:

1. Create a Firebase project in the Firebase Console.
2. Download the `serviceAccountKey.json` file for authentication.
3. Set up Firebase Cloud Messaging for the platforms you are targeting.

## Installing the Required Python Libraries

For this project, you’ll need the `firebase-admin` Python library:
To install the `firebase-admin` library, run the command:
```bash
pip install firebase-admin
```

## Initialize the Firebase Admin SDK

```python
import firebase_admin
from firebase_admin import credentials, messaging

# Initialize Firebase with the service account key
cred = credentials.Certificate("path/to/serviceAccountKey.json")
firebase_admin.initialize_app(cred)
```

## Sending a notification to a device token

```python
import firebase_admin
from firebase_admin import credentials, messaging

def send_data_notification_to_device(token, title, body, data):
    # Create the message with additional data
    message = messaging.Message(
        notification=messaging.Notification(
            title=title,
            body=body,
        ),
        data=data,  # Dictionary of additional data
        token=token,
    )

    # Send the message
    try:
        response = messaging.send(message)
        print('Successfully sent message:', response)
    except Exception as e:
        print('Error sending message:', e)

# Usage
device_token = "your-device-token-here"
extra_data = {"key1": "value1", "key2": "value2"}
send_data_notification_to_device(device_token, "Test Title", "Test Body", extra_data)
```

## Sending a notification to a topic
```python
def send_data_notification_to_topic(topic, title, body, data):
    # Create the message with additional data
    message = messaging.Message(
        notification=messaging.Notification(
            title=title,
            body=body,
        ),
        data=data,  # Dictionary of additional data
        topic=topic,
    )

    # Send the message
    try:
        response = messaging.send(message)
        print('Successfully sent message:', response)
    except Exception as e:
        print('Error sending message:', e)

# Usage
topic_name = "news"
extra_data = {"key1": "value1", "key2": "value2"}
send_data_notification_to_topic(topic_name, "Test Title", "Test Body", extra_data)
```
