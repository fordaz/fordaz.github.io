---
layout: post
title: "Is Fail Fast always the way to go?"
description: "Contrasting Fail Fast and Self Healing principles to increase resiliency in Distributed Systems."
tags: [distributed systems, design principles, fail fast, automate everything, self healing]
comments: true
---

## Motivation
Design principles are powerful tools that guide our decisions and choices while building software systems. When applied properly, things just seem to fall into place, but when principles are applied to the wrong situation, they work against us and lead to awkward solutions.

More often than not, multiple principles can be applied to a given problem, and in some cases some of these principles might be in conflict, making it a little bit tricky to apply them.

In this post, we are interested in contrasting the Fail Fast vs Self Healing[^fn-self-healing] principles. Specifically, how should a service A handle the case when it tries to contact a service B to retrieve critical information during its bootstrap phase, and service B is unavailable ? Do we make service A Fail Fast and stop, or do we allow it to come up and keep on retrying until eventually service B is available and Self Heal ?

[^fn-self-healing]: Part of a broader principle called Automate Everything as pointed out in "Scalability Lessons from Google, eBay, and Real Time Games" by Randy Shoup

## Fail Fast
In this context, this means service A stops running, and any external service will see it as unavailable. This will likely happen in conjunction with a critical notification, and will require some external entity (human or automated) to correct the issue and restart the service A after service B has recovered.

To better understand the implications of this approach, let's see what would happen if we had N number of services, with M interdependencies that evolve over time, eg. new services come, dependency from service **i** to **j** is established, others are removed, etc.

Having an external script that automatically restart the failing services, still leaves the door wide open for a situation when dependent services being restarted see one or more of their dependencies as unavailable, creating a race condition that leaves no guarantees to converge to having all the services up and running.

This makes the system fragile, as a given failing service can cause transitive failures to other services, even when they don't depend on it directly. It also creates the problem of having to control the order in which the services have to be started. We could ask, can we come up with solutions to this problem ?, surely we could try to solve it by keeping a very detail representation of the dependencies and have automation around it, but in my opinion this is a problem that should have not existed in the first place.

## Self Heal
A better alternative is to follow the automate everything mantra, and decide to build a internal retry mechanism that would allow the service A to eventually catch up and retrieve the needed information once service B has been restarted. In this approach, no other service sees service A as unavailable, therefore the ripple effects of one failing service are gone.

It is worth noticing, that automation is used in both approaches, when using Fail Fast the dependent service fails, then an automated script can restart it. On the other hand, when using Self Healing the automation is built in inside the actual service itself.

Alerts and notifications are also needed in this approach, but the way the operator or automated service management react is different, now the problem is much simpler, just restart the failing service B or failing services, and rest assure that services that depend on them will self heal eventually.

There are still the questions of what should be behavior of service A during this self healing phase ?, and for that matter what should be the behavior of the entire system under these conditions. Well, the reality is that in distributed systems, failures are expected, not only during the bootstrap sequence, but at any given point in time. This is a challenging problem, with no silver bullet solution, but it is definitely not introduced by the application of the Self Healing principle. This is also a good reason, to equip each service with the self healing mechanisms[^fn-shared-libraries].

[^fn-shared-libraries]: This type of of common concerns for all the services, should be provided by shared libraries, eg. an HTTP client which provides retry strategies, circuit breakers, bulkheads, etc.

Among other options, we could embrace the graceful degradation principle and make the service A to operate providing limited functionality if possible, or even failing (Fast) all inbound requests. Other upstream services, should also behave in the same way, providing some limited functionality at a macro level.

## Wrapping up

Of course, this is not about blindly choosing Self Heal over Fail Fast in every occasion, it is more about remembering that there are different ways to apply a given principle, and that when multiple ones are available, one must understand their implications with trivial and non trivial cases. It is about determining what is more valuable or critical in a given situation and then choosing the principle that helps the most to achieve it.

It is also about remembering that more often than not, a combination of self healing + graceful degradation + arbitrary bootstrap/deployment order leads to more robust and resilient distributed systems.

---

