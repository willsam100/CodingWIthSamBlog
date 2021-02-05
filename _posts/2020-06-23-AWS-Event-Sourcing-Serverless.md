---
title: Event sourcing with serverless AWS
date: 2021-02-05 21:00:00 +12:00
categories: [Cloud, AWS]
tags: [F#, .NETCore, AWS, Lambda]
description: Event-Sourcing on AWS using serverless technologies
# image: /assets/img/2020-06-13-client.png
published: true
---

## Why event-sourcing

Event sourcing is an alternative application architecture to CRUD (Create, Read Update, Delete). Its mains goals are to improve the accuracy and explainability of a software application. It also the benefit of improving application performance.

A CRUD system has limited ability to explain why the system is in its current state. It also can not answer what the user did (if an audit is ever required). A CRUD system can address this with additional systems to track changes. An event sourcing system starts with change tracking and builds the entire application out of an audit log. Performance is improved by performing calculations in the background, returning a precalculated state to the end-user when request. 

Building the application from an audit log (a list of events) provides confidence in the audit log. It also provides more tools for developers to track down difficult bugs and fix them. The series of events is the source of truth. 


## Principles of event-sourcing

- Actions represent an intent to do something. When the action is accepted it becomes an event
- Every change to the current state of the application is an event
- Events are immutable (once created, an event can not be changed)
- Events are in the past tense. 
    - Events are facts of what happened. 
    - The past cannot be changed
- Create new events to change the future 
    - A compensation event can revert an error that was made in the past 
- Current state calculated:
    - an initial state
    - a function to join an event with the state
    - apply the initial state recursively to all events. 
- New events must recalculate the state of the system
- event sourcing is not a top-level architecture
    - Apply to a micro-service, not the entire application

To those who follow functional programming, you will observe that the combing function is a fold. It requires a list of events and a starting state. 


## AWS servless technologies 

Some frameworks can do this. Frameworks come with a cost. It is possible to build a flexible system with computing, queues, and storage from AWS. 

The basic solution: 
- Use a lambda to receive events 
    - store the events in DB as immutable facts
    - once stored, publish the event to an SQS queue
- use a lambda on an SQS queue to re-calculate the state
    - the lambda either pulls the current state and adds on the new event
    - the lambda could load all events and update the state
    - store the current state in another table/DB
- use another lambda the reads the current state as the frontend for the system. 

We have:
- 3 lambdas
- 1 DB, 2 tables (Assuming an RDS eg Postgres)
- 1 SQS queue

## Applying the pattern - Receiving lambda

The first lambda has 3 primary tasks:
- run the rules for actions to event
- save the event
- publish the event

Consider the example of an ATM. A request (action) to withdraw funds is only valid if the user has money. This rule must be run first. If it passed then an event (withdraw funds) can be created. 

The event must then be persisted. With an RDS, this would be saved to the events table. The table must contain a few important columns. The event type is the first (for the ATM example there would be at least `withdraw` and `depost`). The next column is a way to determine the order. A date is the most obvious, but careful thought needs to be placed as to if the date will always be unique. A sequence number could be used, but this has the challenge of ensuring there are no errors with the generator of IDs. It has the additional downside of reducing event ingestion capacity as a DB read will be required to get the next event id. A date column will also be required for when the event is for. This column helps with debugging and for many applications will be a requirement. For most event-sourced systems there will be at least 2 dates. One to track the time the event is for, the other to track the event order. 

The final step after storing the event is to publish the event. There are a few ways to do this. For a small system, the simplest is to publish the ID of the event. The folding lambda can then load the events from the DB. For a larger system, enough information must be published for the folding lambda to do fold to the current state. For high throughput ideally, this would be the current state + the new event. The other alternative is to publish the list of all events. 


## Applying the pattern - The folding lambda

The lambda that calculates the current state, this lambda will contain most of the application's business logic. 

For a small scale system, the fastest (in terms of developer hours) is to load the entire event history, recalculated the state, and then persist to the state table. Each event triggers the whole calculation again. For developers and testing, this approach is quite simple to build. The tradeoff is higher costs (due to the increase computed) and high performance. Clearly, the system can be optimized further. 

For a high-performance system, then the current state + the new event can be adopted. The current state operates as a cache. This brings some coding complexities. The developer (tester and product person) must ensure that all variations of current state + events types are accounted for. Some up-front thought needs to be taken on the types of events as well. The cache is hard to rebuild, so a compensation event must be published. If there are errors (or a missing event type) then the system will need to be updated. The major benefit of event-sourcing is that the complexity is constrained to this single part of the application. It can also be handled by a clear set of unit tests over all possible combinations. 

When the new current state has been calculated, it can be written out the state table (or separate DB for high-performance). Again the simple approach is to perform an UPDATE and only have 1 row per user per state. Storing the history of the state is not really needed, as the intermediates states can be recalculated with the events and end date (a time before now). 

## Applying the pattern - Displaying current state

The final lambda is the simplest of all. Most software applications are read-heavy (ie 75% reads only 25% writes), keeping this lambda simple allows it to scale really well. 

The only task for this lambda is to establish the user to know which row of current state to retrieve. That's it!


## Get started!

Using common services from AWS is a low-cost and more importantly a low-risk solution to getting started with Event sourcing. Event sourcing is a great architecture that I believe should be the default for any new application (and changed to something elsewhere event sourcing is found to be unsuitable). 