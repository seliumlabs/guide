# Welcome to Selium's user guide

Ahoy, and thanks for being part of the community! In this wiki you'll learn about Selium, its
components and how to use them.

If at any point you find yourself thinking "I wish they'd cover ...", please
[start a new discussion](https://github.com/seliumlabs/guide/discussions/new?category=ideas). We
would really value your feedback and ideas. Thanks!

## What is Selium?

You may have seen this description of Selium on the organisation readme: "Selium is an extremely
developer friendly composable messaging platform with zero build time configuration." This is a
nice statement, but lacks any depth. So, let's add some, shall we?

**_"Selium is a messaging platform."_** Messaging is simply a method of exchanging data in a
structured way. You may have come across the term "Pub-Sub", which is a method of sending
(_publishing_) the same message to lots of receivers (_subscribers_). That may be a bit of a grey
area to people new to messaging, but you'll definitely know this one: "HTTP". Yep that's right, web
requests are a form of messaging too. It's called an "RPC", or more colloquially "request-reply".

**_"Selium is composable."_** We've built Selium as a collection of parts that you can stick
together to aggregate, manipulate and disseminate your data however you require, much like
everyone's favourite childrens' toy that rhymes with "Pego". You guessed it - Nanoblocks!

**_"Selium is extremely developer friendly."_** Selium is designed from the ground up for developer
ergonomics. Whatever requirements you have for your services and data, you can compose them in at
runtime using our functional API. In other words, Selium has _"zero build time configuration"_.

## Who should use Selium?

_(Anyone with ops-related trauma)_

Selium is designed for software developers. If your project needs to move data around, expose
services, discover services, introduce resiliency, scale out, speed up, produce or consume events,
or otherwise run screaming from HTTP, Selium is for you. Liberate your stack from DevOps, simplify
your deployment pipeline and never use the word "idempotent" again.

Let's [get started!](./getting_started/first_steps.md)
