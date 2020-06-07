---
layout: post
title: Predictable Code
date: 2016-12-14 18:49:43.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Functional-Programming]
tags: [F#, Functional-Programming]
meta:
  _edit_last: '1'
  _mytory_markdown_etag: '"bc1b64be7ce28e289b8daae3a5f3d6737df8be9c"'
  mytory_md_path: https://raw.githubusercontent.com/willsam100/CodingWithSam/master/Predictable%20Code.md
  mytory_md_text: ''
  mytory_md_mode: url
  mashsb_timestamp: '1590619725'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/predictable-code/
  dsq_thread_id: '6141505729'
  _yoast_wpseo_content_score: '30'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: ''
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2016/12/14/predictable-code/"
---
Code, code, code it's everywhere! As developers we have to read, understand and maintain it. Code that is predictable is code that can be understood just by reading it. On the contrast unpredictable code is code that is either ambiguous or misleading; in any case it will need to be executed somehow to see what it does. This post will show you how to write predictable code.

## prerequisites
- Assumed knowledge of an OOP language like C# or similar
- A simple understanding of a functional programming language. (Code examples are in F#)

## Problem: mutating state
An interface with some math operations for a list. I've modeled with an interface since the premise of OOP is about abstractions and encapsulation.
<table class="pre">
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
<span class="l">5: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">public</span> <span class="k">interface</span> IMath
{
    List&lt;<span class="k">int</span>&gt; Double(List&lt;<span class="k">int</span>&gt; input);
    List&lt;<span class="k">int</span>&gt; Square(List&lt;<span class="k">int</span>&gt; input);
}
</code></pre>
</td>
</tr>
</table>
First cut of the tests (for using the Math implementation of IMath)
<table class="pre">
<tr>
<td class="lines">
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">public</span> Main() 
{
    IMath math <span class="o">=</span> <span class="k">new</span> Math();
    <span class="k">var</span> inputDoubled <span class="o">=</span> math.Double(<span class="k">new</span> List&lt;<span class="k">int</span>&gt; {<span class="n">1</span>,<span class="n">2</span>});
    <span class="k">var</span> inputSquared <span class="o">=</span> math.Square(<span class="k">new</span> List&lt;<span class="k">int</span>&gt; {<span class="n">1</span>,<span class="n">2</span>});

    Assert.Equal(<span class="n">2</span>, inputMultiplied[<span class="n">0</span>]);
    Assert.Equal(<span class="n">4</span>, inputMultiplied[<span class="n">1</span>]);

    Assert.Equal(<span class="n">1</span>, inputSquared[<span class="n">0</span>]);
    Assert.Equal(<span class="n">4</span>, inputSquared[<span class="n">1</span>]);
}
</code></pre>
</td>
</tr>
</table>
After code refactor
<table class="pre">
<tr>
<td class="lines">
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">public</span> <span class="k">void</span> Main()
{
    IMath math <span class="o">=</span> <span class="k">new</span> Math();
    <span class="k">var</span> input <span class="o">=</span> <span class="k">new</span> List&lt;<span class="k">int</span>&gt; {<span class="n">1</span>,<span class="n">2</span>};
    <span class="k">var</span> inputDoubled <span class="o">=</span> math.Double(input);
    <span class="k">var</span> inputSquared <span class="o">=</span> math.Square(input);

    Assert.Equal(<span class="n">2</span>, inputMultiplied[<span class="n">0</span>]);
    Assert.Equal(<span class="n">4</span>, inputMultiplied[<span class="n">1</span>]);

    <span class="c">// Fails</span>
    Assert.Equal(<span class="n">1</span>, inputSquared[<span class="n">0</span>]);
    Assert.Equal(<span class="n">4</span>, inputSquared[<span class="n">1</span>]);
}
</code></pre>
</td>
</tr>
</table>
Clearly this is some unpredictable code; the refactor should have worked. At first glance it appears that the interface for IMath is the problem. It will return you a new list. Unfortunately, after the refactor, it is clear that it modifies the list. The problem is the List class. It is mutable in every way. The size can be changed, the elements can be changed. It's open for everyone.

## Mutability leads to unpredictable code

- Mutation/Mutability (definition): when a variable or object inside an instance can be changed/replaced
- Mutation makes it hard to model time
- Mutation makes it hard to keep constraints with polymorphism (see wikipedia link below)
- Liskov substitution principle is followed by using immutable classes and is even stated on wikipeida

<blockquote>
Mutability is a key issue here. If Square and Rectangle had only getter methods (i.e. they were immutable objects), then no violation of LSP could occur.

<a href="https://en.wikipedia.org/wiki/Liskov_substitution_principle">Liskov substitution principle</a>
</blockquote>

## What to do then

- Avoid mutable classes. Copy the data and change the values during the copy, returning a new instance.
- To make a class immutable in C#: GetHashCode and Equals must be overriden
- Use a Functional language (F#), you get this all for free while still being on .NET


## How does F# do this
Functional programming is about using functions + data to model the domain/behaviour. Encapsulation and abstractions are not goal. Common patterns are still factored out but these a for a another post. Our problem above written in F# looks like this (with the implementation):
<table class="pre">
<tr>
<td class="lines">
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
<span class="l">16: </span>
<span class="l">17: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">module</span> <span class="id">Math</span>

<span class="c">// The type of the function. It takes a list of ints and returns a list of ints</span>
<span class="k">let</span> <span class="id">doubleList</span> <span class="pn">(</span><span class="id">input</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="id">int</span> <span onmouseout="hideTip(event, 'fs2', 2)" onmouseover="showTip(event, 'fs2', 2)" class="id">list</span><span class="pn">)</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs1', 3)" onmouseover="showTip(event, 'fs1', 3)" class="id">int</span> <span onmouseout="hideTip(event, 'fs3', 4)" onmouseover="showTip(event, 'fs3', 4)" class="id">List</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs3', 5)" onmouseover="showTip(event, 'fs3', 5)" class="id">List</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs4', 6)" onmouseover="showTip(event, 'fs4', 6)" class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span> <span class="pn">*</span> <span class="n">2</span><span class="pn">)</span> <span class="id">input</span> 
<span class="c">// The type of the function. It takes a list of ints and returns a list of ints </span>
<span class="k">let</span> <span class="id">squareList</span> <span class="pn">(</span><span class="id">input</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs1', 7)" onmouseover="showTip(event, 'fs1', 7)" class="id">int</span> <span onmouseout="hideTip(event, 'fs2', 8)" onmouseover="showTip(event, 'fs2', 8)" class="id">list</span><span class="pn">)</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs1', 9)" onmouseover="showTip(event, 'fs1', 9)" class="id">int</span> <span onmouseout="hideTip(event, 'fs3', 10)" onmouseover="showTip(event, 'fs3', 10)" class="id">List</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs3', 11)" onmouseover="showTip(event, 'fs3', 11)" class="id">List</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs4', 12)" onmouseover="showTip(event, 'fs4', 12)" class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span> <span class="pn">*</span> <span class="id">x</span><span class="pn">)</span> <span class="id">input</span> 

<span class="k">let</span> <span class="id">main</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">input</span> <span class="o">=</span> <span class="pn">[</span><span class="n">1</span><span class="pn">;</span><span class="n">2</span><span class="pn">]</span>
    <span class="k">let</span> <span class="id">inputDoubled</span> <span class="o">=</span> <span class="id">doubleList</span> <span class="id">input</span>
    <span class="k">let</span> <span class="id">inputSquared</span> <span class="o">=</span> <span class="id">squareList</span> <span class="id">input</span> 

    <span class="id">Assert</span><span class="pn">.</span><span class="id">Equal</span><span class="pn">(</span><span class="n">2</span><span class="pn">,</span> <span class="id">inputDoubled</span><span class="pn">.</span><span class="pn">[</span><span class="n">0</span><span class="pn">]</span><span class="pn">)</span>
    <span class="id">Assert</span><span class="pn">.</span><span class="id">Equal</span><span class="pn">(</span><span class="n">4</span><span class="pn">,</span> <span class="id">inputDoubled</span><span class="pn">.</span><span class="pn">[</span><span class="n">1</span><span class="pn">]</span><span class="pn">)</span>

    <span class="id">Assert</span><span class="pn">.</span><span class="id">Equal</span><span class="pn">(</span><span class="n">1</span><span class="pn">,</span> <span class="id">inputSquared</span><span class="pn">.</span><span class="pn">[</span><span class="n">0</span><span class="pn">]</span><span class="pn">)</span>
    <span class="id">Assert</span><span class="pn">.</span><span class="id">Equal</span><span class="pn">(</span><span class="n">4</span><span class="pn">,</span> <span class="id">inputSquared</span><span class="pn">.</span><span class="pn">[</span><span class="n">1</span><span class="pn">]</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</table>

## Why does this work

- The list type in F# (which is different to C#'s list) is immutable so the result is copied and changed.
- It's harder to create mutable objects:
- F# creates constants by default (these are called bindings, and means the reference can't be changed. If the object is not immutable then elementobject can be change eg. an array)
- A variable in F# requires the ```mutable``` keyword (It's more work, and some IDEs highlight the keyword to be explicit)


## What's the point

- immutability makes the code predictable. It can understood without executing it.
Mutability and unpredictable code requires you execute the code somehow to check it does what you think.

**Use F# if you don't like writing boilerplate code and/or want your code to just work** Use F#


## Immutability FTW
NB: I claimed that FP does not have abstractions as goal. FP makes great leaps to reduce boilerplate code and encourage code reuse, but takes a mathematical approach to do this. Higher order functions and Monads (eg. State Monad) are examples of these.