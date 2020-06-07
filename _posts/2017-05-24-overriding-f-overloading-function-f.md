---
layout: post
title: Overriding functions and overloading functions in F#
date: 2017-05-24 19:52:26.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: OOP
tags: [F#, Functional-Programming, OOP]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1590833717'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/overriding-methodsfunctions-in-f/
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '47'
  ampforwp_custom_content_editor: ''
  ampforwp_custom_content_editor_checkbox: ''
  ampforwp-amp-on-off: default
  _yoast_wpseo_metadesc: Overloading methods is common practice in OOP. OVerride functions,
    as well as overloading functions in F# is a little different. Here is the needed
    keyword
  _yoast_wpseo_content_score: '30'
  dsq_thread_id: '6141489708'
  _wp_old_slug: overriding-methodsfunctions-in-f
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2017/05/24/overriding-f-overloading-function-f/"
---
Overloading methods are common practice in many OOP languages, but in functional languages (using a functional style) not only is this not common practice, sometimes it's not even possible. This short post highlights why and shows the alternatives.

## prerequisites
- A simple understanding of an OOP language (C# or Java)
- A simple understanding of a functional programming language. (Code examples are in F#)

## The Problem
As always we need some code to talk about. Here's some OOP style using F#
```fsharp
type LibraryClass() = 

    member this.PrintNumber (myNumber: int): unit = 
        printfn "%f" (float myNumber)
    member this.PrintNumber (myNumber: float): unit = 
        printfn "%f" (float myNumber)

// Using class
let myLibrary = new LibraryClass()
myLibrary.PrintNumber 10
myLibrary.PrintNumber 10.0
```


No surprises here, everything works as expected. So what happens if we want to write this in more of a functional style. Here's a first attempt:

```fsharp
module Library = 
    // Does not compile
    let PrintNumber (myNumber: int) = 
        printfn "%f" (float myNumber)
    let PrintNumber (myNumber: float) = 
        printfn "%f" (float myNumber)
```

<em>This does not compile</em> and gives and the error <em>Duplicate definition of value 'PrintNumber'.</em> Argh, at first glance this would appear like F# is missing some basic machinery.

## Discussion
The reason why this is happening is that the F# compiler cannot determine the difference between the functions. Either the name needs to be changed or the number of parameters. If each function is rather different in nature then changing the name might be a better way to go. However, if your design is pretty good, then chances are these functions should have the same name.

## Last Effort
Here is the final effort and fortunately this code compiles:


```fsharp
module Library = 

    let inline printNumber myNumber = 
        printfn "%f" (float myNumber)

Library.printNumber 10
Library.printNumber 10.0
```

The keyword ```inline``` comes to the rescue. ```inline``` can be used to relax the type inference on a function, despite the implementation not doing anything related to this. When the compiler reads ```inline```, instead of creating a function, it replaces the caller with the function itself, ie it 'inlines' the function. ```inline``` can also be used for performance-related tasks as a last effort. For a full read see <a title="https://docs.microsoft.com/en-us/dotnet/articles/fsharp/language-reference/functions/inline-functions">Inline Functions</a>.

As a result, when a function is inlined, its type inference is the most general it can be. I think a quote is relevant here
<blockquote>With great power comes great responsibility</blockquote>

Use ```inline``` sparingly, only to make the code more readable. As a guideline, the types of the inlined function should still have some restriction, not ```obj -> obj```. If you have this as a problem, find a way to restrict the types. Perhaps by using a discriminated union (DU) and passing that through, but each case will differ.
Now you can use this with iOS and Android, where overriding and overloading typically can't be avoided. [Android and F# are at last friends]({{ "/2017/08/15/android-and-f-are-at-last-friends/" | relative_url }})
