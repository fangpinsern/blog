---
title: "Using Cloudflare Workers for Form Submission"
date: 2023-01-30T11:42:19+08:00
showToc: true
TocOpen: false
author: ["Pin Sern"]

ShowReadingTime: true
ShowWordCount: false
# draft: true
---

Contact me forms are part of many websites. These forms are a way for a user or interested customer to contact the business. To manage this service, companies usually require a backend service to manage and process the inputs. However, service deployments and server maintenance are expensive and time-consuming. For small projects, it might be more worthwhile to use serverless technologies like Cloudflare Workers or AWS Lambda.

## What is serverless?

Contrary to what the name suggests, a serverless architecture still uses servers in the background to run. However, the management of these servers is abstracted away and managed by the provider offering this service. This means the developer just needs to focus on developing the code needed to execute the task. The overhead of managing the health of these servers will fall to the provider. You do not need to worry about problems like resource allocation, service health, or monitoring.

Unlike a traditional server, serverless providers charge by the number of requests. This can translate to large cost savings if your project is just in its infancy period and have a small number of requests. In a traditional server architecture, providers will charge based on the time your server is on. Hence even if your server is idle and not serving any traffic, it may still be costing you.

> Developers who want to decrease their go-to-market time and build lightweight, flexible applications that can be expanded or updated quickly may benefit greatly from serverless computing. - Cloudflare

## Use Case

For me, I needed a low-cost way to manage the forms on my [website](https://codingcrayons.com). These forms include `Contact Me` forms and `Early Interest` forms. The behavior for these forms is simple. Notify me when someone has submitted a form. I will manually make a reply to them on the given contact.

## Requirements

1. Notify me in a private telegram channel about any form submissions

2. Quick to deploy

3. Handle different types of forms

4. Low-cost or Free

## Constraints

We are restricting to basic form uploads with only text-based fields and no confirmations sent to the user. The choice to not send user confirmation is deliberate and will be discussed in the appendix. For the current iteration, we will be only sending a notification through a telegram bot to a pre-created channel.

## Choice of provider

There are multiple serverless technology providers. Google has [Google Cloud Function](https://cloud.google.com/functions). Amazon has [AWS Lamda](https://aws.amazon.com/lambda/). I will be using [Cloudflare workers](https://workers.cloudflare.com/) as my serverless provider.

The choice to use Cloudflare workers was:

1. Simple to start. No need for complicated signup processes and no need to provide credit card details.

2. Very very generous free plan. 100,000 requests/day. If I need to handle more than that, it is a good problem to have.

## Getting Started

To satisfy the requirements, we need to solve 3 smaller problems:

1. Setting up a telegram bot and channel

2. Setting up Cloudflare Worker service

3. Integrating

### Telegram

Setting up a telegram bot and a channel is trivial. You can have a read about it [here](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-telegram?view=azure-bot-service-4.0)

To interact with telegram we need the following information:

1. TELEGRAM_BOT_TOKEN - The token to have access to your bot

2. TELEGRAM_CHANNEL_ID - The channel ID you are using for notification

The above should be set as environment variables for security reasons.

Telegram exposes an HTTP API for us to interact with the bot and use it to send [messages](https://core.telegram.org/bots/api#sendmessage).

```javascript
async function sendMessage(message) {
  const response = await fetch(
    `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        chat_id: TELEGRAM_CHAT_ID,
        text: message.toString(),
      }),
    }
  );
  return response;
}
```

The above function takes in a message and sends it as a string (regardless of type) to the `chat_id` specified in the body. the token in the URL will tell telegram which bot is used to send the message.

Both `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` will be passed in from Cloudfare workers environment variables.

Note that Cloudflare workers only support Javascript code. It uses the V8 engine created by Google to manage its service. Here is the [transcript](https://www.infoq.com/presentations/cloudflare-v8/) of a presentation given by the lead of the project if you are interested.

### Cloudflare workers

There are 2 ways to write the function that runs in a worker.

1. Through their CLI tool called [wrangler](https://developers.cloudflare.com/workers/wrangler/)

2. Cloudflare web-based development tool

We will be editing the function through the web development tool. The worker calls the function `fetch` with the `request` and `env` parameters as inputs. The request contains all relevant information about the request.

In our case, we will be making a `POST` request with the content-type of `application/json` as the body.

```javascript
if (request.method === "POST") {
  const reqBody = await readRequestBody(request);
  const reqType = reqBody.type;
  let retBody = "Type not defined in request body";
  if (reqType == "interest" || reqType == "contact") {
    await sendMessage(JSON.stringify(reqBody, null, 2));
    retBody = `Message sent`;
  }
  return new Response(JSON.stringify({ message: retBody }), {
    headers: corsHeaders,
  });
}
```

To handle the post request, we check the method of the request sent. If the method is `POST` we will run our notification logic.

Request Body Schema

| key  | type   | required | description                                                                                                                                                                            |
| ---- | ------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type | string | true     | the type of notification. Should be unique to the form that is sent. This will be used on the owner end to differentiate where this form is sent from and make the appropriate actions |
| data | Object | true     | Json data dump of all the info in the form.                                                                                                                                            |

After we read the request body, we check the body for the `type`. If the type is not equal to what we defined as valid, we will not send the message and just return it. This is not defined as a fail as the request was successful but the body did not fit the specification required.

If the type is allowed, we will run the send message function that sends the message to the telegram channel through our bot. `JSON.stringify(reqBody, null, 2)` is used to prettify the string output on our message. It is not required to send the message. Just stringify will do.

### Integration

If we were to deploy this, it would run. Making a request from Postman to test this will work. However, running it from our frontend app will being about a fetch error.

This is because of a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) failure.

In summary, what a preflight request does is for the client to check if the server is accepting the type of requests you want to send. Preflight requests mainly check 3 things:

1. Access-Control-Request-Method

2. Access-Control-Request-Headers

3. Origin

`Access-Control-Request-Method` asks the server if a specific request method is allowed.

`Access-Control-Request-Headers` checks with the server what header fields are allowed to be sent from the client

Lastly, `Origin` checks with the server what hostnames are allowed to access the resource.

This preflight request is sent through the `OPTION` method with only headers. For example, if the following headers is the response to the request:

```json
{
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type"
}
```

This means the server is allowing requests from all origins with methods of `GET`, `POST`, and `OPTIONS`. Finally, the only header allowed is `Content-Type`.

This is a specific behavior of a browser as a form of security and optimization. If it is blocked, don't waste time trying. That is why the request works on Postman but not on your browser. To fix this, we need to handle the `OPTIONS` request.

```javascript
const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "GET, HEAD, POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type",
};

