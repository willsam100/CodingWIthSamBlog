---
layout: post
title: How to turn runtime exceptions into compiler errors
date: 2016-12-03 08:40:48.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Functional-Programming]
tags: [Exceptions, F#, Interfaces, Option-Type]
description: When APIs are written poorly, the abstraction leaks.Runtime
  exceptions can be the result. Overcome this by turning runtime exceptions into
  the API contract!
meta:
  _edit_last: '1'
  _wpcom_is_markdown: '1'
  mytory_md_path: https://github.com/willsam100/CodingWithSam/raw/master/Option%20type%20-%20part%201
  mytory_md_text: ''
  mytory_md_mode: url
  _mytory_markdown_etag: '"a30ee482412972028b7a2d453d3dc99f844922c4"'
  mashsb_timestamp: '1574519511'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/net-none-none-none-or-exceptions/
  dsq_thread_id: '6135227955'

  _yoast_wpseo_content_score: '30'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '28'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2016/12/03/net-none-none-none-or-exceptions/"
redirect_from:
  - net-none-none-none-or-exceptions/
---

Creating a clear interface/API is key to writing <a href="https://www.amazon.com/gp/product/0132350882/ref=as_li_tl?ie=UTF8&amp;camp=1789&amp;creative=9325&amp;creativeASIN=0132350882&amp;linkCode=as2&amp;tag=codingwithsam-20&amp;linkId=4a9afb03ee2cebd83bec7e5f2b1abdc2" target="_blank" rel="noopener">Clean Code</a><img style="border: none !important; margin: 0px !important;" src="{{ site.baseurl }}/assets/img/ir?t=codingwithsam-20&amp;l=am2&amp;o=1&amp;a=0132350882" alt="" width="1" height="1" border="0" />. Object-Orientated code attempts this, but sometimes falls short. In this series, I'll aim to show a basic type that functional programming has that leads to interfaces/APIs that are easier to understand.

## Prerequisites

I'll assume that you know C# or similar Object Orientated language, as well as two other concepts, pattern matching (this is coming to C# 7) and discriminated union (DU) types. See <a href="https://fsharpforfunandprofit.com/posts/match-expression/">Pattern Matching</a> and <a href="https://fsharpforfunandprofit.com/posts/discriminated-unions/">Discriminated Union</a> for more details.
We need a problem to solve, lets assume you need to write the a method/function to divide two numbers
```csharp    
public interface Math
{
    int Divide(int top, int bottom);
}
```
Ok, so we've got an interface, and we can easily write the implementation (or skip the interface and write it as a static class). The problem comes when we do the following:
```csharp
  int result = Math.Divide(1, 0);
```
The are a few problems with this:
- Throws an exception at runtime
- The interface does not suggest that it throws an exception


## An OOP approach
- surround with a try catch to and handle the case (after observing the failure at runtime)
- write a unit test to assert the error case (divide by 0) is handled correctly.
- In the extreme case, as advised in <a href="https://www.amazon.com/gp/product/0132350882/ref=as_li_tl?ie=UTF8&amp;camp=1789&amp;creative=9325&amp;creativeASIN=0132350882&amp;linkCode=as2&amp;tag=codingwithsam-20&amp;linkId=4a9afb03ee2cebd83bec7e5f2b1abdc2" target="_blank" rel="noopener">Clean Code</a><img style="border: none !important; margin: 0px !important;" src="{{ site.baseurl }}/assets/img/ir?t=codingwithsam-20&amp;l=am2&amp;o=1&amp;a=0132350882" alt="" width="1" height="1" border="0" />, a unit test could be written against the divide interface and then referred back to later.

However, I see a few limitations with this:
- The exception must be observed to code for it. For this contrived example, it is possible to reason that this fail, but that can not be known with 100% certainty prior to executing the code (without looking at the implementation). In OOP for error handling, either an exception can be thrown or -1 could be returned (rare cases use an out variable), either case is unclear.

- Developers rarely read unit tests to understand what the code is doing. Unit tests are written to prevent regressions in the software.
- A unit test does not protect against new usages of the api. The developer is required to remember that it throws an exception.


## A problem with no solution; hardly
The solution is to reduce how much the API promises by using types. By making the API more explicit, it will be clear to developers that errors can occur, and to handle them. So what type should this be?

## An example
It's not idiomatic to model this in an OOP language, however, in a functional language this is very straight forward. There are many different ways to encode the error into the API, I'm going with the simplest. In F# it would be as follows:
```fsharp
let divide (top: int) (bottom: int) : int option 
```
The divide function returns an option of int. An option type is a choice between nothing, represented by ```None```, or something plus the value ie ```Some `a```. The ``` `a ``` syntax is the same as generics in C# e.g. ```<T>;```. To make this clearer lets look at the implementation.
Option uses a type that C# does not have called a discriminated union (DU). A DU is able to represent a choice between many different cases. For option we only have two cases ```None``` and ```Some `a``` so it is represented as the following:
```fsharp
type Option = 
  | Some of 'a
  | None
```

I'll show how to get the value out of success in a moment. Before that, lets first look at a contrived implementation and usage of divide. ```Some``` should be returned with the value if it succeeded, or ```None``` should be returned if it failed. With the usage of pattern matching (a functional construct that is basically a better switch statement) would look as follows:
```fsharp
let divide (top: int) (bottom: int) : int option = 
  match bottom with 
  | 0 -> None
  | _ -> top / bottom

let result = divide 1 0 
// result: None

let result = divide 10 5 
// result: Success 2
```

Two example are shown, one with the failure case of ```None``` and one for the success case ```Success 2```. The value of 2 is stuck inside the success type. Using pattern matching again, the value can be extracted and then used as an integer. ie:
```fsharp
let businessCode x y =  
    match divide x y with 
    | None -> // handle error
    | Some x -> // x is the result of the division
```

## How is this better?
- the types of the function (return type of int option) shows the that failure can occur
- The developer does not need to execute the code to see if it can fail
- The developer must write an implementation for the failure case when using pattern matching (or similar approach)


## Going further
Stay tuned for part 2 where I wire this up into an example. In subsequent posts I'll also show how using the FP version you can abstract away the explicit pattern match in the business code. I'll also show you can improve the error handling to provide even more information.
