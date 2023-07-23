---
layout: post
title: 'Notes on Resilience'
categories: architecture system-design
---

## Handling Retries upon Failures

In a distributed system, the naive approach for having a client retry a request to your service, is to simple make the client do a retry of the request as soon as the failure happens. And if that retry fails, then make another attempt immediately, and repeat that approach.

This has some serious consequences, when you think of this for systems built at scale.

### The Problems

- **Server doesn't get time to recover**
  - When too many clients bombard the server with too many retry requests, we do not give it time to recover. So how to do you space out your retries in order to give server time to recover ?
- **Post recovery, clients bombard server with retry requests**
  - Too many retry requests are waiting from multiple clients, and unaware of the situation, they just bombard the server with a ton of requests, causing the server to again go down

### The Solutions

There are a couple of industry standard solutions for these problems - Exponential Backoff with a retry schedule, and by adding what we call a Jitter. Let's get into the details to understand better

- **Exponential Backoff**
  - When the server goes down due to some issue, the steps it needs to perform to recover, and the amount of time it takes to recover could vary. For eg. establishing DB connections, rebooting of the instance, maintenance activities all take different amount of time. So retries need to be spaced out to give enough time for recovery
  - The easiest solution is to setup a retry schedule for all our clients to space out the requests to the server, thus ensuring distributed load on the system. We use a technique called **[Exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff)** here. For eg. As the number of retries progress, the wait times exponentially go up as well, and that peaks at say 5 seconds or gets capped at a certain limit of retries, both of which can be configured. (for eg. Retry schedule could be this - 100ms, 250ms, 500ms, 1000ms ...)
- **Thundering Herd and Random Jitter**
  - But this has some issues, what if all clients try this approach, the server could still be bombarded with requests at certain intervals - this is called the **[Thundering Herd](https://en.wikipedia.org/wiki/Thundering_herd_problem)** problem.
  - The solution to this is to add a random variance (for eg. + or - 10%) to the wait times before retries. This is calling adding a **Jitter**, and this helps unevenly space out the client retries, thus providing the space for the server to handle these incoming requests.

Considering the failure scenarios in a distributed system and taking the right steps to handle them is of extreme important in building APIs that are robust, predictable, and fail-safe. The above techniques help achieve the same easily in your technology stack. Also we can talk about Idempotency in an upcoming post where I will explain another technique of ensuring failures are handled safely.

Keep learning !

**More sources**

- [How Braintree solved the Thundering Herd Problem](https://medium.com/paypal-tech/thundering-herd-jitter-63a57b38919d)
- [Be a good client: jitter](https://medium.com/@juliozynger/be-a-good-client-jitter-19714ced379)
