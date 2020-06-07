---
layout: post
title: Technical debt is not real
date: 2019-08-04 09:58:27.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Best-Practice]
tags: [Best-Practice, Communication]
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1590990953'
  _yoast_wpseo_primary_category: '74'
  _yoast_wpseo_content_score: '90'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_og_title: Technical debt is not real
  mashsb_og_description: Misunderstandings by developers lead to a loss of productivity
    and a not-so-fun-to-work-with codebase. By improving our communication and being
    clear with the trade-offs we're making we can avoid these problems and enjoy a
    happier life writing software.
  _yoast_wpseo_metadesc: Misunderstandings by developers lead to a loss of productivity
    and a not-so-fun-to-work-with codebase. By improving our communication and being
    clear with the trade-offs we're making we can avoid these problems and enjoy a
    happier life writing software.
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2019/08/04/technical-debt-is-not-real/"
---

## It's about communication - technical debt is a metaphor
Technical debt is a metaphor to help communication between those writing software and those making business decisions. The metaphor is pretty good, which is probably why have all heard of it. As with all metaphors though, it's not the real thing. When taken too far, the benefits are lost and using the metaphor becomes harmful. Poor communication leads to misunderstandings. The company suffers as a result (and employees indirectly). Misunderstandings make our job as software developers suck.
This is a post to help us be a bit more mindful in our communication. By highlighting where the metaphor is not very useful, we can choose to instead talk in terms of trade-offs. As a result, we can make better choices. We can enjoy our jobs more and the software we write will be better (whatever 'better' means for the company).

## Where did the term come from?
Ward Cunningham was the first to use the term. He used the term to persuade his boss to justify a refactor of the code base and so used financial metaphor. It seems this was in 1992. More details are <a href="https://www.agilealliance.org/wp-content/uploads/2016/05/IntroductiontotheTechnicalDebtConcept-V-02.pdf">here</a>.
The world was a different place in 1992. There was less software, and management probably did not understand it very well - how could they? The Internet was still growing up, and most software was shipped via disks. Updates to programs were hard. The software had to be correct, and fixing bugs was expensive. Today, we have the Internet to update software. There is software all around us, from our computers to web apps, to mobile apps. We have grown up with it over the last 25 years. It's now possible to run a business purely on technology - even a book store such as Amazon.

All of this is to suggest that maybe the technical debt metaphor is not as helpful as it once was. Maybe management has a slightly better understanding of the software.

## What about the developer side of the metaphor?
The metaphor only works if both sides understand the meaning. This implies that developers have an understanding of financial debt. I think this holds for the general concept. Many of us understand debt. However, I think that metaphor from a basic understand implies assumptions. A basic understanding of debt assumes that it will be paid back. Savvy business people do not always make this assumption. I know real estate investors that hold interest-only loans and never intend to pay it back. I have not heard of a software developer taking on technical debt with the assumption of keeping it.

## Is the metaphor detailed enough
The concept of not paying a financial loan may be a new concept to you. It raises the question, under what conditions would this be a good idea. To find an answer, at least three things would need to knowen (estimated). The size of the upside - asset.opportunity. The amount of debt that would be taken on and the interest rate. If the opportunity is large, and interest rate low with a mid or low size loan then paying off the debt might not make sense. Especially if the asset increases in value. When it comes to technical debt, these are generally not the conversations that are had. It would be good to have these conversations, but that involves some kind of guessing (how big is the opportunity/market). Guessing is not very good, especially when dealing with something as tricky as the market opportunity. Instead of trying to guess the interest rate (and other details), consider the thing itself. Let's talk in terms of trade-offs and work to find out through experimentation what the real cost would be.

## The metaphor is for developers to management
I still think the metaphor is useful, particularly when talking to non-technical people. However, I think the metaphor serves very little use among developers. The metaphor fails to be specific enough. Developers understand code, so talk in code. We are trying to make good decisions given some criteria (these criteria are discussed next), using concrete language, and talking in terms of a trade-off should make this easier. Trade-offs such as when the code should be changed, or even if it should be changed. Will this be a permanent decision? Is this a trivial choice, ie easily reversible? What are the consequences if we are wrong?

Many of these questions help to find a great choice, which may not have been first thought of.

## There are many questions
Given there are many questions, this highlights another aspect of software development. It's all about trade-offs and not just along the time axis. Some choices are about security, some about performance. Others are about who can maintain the code (Can we find React or F# developers). These choices also range from conversations at the business/product level down to the details of a single line of code. It's trade-offs at every possible level. Many of these choices we don't label as technical debt since they simple; the best choice is clear and it's easy to change (e.g. following a software design pattern). Others are more important, with greater consequences.

## Finding answers
Given all the questions at all these levels, how would anyone get anything done? If these were all to be asked constantly, clearly things would grind to a halt. To keep things moving, context and heuristics make all the difference. Stating the heuristics upfront and defining the context can make it easier for people to make the right decisions. This is also known as culture. Facebook is known for this by the statement 'move fast and break things'. This is a guide on how to make technical trade-offs. Making code changes that might not work is accepted at Facebook. Would the same be accepted as a bank? I doubt it. At Facebook, breaking things is not considered technical debt. At a bank, things proceeded with more caution. Netflix has its culture deck too. They prefer making long term decisions. That might look different to Facebook and a bank. To use the term technical debt is unclear between developers, as the business (context) has different definitions of what is accepted and what is not.

## Who you're talking to is key
Being explicit about context and trade-offs (move fast and break things) won't solve all the hard questions though. To improve the choices being made we must consider who we are talking with. If it's another developer, then steering clear of the term technical debt and talking about trade-offs is probably going to be better. Fewer misunderstandings. If the people managing the product were technical and understand code then continue in terms of trade-offs. Additionally, if the people managing your product and don't know anything about finance, then it's also better to talk about trade-offs. Leave the term technical debt for the high-level execs, the business owners who have little understanding of software and solid understanding and business practices.

Misunderstandings by developers lead to a loss of productivity and a not-so-fun-to-work-with codebase. By improving our communication and being clear with the trade-offs we're making we can avoid these problems and enjoy a happier life writing software. Being mindful of the company culture (and shaping it) is crucial for how these trade-offs will play out over time and consequences.
