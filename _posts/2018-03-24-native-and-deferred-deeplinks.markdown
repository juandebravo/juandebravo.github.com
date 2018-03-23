---
layout: post
title: "Communicating with native applications (1/2)"
categories: [ios, android, adjust, appsflyer]
---

Over the last years, the two predominant mobile OS have implemented several
mechanisms for enabling users and application developers different ways of creating
links that will open a specific screen in a native application or that will show
a specific content inside the native app, giving the option to send
parameters as part of that link.

=> Why this is relevant?

This blog post goes over the main functionalities provided by iOS and Android
for direct communication with an application.

The second part of this post series will tackle a related functionality,
that is how the attribution platforms have built the functionality to deliver content to
an application, no matter if it's installed or not in a mobile device.
While this might look like magic, understanding the internals of how iOS, android and the attribution platform works make the end scenario quite simple.

## The old era. Custom URL schemes

URL scheme lets you communicate with an application through  a protocol
that you define.

Both iOS and android introduced custom URL scheme some versions ago.

=> When iOS
=> When Android

Having said that, it's very unlikely that you will need to implement custom
URLs at this moment, due to the fact that iOS versions older than 9 has a XXX
market share, and Android version lower than 6 (API level 23) represents YYY
of the Android market.

## The new era. Universal (iOS) and App (Android) links

