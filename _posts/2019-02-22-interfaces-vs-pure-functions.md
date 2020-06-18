---
layout: post
title: Interfaces vs Pure Functions
date: 2019-02-22 20:27:44.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Functional-Programming]
tags: [F#, Functional-Programming, Interfaces, PureFunctions]
meta:
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591356429'
  _yoast_wpseo_primary_category: '28'
  _yoast_wpseo_content_score: '60'
  mashsb_jsonshares: '{"facebook_total":1,"facebook_likes":1,"facebook_comments":0}'
  mashsb_shares: '1'

# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
description: Coding to an interface is not good advise. Instead, code
  to a pure function. Your code will be self documenting and you;ll be more productive.
permalink: "/2019/02/22/interfaces-vs-pure-functions/"
---

## My argument laid out

Coding to an interface is not very useful, instead, we should code to a pure function. By doing this, your code will be easier to read/maintain. It will be <g class="gr_ gr_115 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling multiReplace" id="115" data-gr-id="115">self documenting</g>. Productivity will improve, as you don't need to read so much code. Interfaces should be reserved for external <g class="gr_ gr_358 gr-alert gr_spell gr_inline_cards gr_disable_anim_appear ContextualSpelling ins-del multiReplace" id="358" data-gr-id="358">depencies</g>. 

For a function to be <g class="gr_ gr_5 gr-alert gr_spell gr_inline_cards gr_run_anim ContextualSpelling ins-del" id="5" data-gr-id="5">pure</g> it must honour some promises and those promises give us confidence that the business logic we write, will execute correctly. By using interfaces (a function could also be used) for external dependencies we can use mocks, to test our logic (we're not try to <g class="gr_ gr_546 gr-alert gr_gramm gr_inline_cards gr_run_anim Style multiReplace" id="546" data-gr-id="546">test  the</g> external dependency).  Business logic should be written with pure functions.`

## A quick recap of interfaces

I expect that you already know what an interface is, so this will be brief.

An interface is meant to be a contract of sorts. 'Code to an interface' is what we're all taught. We are sold on the idea that all we need is an interface to make our code work.

In F#, the language enforces this. Once you define an instance/object to be of a given interface, you can't access any other methods on it - the way it should be. In contrast, C#/Java do not make assertions. Given an instance that implements some interface, the languages allow the developer to call either methods on the class or methods on the interface.  Here a simple code snippet to demonstrate how this works in F#:
<table class="wp-block-table pre">
<tbody>
<tr>
<td>
<pre class="fssnip"><span class="l"> 1: </span>
<span class="l"> 2: </span>
<span class="l"> 3: </span>
<span class="l"> 4: </span>
<span class="l"> 5: </span>
<span class="l"> 6: </span>
<span class="l"> 7: </span>
<span class="l"> 8: </span>
<span class="l"> 9: </span>
<span class="l">10: </span>
<span class="l">11: </span>
<span class="l">12: </span>
<span class="l">13: </span>
<span class="l">14: </span>
<span class="l">15: </span>
</pre>
</td>
<td>
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="if">IFoo</span> <span class="o">=</span> 
    <span class="k">abstract</span> <span class="k">member</span> <span onmouseout="hideTip(event, 'fs2', 2)" onmouseover="showTip(event, 'fs2', 2)" class="fn">Foo</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs3', 3)" onmouseover="showTip(event, 'fs3', 3)" class="vt">int</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 4)" onmouseover="showTip(event, 'fs3', 4)" class="vt">int</span>

<span class="k">type</span> <span onmouseout="hideTip(event, 'fs4', 5)" onmouseover="showTip(event, 'fs4', 5)" class="rt">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span onmouseout="hideTip(event, 'fs5', 6)" onmouseover="showTip(event, 'fs5', 6)" class="id">this</span><span class="pn">.</span><span class="fn">Bar</span> <span onmouseout="hideTip(event, 'fs6', 7)" onmouseover="showTip(event, 'fs6', 7)" class="id">a</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs6', 8)" onmouseover="showTip(event, 'fs6', 8)" class="id">a</span>
    <span class="k">interface</span> <span onmouseout="hideTip(event, 'fs1', 9)" onmouseover="showTip(event, 'fs1', 9)" class="if">IFoo</span> <span class="k">with</span> 
        <span class="k">member</span> <span onmouseout="hideTip(event, 'fs5', 10)" onmouseover="showTip(event, 'fs5', 10)" class="id">this</span><span class="pn">.</span><span class="fn">Foo</span> <span onmouseout="hideTip(event, 'fs7', 11)" onmouseover="showTip(event, 'fs7', 11)" class="id">i</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs7', 12)" onmouseover="showTip(event, 'fs7', 12)" class="id">i</span>    

<span class="k">let</span> <span onmouseout="hideTip(event, 'fs8', 13)" onmouseover="showTip(event, 'fs8', 13)" class="id">x</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs4', 14)" onmouseover="showTip(event, 'fs4', 14)" class="rt">Foo</span><span class="pn">(</span><span class="pn">)</span>
<span class="k">let</span> <span onmouseout="hideTip(event, 'fs9', 15)" onmouseover="showTip(event, 'fs9', 15)" class="id">y</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs4', 16)" onmouseover="showTip(event, 'fs4', 16)" class="rt">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">:&gt;</span> <span onmouseout="hideTip(event, 'fs1', 17)" onmouseover="showTip(event, 'fs1', 17)" class="if">IFoo</span>
<span onmouseout="hideTip(event, 'fs8', 18)" onmouseover="showTip(event, 'fs8', 18)" class="id">x</span><span class="pn">.</span><span class="id">Foo</span> <span class="n">42</span> <span class="c">// won't compile</span>
<span onmouseout="hideTip(event, 'fs9', 19)" onmouseover="showTip(event, 'fs9', 19)" class="id">y</span><span class="pn">.</span><span class="id">Bar</span> <span class="n">42</span> <span class="c">// Won't compile</span>

<span onmouseout="hideTip(event, 'fs8', 20)" onmouseover="showTip(event, 'fs8', 20)" class="fn">x</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs10', 21)" onmouseover="showTip(event, 'fs10', 21)" class="id">Bar</span> <span class="n">42</span> <span class="c">// happy path</span>
<span onmouseout="hideTip(event, 'fs9', 22)" onmouseover="showTip(event, 'fs9', 22)" class="fn">y</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs11', 23)" onmouseover="showTip(event, 'fs11', 23)" class="id">Foo</span> <span class="n">42</span> <span class="c">// happy path</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

The two images below show the intellisense results:
<figure class="wp-block-image"><img src="{{ site.baseurl }}/assets/img/Intellisense-inferfaceVsPF-class.png" alt="Intellisense dialog showing that the interface methods are not available on the class" /></figure>
<figure class="wp-block-image"><img src="{{ site.baseurl }}/assets/img/Intellisense-inferfaceVsPF-interface.png" alt="Intellisense dialog showing that the class methods are not available on the interface" /></figure>

F#, a functional first programming language, does OOP better than C#/Java!`

## What is an interface missing?

Let's assume that we know how to code to an interface. The challenge is that it does not tell us enough to actually code against it. Consider the following:
<table class="wp-block-table pre">
<tbody>
<tr>
<td>
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td>
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span onmouseout="hideTip(event, 'fs12', 24)" onmouseover="showTip(event, 'fs12', 24)" class="if">IOperation</span> <span class="o">=</span> 
    <span class="k">abstract</span> <span class="k">member</span> <span onmouseout="hideTip(event, 'fs13', 25)" onmouseover="showTip(event, 'fs13', 25)" class="fn">Run</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs3', 26)" onmouseover="showTip(event, 'fs3', 26)" class="vt">int</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 27)" onmouseover="showTip(event, 'fs3', 27)" class="vt">int</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 28)" onmouseover="showTip(event, 'fs3', 28)" class="vt">int</span> 
</code></pre>
</td>
</tr>
</tbody>
</table>

The above defines an interface <code>IOperation</code> that has one method Run. The Run method takes in two integers and returns an integer. Clearly, the naming is terrible here, and that is the point. Naming is always hard, and once published it is even harder to change. Instead of spending all our time thinking of the 'right' name (naming is subjective), what can we do instead is make the code easier to understand (good naming should still be followed).

Considering the <code>IOperation</code> interface, all that we know is that it appears to be some kind of math operation. In reality, though, we really have no idea. If the interface is implemented in our code-base we could go have a look at what it does, which somewhat defeats the purpose of coding to an interface. Additionally there is no proof that the method uses the inputs. There is no proof that the return integer is useful as well. Does it throw exceptions?

There an alternative!`

## What is a pure function

A 'pure function' is the cornerstone of functional programming. The concept is rather simple, given the same inputs, the same output must be returned. For a 'total pure function', the function guarantees that for every range of possible inputs there will be an output of the same type. Total pure functions are the smallest set. Pure functions include total functions and additional functions. Finally, we have just functions (a function attached to a class is a method) which can do anything.

Very few languages can type check that a function is a total pure function. Some languages such as Haskell and PureScript can type check that a function is pure. While the rest of the languages (including F#) require the developer to honour that the function is pure.

Here is an example of a total pure function
<table class="wp-block-table pre">
<tbody>
<tr>
<td>
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td>
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">foo</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs3', 29)" onmouseover="showTip(event, 'fs3', 29)" class="id">int</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 30)" onmouseover="showTip(event, 'fs3', 30)" class="id">int</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 31)" onmouseover="showTip(event, 'fs3', 31)" class="id">int</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

Consider this function, which is similar to our interface. Given we know it is a total pure function, we can make some intelligent deductions about the function.`

## Eliminating possibilities

What it can't do:

- can't talk to the web (that might fail)
- can't talk to a DB (that might fail)
- can't be random - the answer must be the same
- can't throw exceptions
- if the function is total then every int is in the valid range


This has eliminated many possibilities. It suggests that the function must be math based on some form. It could be +, - or * (a poor implementation could ignore one of the inputs, eg the identity function and still be a total pure function. I am assuming that as developers we're attempting to write quality code.) If we know it to be a total pure function then it can't be divide as that would fail with x/0. Coding to this function is going to be a lot easier than the interface we had before, and we don't even have a name. The code we write for this function will execute without exceptions.

Total pure functions provide the highest level of information to code against. Pure functions are the next best, but in some cases, the input may not be valid. For example, if we know that <code>foo</code> was only a pure function (not total), then <code>divide</code> could be a valid implementation. Many cases are valid, but one case is not, x/0. It's possible to convert to convert some pure functions into total pure functions.

Pure functions are what gives FP its reputation 'if it compiles it works'.`

## Not all errors are the same

Now just blindly coding to this function will not produce a correct program with regards to the acceptance criteria (or the end user). If the function should be have been add, but we used subtract then our application is wrong. Regardless of whether we use an interface or pure function, both applications will suffer from this error (and I don't believe it to be a coding issue either). A tester, TDD and BDD automated testing will help with these errors.

Pure functions help first and foremost by eliminating exceptions from our business logic. Pure functions also reduce the size of possible functions (self-documenting code through types), giving a greater chance of choosing the right pure function. It is these advantages that make using a pure function much more compelling over an interface.`

## Pure function provide more information than an interface

Coding to a [total] pure function gives us confidence that our software won't throw exceptions in the pure functions.

In this post, I intentionally left off meaningful names. When we put meaningful names back in, the risk of choosing the wrong pure function goes down considerably. The challenge is simply selecting the most meaningful name from a set of functions that fit the type signature.

Naming the function is also much easier as there are less subtle differences between each of the functions. Naming is now a process of following domain driven design (DDD), so each of the names will be well-defined. Types can also be used to control the input, and these too will part of bounded-context under DDD.

Considering our example above, if we must code to that pure function, then the set of functions that we could provide are limited to some of the following [add, subtract, minus, pow].

Even if these functions (<code>foo:int -&gt; int -&gt; int</code>) were not math functions, but business logic functions, the name plus the knowledge that they are pure functions allows us to make intelligent assumptions around what they will do. The same can not be said for interfaces. As already stated, with interfaces we do not know anything about the relationship of inputs to outputs. We don't know if it will fail either (FP has types that represent the failure case in the type system).`

## Creating <strike>documentation</strike> types

Meaningful names can also be extended to the types as well. In most FP first languages, creating types (classes) is very easy and only a few lines of code (not an entirely new file).

Creating additional types to describe our domain (business logic) can convert pure functions to total functions. This improves the chances of our application being correct, as it removes possible invalid function. It also helps document our code with types (less documentation required).

Covering this in more detail is the subject of another blog post.`

## Productivity boost - fixing errors

By coding to a pure function, rather than an interface, errors are faster to fix (an error is when the application does the wrong thing, i.e. not meeting the acceptance criteria). Because the interface makes very little guarantees, the developer will most likely need to debug the code in the interface. This will includes exceptions and coding errors as well as not meeting the acceptance criteria.

The pure function, however, guarantees that it will not throw exceptions, so needs to only look at his/her own code. Typically either the inputs to function were wrong, or the wrong function was used.

This reduces the search time to find the bug, improving developer productivity.`

## Pure functions in the real world

To prove a function is a total pure function, property-based testing (PBT) must be used. PBT is the process of using randomness and properties (characteristics of a function that describe the relationship of inputs to outputs) to test functions. The randomness attempts to verify that the function is valid under all possible inputs. PBT is beyond the scope of this function (leave a comment if you want more information).

Haskell is the most well known (and used) language that provides compile checking for pure functions. For this reason, Haskell makes for a great learning language to prove that the function is pure. (You could consider using Haskell in production if you're writing a backend).

For many other languages, F# included, practice and care will need to be taken when writing functions. After sufficient practice, it is easy to create applications with pure functions, and the need for the type level checking (eg learning with Haskell) is not required so much.

Write your business logic using pure functions. Model external dependencies using interfaces. Your program will be well structured according to the SOLID principles.`

## Taking action

Consider writing some Haskell in your spare time. Write the business/core logic of the application in a function. Be sure to write out the type signature and leave out IO from the type signature. This will teach you how to identify a pure function and feedback (from compiler) will be given if you are wrong.

Change to an FP first language that is suitable for your domain - F# (my personal favourite), Kotlin, TypeScript or Rust should cover most cases. These languages prefer a functional style so will make it easier to code with pure functions. Languages such as C#, Java, C++ can use methods are pure functions (LINQ in C#, streaming API in Java 8) but the code will be very verbose and not very idiomatic.

Educate your co-workers and managers on the benefits of pure functions. Code reviews are a great opportunity for this.

Happy [type] safe coding

CodingWithSam