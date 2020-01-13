---
layout: post
title: Fixing iOS development basic errors
date: '2020-01-12T11:40:00.005-07:00'
author: Anu Mittal
categories: 'anu'
subclass: post
tags:
- iOS
- Swift
- Xcode
---

Error to fix: 
xcrun: error: active developer path ("/Users/ ....") does not exist


Given: Your current xcode is not installed in applications folder
When: If you update your Xcode from App Store
Then: Your App Store downloaded Xcode version will be move to application folder
and your Xcode-select path might change and result in the above error.

To fix: 
Update your xcode path by running the following command: 
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
