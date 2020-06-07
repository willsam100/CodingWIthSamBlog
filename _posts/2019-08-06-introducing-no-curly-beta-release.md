---
layout: post
title: Introducing No Curly - beta release
date: 2019-08-06 21:12:47.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Tools]
tags: [C#, F#]
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591326083'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_content_score: '30'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2019/08/06/introducing-no-curly-beta-release/"
---
No Curly - a chrome extension to convert C# to F#. Click the link at the top of this blog to try it out!

## History - Why this tool matters
For most of my career, I have been building mobile apps. That is how I got paid. During this time I have also been learning how to write functional code as in functional programming. Xamarin was the first cross-platform to combine them with F#. F# is currently not as popular as C#, and many of the code examples are written in C#. When using Xamarin, it's also not possible (or rather hard) to keep adding platform code ie an Android Intent. The C# code needed to be converted to F#, and I grew weary of making the same basic changes  - syntax recording of types and names.

This birthed the idea of the tool that could do this. The first cut of this sucked. It was a find and replace. One of the challenges I had was that the code I was converting was mostly on the web. It was either code documentation on a GitHub readme or a blog post. Additionally, I had to be realistic, there are more C# devs than F# devs. So documentation  for Xamarin will continue to grow at a faster rate using C#

## The goal
This lead to some requirements, and me taking the tool a little more seriously. The tool must convert C# code to F# and it must do it on the web. The code may not be completed or perfectly valid. It must still work to convert it. The tool does not need to convert an entire application (since that could go into a C# project, and would not risk a conversion error). To simplify the goal, I wanted to convert all C# documentation on the web to F# (this is still my goal and I realize this is really, really hard) - No Curly as a tool was born.

To meet the requirements that I had, the tool would need to be able to understand C# and F#. It would need to be able to format the F# code. The tool would also need to be easy to use and not require lots of copying and pasting. Given the spec is to convert code on the web it is clear that desktop or mobile apps were not suitable. A web-page to convert the code was considered, but then the user is always copying code, going to the site and converting it. It also does not create a nice experience reading the blog post. I found that a browser extension (Chrome for now)  could convert code on the page. It could also call a server - the conversion engine. The result is No Curly - a Chrome extension that reads a webpage and adds an F# translation to every C# piece of code on the page. If the tool works, then this is very close (IMO) to meeting my goal of converting all C# documentation to F#. Visit any webpage, click on No Curly, and in a few seconds the webpage will contain F# - those curly braces removed :)

## Future direction(s)
No Curly is not yet perfect. I do think that it almost reaches the definition of beta. Most of the time it works or the output is very close, requiring a few manual changes; nothing repetitive. The current plan is to keep improving the tool, specifically for code related to business. That means language features that are used in code for Xamarin and possibly ASP.NET. Complex algorithms and obscure language features are not yet in scope ( though I'll always appreciate feedback). I have also had feedback that No Curly would be great for learning. It does a direct translation of C# to F#. If you are learning F#, or are in education and want to help I would love to hear from you (just saying hi on twitter is fine too @willsam100).

## How does the tool work
As mentioned earlier, the first version was a simple find and replace. It wouldn't scale. I needed a solution that could understand C# and F#. So for that, I used both the C# compiler and F# compiler. I just stitched them both together and observed. It was better than find and replace, but still hopeless in achieving the end goal - all C# documentation to be converted. There were two main errors. The first is that F# is more strict that C#, and there are many cases were the C# syntax tree could not be converted directly to F# using one pass. Just as Fable has an intermediate tree going to JavaScript, I also had to introduce a custom abstract syntax tree (AST). The second challenge was the formatting. Formatting F# is a separate task and seemed like something that No Curly should not need to know about. Fantomas to the rescue.

Using the two compilers, an intermediate representation and Fantomas leads us to the current state of the system. Most of the hard work has been done manipulating the intermediate tree state to produce valid F#. These challenges include: if (else) statements, async code, C# pattern matching, C# class ordering, for loops, let statements (these need to be enclosed in code to create a valid F# AST). Not all of the conversions have been completed, but these items are among the hardest for a translation. I'm sure there are still a few hard things. But there is a long tail of small language features that will never end - all those tiny features C# has been bloated with.

To take it for spin  - click the link at the top 'No Curly' on the linked page this is a link to the Chrome Extension store. PS: I'm not good at HTML. The linked page is a simple place to use the tool with any C# code. Help styling and making the page better would be appreciated.

**Leave a comment (or better share this post) if you want this tool to continue to be developed or you have any feature requests.**
