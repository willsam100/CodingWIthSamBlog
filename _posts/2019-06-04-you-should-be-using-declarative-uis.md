---
layout: post
title: You should be using Declarative UIs
date: 2019-06-04 20:36:21.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Apple, declarative, DeclarativeUI, GUI, iOS, MVU, Swift]
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591349820'
  _yoast_wpseo_primary_category: '29'
  _yoast_wpseo_content_score: '90'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'

# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
description: Apple has just announced SwiftUI a declarative UI framework.
  Declarative UIs are the future for GUI development and this release marks it's
  here for apps!
permalink: "/2019/06/04/you-should-be-using-declarative-uis/"
---
Apple has just announced SwiftUI - a library that allows for a declarative way to describe a UI. This is a game changer and the best time to programming if you code for iOS.

Declarative UIs are the future for all GUI development, they have transformed the web, and now transforming mobile app development.

## What we already know?
Declarative UIs are a lot better than the traditional way that we have building UIs. The traditional way to build a UI was one of two ways
- some XML based approach - a storyboard for iOS (Gui to edit but under the hoo it's XML), Android had XML, Xamarin/WPF has XAML
- Imperative code - the code built the UI and has to run in order to do this.

Each had its pros and cons. The XML approach was great because you could 'see' how the UI would look. The downside was that there was no type checking over what was being built.

The code approach solved the type-checking problem, but now you can't 'see' how the code will look. You can mentally execute the code, but you might be wrong. In my experience, I found most people preferred the XML approach - it's just easier to understand.

## The best of both worlds
Declarative UIs are much better because they are the best of both worlds. It is possible to 'see' the UI as the code describes exactly what should be there. More importantly, because it is code, it is now type checked.
This means that all those late nights trying to figure out why that value isn't appearing, well, those are over with a declarative UI.

## New tech scares me - what about bugs
I agree that we should caution of new tech. Assuming that you have customers/clients depending on your work, it's important that we take care before introducing some unproved techniques into our apps.

### Not too fast though

Declarative UIs are not new! Most frameworks have moved them. The most popular one is React (React Native) which shook up how the web has done things. Flutter is new on the block and taking the same approach. My personal favourite is one for Xamarin Forms called <a href="" title="https://fsprojects.github.io/Fabulous/">Fabulous</a>.
In fact, the idea of declarative UI actually goes back as far as 2012 with the language (and framework) Elm.
Declarative UIs are just a modern spin on functional programming (which is the same age as OOP, and older as a general concept).

## Do you still need MVC?
Generally, apps are built with some MV* pattern. The two most common are MVC and MVVM. I started looking for MVVM alternatives in my Xamarin apps a <a href="" title="www.codingwithsam.com/2017/02/08/bettermvvm/">few years ago</a>. It lead me to declarative UIs and and a new architecture called MVU - model-view-update. <a href="" title="https://fsprojects.github.io/Fabulous/">Fabulous</a> follows this and it's a similar approach as React + Redux.

One simple way to understand how this works is to imagine that anytime a change to the UI is made, the whole UI is thrown away and a new one is made. In code, this is how 'feels', in reality, smart things happen so only the important changes and the UI is still super fast.

the MVU architecture is a great compliment to a declarative UI as it aids in simplifying the code, and makes for a great productivity boost - no longer chasing down that state bugs because you forgot to update some variable somewhere when you loaded that random page.

It's up to you if you want to change - having an understanding of MVU is worth the investment, even if you still use MVC.

## Where's the risk
So really the only risk here, is that Apple might have a few bugs in there framework. Very unlikely given it's rather simple to build these libraries.
The only other risk that you might have, is that you have never written a declarative UI before. A demo app and an afternoon of playing will solve that. Warning - you won't want to go back once you figure out how simple it really is.
Declarative UIs - it's 2019 people!
Helping developers love their job && ship perfect code

Coding With Sam
