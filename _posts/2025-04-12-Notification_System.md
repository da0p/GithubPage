---
title: "Notification System"
date: 2025-04-12
---

A notification system delivers messages to subscribers. Messages can be mobile
push notification, SMS message, and Email.

## Different Types of Notifications

### iOS Push Notification

There are three components needed to send an iOS push notification:

- **Provider**: a provider builds and sends notification requests to Apple Push
Notification Service (APNS). In order to construct a push notification, the
provider provides the following data:
  - _Device token_: This is a unique identifier used for sending push notifications.
  - _Payload_: This is a JSON dictionary that contains a notification's payload
  - _APNS_: This is a remote service provided by Apple to propagate push notifications to iOS devices.

![iOS Push Notification](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/iOS_push_notification.drawio.png)

### Android Push Notifications

Instead of using APNs, Firebase Cloud Messaging (FCM) is commonly used to send
push notifications to android devices.

![Android Push Notification](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/android_push_notification.drawio.png)

### SMS Message

For SMS messages, third party SMS services like Twilio, Nexmo, and others are used

![SMS Push Notification](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/sms_push_notification.drawio.png)

### Email

Commercial email services can be used to offer a better delivery rate and data analytics

![Email Push Notification](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/email_push_notification.drawio.png)

## System Design

![Notification System](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/notification_system.drawio.png)