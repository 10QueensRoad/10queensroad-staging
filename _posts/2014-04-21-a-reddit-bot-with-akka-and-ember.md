---
layout: post
title: "Building a Reddit bot with Akka, Ember and Spray"
tags: [akka, emberjs, scala, spray]
author: "Andrew Miller"
---
{% include JB/setup %}

Building an application with technologies you aren't familiar with is often a
good way to evaluate those new technologies, and compare them to your current
tools. [The code is here](https://github.com/A1kmm/redditdespammer).

I built an advanced anti-spam Reddit bot (including a web-based administrative
console, and user management / permissions system), using Scala, Akka to support
an actor-based architecture, Spray (providing both HTTP client and HTTP server),
and EmberJS (for the client side of the management console). I went in to this
project a complete newbie with EmberJS (recently I have mainly been doing
AngularJS), and this is my biggest project so far to use Akka or Spray, so this
was a good opportunity to evaluate those technologies.

<!--end excerpt-->

It is worth noting that this project doesn't really need Akka for concurrency
reasons - reddit doesn't allow a single IP address to send more than 30 requests
in a minutes (and the bot uses Akka to rate limit to this rate), and so unless I
had a giant pool of IPv4 addresses (reddit is IPv4 only), this is always going
to be the limiting factor anyway. However, the complexity overhead of this model
wasn't too bad, and having lightweight actors made the code quite simple to read
compared to, say, juggling with Futures or putting together pipelines for Netty.

I found Akka Persistence to be a helpful feature (note that the feature is
not yet final). The idea is that you define a Processor which processes messages,
and messages marked as Persistent (in my app, commands that have been validated)
are persisted and replayed in future. If needed (I haven't implemented this yet)
Snapshots which wrap up multiple Persistent messages to get the current state
can be added. Every time the app is restarted, all Persistent messages in the journal
after the last snapshot are replayed to get the new state of the application.

One thing to be aware of with this approach is that migration then probably
requires writing code in the application (for example, by adding new classes
to replace the current classes, while keeping the old classes and writing migration
from the old to new).

Compared to Angular, I actually like the Ember design better. In Angular,
models are simple JavaScript objects, which means that Angular needs to do dirty
checking and recompute all computed properties all the time. Ember provides its
own properties system built on top of its own inheritence system built on top of
JavaScript prototyping, and this properties system allows computed properties and
their dependencies to be declared, so that Ember can work out exactly what
properties it needs to recompute.

Handlebars (the templating system used with Embers) forces you to not put any
computation in templates at all. With Angular, I often put very basic logic that
is more related to the 'view' than the model or controller in the template (for
example, &lt;div ng-show="type == 'Blah'">...&lt;/div>); even basic comparisons like
this are not supported in Handlebars templates - this is by design, to force you to
put logic on the controller, a helper, a view, or a model as appropriate (although
there are hacks out there to circumvent this and help you move that into the
template). Forcing the developer to do things the idiomatic way does have the
benefit of ensuring consistency and ensuring that no logic bleeds into the
templates.

So far, I've found that Ember had a steeper learning curve to get started, but
after that, extending it to support functionality that goes beyond the basics
is easier and more predictable than Angular.
