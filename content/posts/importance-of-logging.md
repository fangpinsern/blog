---
title: "Importance of Logging"
date: 2022-01-24T11:19:44+08:00
showToc: true
TocOpen: false
author: ["Damien"]

ShowReadingTime: true
ShowWordCount: false
---

When thinking of building applications, many think of how the application works, how to design the system and schemas. But not much thought is put into writing logs for the application. However, logs may just be the most important thing in an application.

If done right, you will not have to think much about it during development. If not done at the start, you may find yourself reading your code again or changing that 'println' or 'console.log' line of code.

## Why is it important?

Logs are, in a way, an inner view of what's happening in our code. It provides sort of a timeline of what occurred in our when the code is run and where the issue lies when there is an error. Knowing where the issues lie usually is half the battle won. More importantly, knowing the events that led up to the issue will help debug the error as well.

In some use cases, logs provide a backup plan when some major error occurs in mission-critical infrastructure. In the worst case, companies can manually read through their logs to ensure correctness rather than doing guesswork.

One such example would be in banks when a bug caused duplicate transactions. This [happened](https://www.channelnewsasia.com/singapore/dbs-duplicate-transactions-double-deduction-credit-debit-cards-1960351) to DBS Bank in Singapore in June of 2021.

In summary, there was a glitch in the system that caused double transfers for some users. Hence the users' account was credited twice. Having reliable logging systems, in this case, helped as they were able to trace which accounts had duplicate transactions and refunded those accounts.

## Where to add logs?

Location of where to add logs differ from person to person. I do believe more the merrier. A log should be placed anywhere a major event has been completed. For example, after you have sorted something, or looped through a list, etc. This, in a way, gives you a breakpoint where you know that up till that point, everything is correct and narrows down the search for any issue that arises.

Any location where we know the program will exit with an error or exit early should have logs as well. This will allow your team to monitor known errors and see if it is commonly faced. For example, in the case of web applications, if certain 404 NOT FOUND errors are constantly occurring, we can look into why these 404s are constantly called. Maybe it's a mistake in the button on the frontend code or a bad design choice that has caused the user to keep going to that page.

In summary:

1. Log after major event or computation has been completed + after a function returns

2. Log where errors will occur or early exit

## What should be in a log?

Logs should contain key information that makes it easy to filter, sort, and extract information about what is going on. Common information in a log are:

1. Time

2. Message

3. Location where a log is called

4. Log Type

5. Request ID

The list above is not exhaustive. More information like IP addresses and request methods can be added for different applications. However, those above are the basic ones.

### Time

We include time to see the exact moment something occurred. This allows you to trace back and filter the issue when it is reported. In a time when we have concurrent programming, it is tough to tell what happened when.

### Message

Information as to what has occurred at the point in time. It would be good if this message provides specific information like what happened, what were the inputs when it happened, at which function it happened etc, so it would be quicker to resolve. However, this is a complex balance between privacy and ease of resolving issues. Information like passwords and credentials should never be logged.

### Location where a log is called

A location like a line number and file name can also be placed in the logs. There may be times when the same error message may pop up and it would be good to know where the log is made. For example, the log came from a file that is recently edited, there is a high chance that the new code has issues.

### Log Type

Logs can be split into different types. Most commonly, it would be these 5 types in increasing order of importance:

1. VERBOSE - nothing much
2. DEBUG - just for development debugging purposes
3. INFO - information about the system running
4. ERROR - error occurred
5. FATAL - ded

These different log types will make filtering your logs much easier. In a full-scale application, every action may generate hundreds of lines of logs. Having different log types will allow you to split them into different files. There are also applications like `sentry.io` and `datadog` that can help you monitor your logs and notify you if there is an issue.

### Request ID

This naming might be a little specific to web applications. However, there should be some form of this in other applications as well. As mentioned above, every action above might bring in hundreds of lines of logs. And if your application serves thousands of clients, the lines of logs may get hard to find the cause of the issue. Hence having a request ID would help with this.

What a request ID does essentially is to give every action a user makes a unique ID. This ID will not change throughout the lifecycle of the action. When it passes a log line, the request ID will be reflected in the log. Hence when an error occurs, we can filter the request ID to get the whole trace of what happened leading up to the error.

Specific to web applications, this request ID can be sent back to the client so when unexpected behavior occurs, there will be a point of reference on what the user did rather than expecting the user to remember every step they did for you to resolve the issue.

## Storage

The above information to include in logs are just basic suggestions. You can add more or less as you please. However, with thousands of lines of logs, how do you store them?

Most loggers do have different modes of storage. 3 common ways are:

1. Print to console
2. Write to a file
3. Write to a database

Print to console is usually great but generally not recommended. It is non-persistent, hard to filter and search, and sometimes troublesome to access. However, it is the fastest to set up.

Writing to file is more common but with more and more logs, there the file may take up a significant amount of storage on your machine. Hence some loggers do date the logs and delete the old ones after a certain time. Files are easier to search than console, but I don't think `Crtl-F` is a very good user experience. It is relatively easy to set up compared to setting up a database.

Writing to a database is the best option in my opinion. Using the query language of a database, we can extract just the logs of specific request IDs or specific log types. However, this is a little troublesome to set up as you need to maintain a database instance. If you use this, you may be able to use some of the monitoring platforms below to help you with monitoring issues.

## Conclusion

Logging is the last thing people think about when coding. But logs are important. They are the sort of thing that you just have to do it once, do it right and forget about it. When something happens you will be thankful that you did it.

Happy Coding

## Log Monitoring Platforms

[DataDog](https://www.datadoghq.com/)
[Sentry](https://sentry.io/)