function handleOptions(request) {
  if (
    request.headers.get("Origin") !== null &&
    request.headers.get("Access-Control-Request-Method") !== null &&
    request.headers.get("Access-Control-Request-Headers") !== null
  ) {
    // Handle CORS pre-flight request.
    return new Response(null, {
      headers: corsHeaders,
    });
  } else {
    // Handle standard OPTIONS request.
    return new Response(null, {
      headers: {
        Allow: "GET, HEAD, POST, OPTIONS",
      },
    });
  }
}

if (request.method === "OPTIONS") {
  return handleOptions(request);
}
```

If the request method is `OPTIONS`, we call the `handleOptions` function and check the headers. If the header has `Origin`, `Access-Control-Request-Method`, and `Access-Control-Request-Headers`, we will send the `cors-headers`. Else, we will just respond with a standard options response which is to tell the client what methods are allowed by the server.

Once we put everything together, it should be able to run as intended. Click on `Save and Deploy` and you should be able to access the service through the link provided.

The full Cloudflare worker function can be found [here](https://github.com/fangpinsern/using-cloudflare-workers-as-form-handler/blob/main/worker.js)

## Drawbacks of the above solution

Currently, the logic is very simple. However, as the project/company size grows, the logic might increase in complexity as well. At that time it might be worthwhile to migrate over to a dedicated server instance instead.

On top of that, serverless solutions usually rely on shared packages. This means you are limited to the packages made available by the provider. For example, on a conventional node server, if you want to send an email on form submission, you can use packages like `nodemailer`. However, you cannot do that with a Cloudflare worker as that package is not available.

Lastly, when developing at scale, working in a team setting might be troublesome. However, this might be because I am using the web console. Maybe the CLI tool might help in a team setting.

## Conclusion

Now I can handle the form submissions without a full backend server deployment. Sometime in the future I may want to integrate a database into the service so I can keep all the information accessible rather than through a private telegram channel. Till then, this solution will work fine.

Happy Coding!

## Appendix

### No user confirmation on form submission

The initial plan was to send the user an email informing them that I have received the email and will get back to them. However, after certain considerations, I decided against it. As a public `POST` API, it exposes some security concerns.

There is no efficient way to filter out if a person is genuine or malicious algorithmically (that I know of). If a malicious attacker wants to spam a person's mailbox, they can access the API and just enter the victim's email address. Furthermore, if we are using a third-party service like [SendGrid](https://sendgrid.com/) or [MailGun](https://www.mailgun.com/) which charges by the volume of emails sent, it may incur large financial costs.

Thus I decided to manually do it till I cane figure out a way to protect the API.
