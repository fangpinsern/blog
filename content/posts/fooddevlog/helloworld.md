---
title: "[Dev Log] Redefining Food Discovery - Part 1"
date: 2023-07-19T10:30:34+08:00
showToc: true
TocOpen: false
author: ["Pin Sern"]

desc: "Solving the hardest problem known to mankind. What to eat?"

ShowReadingTime: true
ShowWordCount: false
---

No matter what type of person you are, we all have one thing in common. We need to eat. We eat 3 times a day on average. Yet we seem to have difficulty answering the question "What to eat?"

> What should we eat?

// insert comic

## Existing Solutions

There are existing solutions to these problem.

### Google Maps

For me, I use google maps and their "food near me" feature to help me find food places near me. Then based on the quantity and quality of the reviews, I will choose where to eat.

For those of you that do not know, you can actually type "[thing you want to find] near me" in the google maps search bar and it would give you a list of things that fit the [thing you want to find] near you. For example, if you want to find somewhere to eat dessert, you can type in "dessert near me" and it would give you suggestions.

This works if you want to find somewhere nearby to eat. However, it does not actively provide you personal recommendations that might be suited to your taste.

### Food Blogs (Recommendations)

There are tons of food blogs and instagram pages recommending food everyday. If you see something you like, you can save it and you can plan to specially go there to eat.

The issue with this is that you need to remember that you want to go to this place. You may forget that is a place you want to visit or not know this place is nearby.

## Idea

The current solutions above do not fully solve the issue we are facing. Hence, we want to build a simple to use solution that would give users recommendations on what to eat. The choice of where to eat should be based on certain given criteria.

// insert comic

With this idea in mind, we have 3 options to achieve it.

### Option 1 - Chat Bot

Make a telegram bot that can give you recommendations on what to eat. You can add it to your friend group chats and it can recommend you places to eat when y'all have no idea where to go. It will be some sort of personal assistant giving you personal food recommendations to your queries.

Pros:

1. Telegram is widely used. So it can reach a big group of people instantly.

2. Free to build on

Cons:

1. Design and control of the bot behavior is limited to what telegram exposes to us.

2. Limited room to expand in terms of features

3. If we are aiming to use natural language, it might be expensive

### Option 2 - Mobile / Web App

Build a mobile/web app that helps solve this question. It will be an app that people can use to quickly get recommendations on what to eat. Search functionality will help with discovery of different restaurant that users may enjoy.

Pros:

1. More control over user experience and behavior.

2. Not everyone has telegram

3. It is cool

Cons:

1. Need to upload to the respective app stores and get approval.

2. On top of that, need to convince people that your app is worth downloading.

3. High complexity and effort

### Option 3 - Food Blog

Write and maintain a food blog that gives recommendations. We can have weekly posts that will give recommendations. Adding a newsletter can further enhance the experience and give weekly recommendations to people.

Pros:

1. Easy to start

2. No need to reinvent the wheel - There are already many blogging apps available out there

Cons:

1. Cannot customize the user experience

2. Crowded space - There are thousands of food blogs out there more established. Difficult to attract and differentiate

### Verdict

After careful consideration, I decided to go with option 2. Option 2 gives flexibility and allows me to expand the feature sets of the app freely. Telegram bots though powerful, is limited to interactions that most people may be unfamiliar with.

## Mock up

Basic mock up of how the app would look like. Please give chance. I'm a software engineer. Not a designer. But it would look rather basic for now and we can iterate through the design in the future.

// Insert mock ups with short captions

## Follow Up Action

1. Finalize the feature and user stories for the app

2. Learn how to build a mobile app with React Native

3. Choose colors for the app

## Conclusion

The first thing people may ask is, "How are you gonna make money?" Well, it is not really about making money at the moment. The problem it solves might not even be real. It is about the process of building something from scratch. Contrary to popular belief, having multiple years of experience at one company does not make you a better software engineer. Yes experience is important. But from a technical standpoint, the years of experience gets you familiar with the companies internal tools. And in big companies, the job gets very narrow and focused. Furthermore, every company uses different technologies. So the important thing is to be adaptable. And well, you cannot be adaptable just doing the same thing every day hahaha.

So here I am trying to make something.

## Others
