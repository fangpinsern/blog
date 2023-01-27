---
title: "What Are the Factors of Authentication"
date: 2022-01-14T10:59:44+08:00
showToc: true
TocOpen: false

ShowReadingTime: true
ShowWordCount: false
---

## Introduction

We always hear people or websites saying 2-factor authentication, but what factors are considered in 2-factor authentication? And what makes 2-factor authentication secure?

## Brief History

In the past, most people accomplish authentication using usernames and passwords. This works well in the past. However, with computers getting more and more powerful, it is increasingly insufficient to use usernames and passwords as a form of authentication over the internet. Powerful computers are able to crack most human-readable passwords in minutes.

Hence a more secure way of authentication is required. This introduces the concept of Multi-Factor Authentication (MFA). Instead of just relying on passwords, we have other factor to check the identity of the person over the internet.

## Factors of Authentication

There are actually 3 different factors of authentication. (Some would argue that there are 5)

### 1. Something you know

This is something you and only you know. The most common form of this is username and passwords. This is known (by some) to be the weakest form of authentication as once someone else knows what you know (username and password in this case) they are able to impersonate you on the web. However, compared to others, this is likely the most usable factor compared to others

### 2. Something you have

This is in the form of something you own. The most common form of this is an access card, keys, or in today's world your phone. This factor is less usable as you need to have the physical item to be authenticated. Imagine going to the office without your access card. What a pain.

### 3. Something you are

This is in the form of what you physically are. The most common form of this is biometrics like thumbprints, facial recognition, retina scans, etc. This in many ways is the most secure on paper. However, it is the hardest to implement. How do you know all 7 billion people on earth have different thumbprints? Or how do you know the machine only allows your face and not the other 7 billion people in the world?

The other 2 to make the 5 is somewhere you are and something you do. These are more common in the digital era and are usually built into the background of many authentication services without knowledge by the user.

A common example of 2 Factor Authentication (2FA), though a little outdated as not many people use it anymore, is ATMs. You have to put your card in (something you have) and then type in your pin (something you know).

## Behind the scenes

In today's context, for anything to be considered secure, it has to contain at least 2 out of the 3 different factors of authentication. That is why most web services, government portals, and online stores try to get you to have 2FA.

After knowing this, you can see why having 2 passwords won't help with authentication as the 2 passwords just serve as 1 factor of authentication. We can assume here that if the hacker has some means of getting the first password, chances are they are able to get the second password.

Many large companies go above and beyond to ensure you are who you say you are. Some do it through subtle backend services not known to the user as well. Here are some ways they do it:

### 1. Tracking of location and login activities

Big corporations like Facebook and Apple often track your login location, activities, and the devices you are logged in. In the event they feel that your activities are suspicious, they may log you out and request for you to authenticate yourself again.

### 2. Limit number of login attempts

This helps ensure that people are not able to brute force (guessing) their way to get your password. After a certain amount of login attempts, they may ask you to wait a certain time before trying again (similar to how your phone will restrict you after a few attempts).

### 3. Remotely log out

They also provide a way for you to log out of your accounts remotely in case you feel your account is compromised.

## Do Your Part

After seeing how companies work to secure your data, it is all in vain if you do not do your part. If you willingly post your account details online, no matter how sophisticated the companies technology is, chances are they are not able to do anything.

So what can you do?

### 1. Set sophisticated passwords

Recommended to have at least 1 uppercase, at least 1 number, and more than 10 characters.

### 2. Save your passwords somewhere secure

If you have to write your password somewhere, it is recommended to put it in a password manager rather than your notes on your phone. Password managers have inbuilt systems to track known hacks to see if your accounts are compromised and you are able to generate secure passwords with it as well. Some recommended password managers are BitWarden, LastPass, 1Password.

### 3. Change passwords once in a while (recommended 6 months)

Though this is troublesome, it ensure that the attacker who knows your passwords have limited time to attack.

### 4. Never give any passwords over the phone or internet (other than if you are actually logging in)

If someone asks for your password over SMS or phone call saying they need it to check your account, chances are it is a scam

### 5. Enable 2FA if possible

2FA reduced the change of unintended login activity. You will likely be notified if someone is entering your account and can quickly take appropriate measures.

All in all, it is highly recommended that you enable 2-factor authentication to secure your important accounts. However, doing so might not be enough due to sophisticated social engineering techniques (see video at the end). Most importantly, never give any of your information, no matter how unimportant you feel it is, to a stranger.

Stay safe

[An intersting video](https://www.youtube.com/watch?v=PWVN3Rq4gzw)
