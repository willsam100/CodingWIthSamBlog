---
layout: post
title: C# - The good parts
date: 2019-06-19 07:24:20.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [State]
tags: [C#, Composition, Functional-Programming, Option-Type, PureFunctions, Subclassing]
meta:
  _wpcom_is_markdown: '1'
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1590991191'
  _yoast_wpseo_primary_category: '47'
  _yoast_wpseo_content_score: '30'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2019/06/19/c-the-good-parts/"
---
Every language has some good parts and some bad parts. Some more than others. Additionally, each language changes slightly over time as developers find better ways of solving problems. Identifying the good parts and the good trends is in a language is a great strategy for success.

Focusing on C#, there are a few bad parts to the language that you may be using that are not helping you. This post will show you some alternatives that will improve your code in the context of a business application. For business software, the goals are generally the correct answer according to the spec, written in a way that other developers can read it, along with some tests (to prove you met the spec). Performance is generally not a goal, and so this post does not focus on that. In some cases your code will be faster, most likely the changes will make the code slower (but only in the order of milliseconds).

## An easy one - avoid sub-classing / inheritance
I call this an easy one because my university actually taught not to use sub-classing, yet I find it still being used to a lot. To be fair, I have read somewhere that C# is better than Java at this :). The problem with sub-classing is that it creates a coupling between classes that is very hard to break once it is introduced. The classic example of this is the fragile base class problem. Changes to the base class ripple throughout the inheritance tree causing unintended changes (ie classes that are in fact tightly coupled). Another challenge of sub-classing is that it is typically hard to get the structure right. Choosing a structure for the base class is required at the beginning of a solution, when little about the domain is known, resulting in a poor domain model.

There are several alternatives available. The most common is composition-over-inheritance. That is, put the logic required in another class, and then new-up an instance of that class in the target class. As developers, this is the 'helper' or 'util' classes that we love to create.

Another alternative that is not always stated is to just duplicate the code. If it's not very complicated, and only duplicated 3 times, then everything is fine.
The solution that I tend to follow is a variation of the composition; static methods without state. Given <code>A</code> as a base class and <code>B</code> as the child, I would move the logic to a static class, <code>AStatic</code>, with static methods. <code>Astatic</code> has not fields, ideally the inputs can't be changed (immutable). Each static method takes in all it needs and returns the result (fanatics call this a pure function). <code>B</code> will simply call <code>AStatic</code> to get the job the done.
Is this anti-OOP?

It has been said that static methods are an anti-pattern in OOP languages like Java and C# (I've heard rumours that people were let go over this stuff- I'm not sure if that true). I consider bugs to be an anti-pattern. If an idea is able to help me then it should be used, ideas that are not helping should be let go. Static classes and static methods, without any state, are very easy to understand and test. Create a set of tests, pass in the data, and then check the result matches the expected output. No mocking, no stubbing, no complex test setups. .

## Avoid Loop constructs
Loops can not be avoided the but the language constructs can be. Avoid the following loop at all costs:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">for</span> <span class="pn">(</span><span class="id">var</span> <span class="id">i</span> <span class="o">=</span> <span class="n">0</span><span class="pn">;</span> <span class="id">i</span> <span class="pn">&lt;</span> <span class="id">items</span><span class="pn">.</span><span class="id">Count</span><span class="pn">;</span> <span class="o">++</span><span class="id">i</span><span class="pn">)</span> 
<span class="pn">{</span>
    <span class="c">// bugs in here ie i++</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The above code may work the first time, but it is very hard to read given the number of variations following the <code>for</code> keyword (incrementing, decrementing, starting from 1, etc). It's also easy to make changes to the index, <code>i</code> which makes for very confusing code.

A slightly better solution is the <code>foreach</code> loop. The collection is always read in the same direction (start to end), a <code>for</code> loop may not always process the list in the same direction ie decrementing <code>i</code>. The collection can't be changed creating a little more certainty about the output. Also every item in the list is visited, a helpful fact that can't be skipped on occasion. Things are looking much better with this. Sadly, there are still some downsides. Given that we can't change the collection, every item must be visited, a bummer if you have a very large collection, but only need a couple of items. It's also not possible to return anything. So variables must be used to build a new collection, introducing important information outside of the loop. Here is an example
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">foreach</span> <span class="pn">(</span><span class="id">var</span> <span class="id">item</span> <span class="k">in</span> <span class="id">items</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="id">itemsPrime</span><span class="pn">.</span><span class="id">Add</span><span class="pn">(</span><span class="id">item</span> <span class="o">+</span> <span class="n">1</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The loop is very nice, but the body is not clear. where did <code>itemsPrime</code> come from? Is it part of the method, or a field on the class? It is impossible to know with just these three lines of code.

## LINQ to solve all looping problems
To address these problems, consider LINQ. It addresses all of the problems above. In some cases, it can even make your code faster. Here is the above snippet using LINQ.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">var</span> <span class="id">result</span> <span class="o">=</span> <span class="id">items</span><span class="pn">.</span><span class="id">Select</span><span class="pn">(</span><span class="id">x</span> <span class="o">=&gt;</span> <span class="id">x</span> <span class="o">+</span> <span class="n">1</span><span class="pn">)</span><span class="pn">;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As you can see, all the information we need is right here. No intermediate variables, <code>itemsPrime</code>, needed. There is no looping construct either (though the computer clearly still loops over the collection). Just with the <code>foreach</code> loop we also know that every item will be visited. If we don't want to visit every item we can use LINQ to address that too:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">var</span> <span class="id">result</span> <span class="o">=</span> <span class="id">items</span><span class="pn">.</span><span class="id">Where</span><span class="pn">(</span><span class="id">x</span> <span class="o">=&gt;</span> <span class="id">x</span> <span class="o">%</span> <span class="n">0</span> <span class="o">==</span> <span class="n">0</span><span class="pn">)</span><span class="pn">.</span><span class="id">Select</span><span class="pn">(</span><span class="id">x</span> <span class="o">=&gt;</span> <span class="id">x</span> <span class="o">+</span> <span class="n">1</span><span class="pn">)</span><span class="pn">;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The code above takes only the even numbers and then add 1 to them. The compiler is very smart as well and will optimise this code a bit (there won't be two for loops). Finally, it is also possible to generate the data, filter it, and then transform it.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">var</span> <span class="id">nonLeapYears</span> <span class="o">=</span> 
    <span class="id">Enumerable</span><span class="pn">.</span><span class="id">Range</span><span class="pn">(</span><span class="n">1970</span><span class="pn">,</span> <span class="n">2019</span> <span class="o">-</span> <span class="n">1970</span> <span class="o">+</span> <span class="n">1</span><span class="pn">)</span> <span class="c">// Better than a for loop</span>
    <span class="pn">.</span><span class="id">Where</span><span class="pn">(</span><span class="id">x</span> <span class="o">=&gt;</span> <span class="id">x</span> <span class="o">%</span> <span class="n">100</span> <span class="o">==</span> <span class="n">0</span> <span class="o">&amp;&amp;</span> <span class="id">x</span> <span class="o">%</span> <span class="n">4</span> <span class="o">!=</span> <span class="n">0</span> <span class="o">&amp;&amp;</span> <span class="id">x</span> <span class="o">%</span> <span class="n">400</span> <span class="o">!=</span> <span class="n">0</span> <span class="pn">)</span> <span class="c">// filter out leap years</span>
    <span class="pn">.</span><span class="id">Select</span> <span class="pn">(</span><span class="id">x</span> <span class="o">=&gt;</span> <span class="k">new</span> <span class="id">DateTime</span><span class="pn">(</span><span class="id">x</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="n">0</span><span class="pn">)</span><span class="pn">)</span><span class="pn">;</span> <span class="c">// convert the DateTime</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
In a few short lines of code, it is possible to use no loops, generate all the years from 1970 to 2019, filter out the leap years, and then turn the years into <code>DateTime</code> objects. Each line contains one idea that builds from the previous. No mixing of logic over several lines.

## Avoid the billion dollar mistake - null
This is another topic that I learned during my time at university, but sadly not taught how to actually do it. The good news is that new features of C# are improving this with C# 8 (non-nullable reference types). I am assuming that you have had many <code>NullRefenceExpection</code>s and I don't need to highlight why it's bad. Instead, covering how to program without null is probably needed.

The easy win is to not use null for a default value. Instead, set it to a sensible default. A test (unit) should catch that the default value does not slip through. If it does though, the application won't crash, the tester/user will just see the wrong value. A wrong value can be much easier to find than crash (more on later).

## Avoid null by using a result variable
Another common pattern is to assign a value from an <code>if ... else</code> block or a <code>try ... catch</code> block. In C# it's not possible this (tip: it is in F#). So the code generally looks as follows:
<table class="pre">
<tbody>
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
<pre class="fssnip highlighted"><code lang="fsharp"><span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="id">string</span> <span class="id">result</span> <span class="o">=</span> <span class="k">null</span><span class="pn">;</span>
<span class="k">if</span> <span class="pn">(</span><span class="id">x</span> <span class="o">==</span> <span class="n">42</span><span class="pn">)</span> 
    <span class="id">result</span> <span class="o">=</span> <span class="s">"42"</span><span class="pn">;</span>
<span class="k">else</span> 
    <span class="id">result</span> <span class="o">=</span> <span class="s">"unlucky"</span><span class="pn">;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This pattern turns up quite often, but the downside is that the compiler can not help to check out work. The following would probably proudce a <code>NullReferenceException</code>:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span onmouseout="hideTip(event, 'fs1', 2)" onmouseover="showTip(event, 'fs1', 2)" class="id">string</span> <span class="id">result</span> <span class="o">=</span> <span class="k">null</span><span class="pn">;</span>
<span class="k">if</span> <span class="pn">(</span><span class="id">x</span> <span class="o">==</span> <span class="n">42</span><span class="pn">)</span> 
    <span class="id">result</span> <span class="o">=</span> <span class="s">"42"</span><span class="pn">;</span>
<span class="c">// Forgot to put in else block, result == null</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Part of this is caused by the use of null (setting a default would help), the other problem is caused by mutation - changing of the value. To leverage the compiler to check our work, use a method with a return value instead.
<table class="pre">
<tbody>
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">private</span> <span onmouseout="hideTip(event, 'fs1', 3)" onmouseover="showTip(event, 'fs1', 3)" class="id">string</span> <span class="id">FindResult</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs2', 4)" onmouseover="showTip(event, 'fs2', 4)" class="id">int</span> <span class="id">x</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="k">if</span> <span class="pn">(</span><span class="id">x</span> <span class="o">==</span> <span class="n">42</span><span class="pn">)</span> 
        <span class="k">return</span> <span class="s">"42"</span><span class="pn">;</span>
    <span class="k">else</span> 
        <span class="k">return</span> <span class="s">"unlucky"</span><span class="pn">;</span>
<span class="pn">}</span>

<span class="k">private</span> <span class="k">void</span> <span class="id">Main</span><span class="pn">(</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="id">var</span> <span class="id">result</span> <span class="o">=</span> <span class="id">FindResult</span><span class="pn">(</span><span class="n">42</span><span class="pn">)</span><span class="pn">;</span>
    <span class="id">var</span> <span class="id">output</span> <span class="o">=</span> <span class="s">"Result: "</span> <span class="o">+</span> <span class="id">result</span><span class="pn">;</span>
    <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="id">output</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
With this approach, if the <code>else</code> block was missing, the compiler would produce an error. Using C# 7 features, it's also possible to shorten this code a little bit by using <code>Local Function</code> ie a method inside of a method. Here is the same example again:
<table class="pre">
<tbody>
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">private</span> <span class="k">void</span> <span class="id">Main</span><span class="pn">(</span><span class="pn">)</span>
<span class="pn">{</span>
    <span onmouseout="hideTip(event, 'fs1', 5)" onmouseover="showTip(event, 'fs1', 5)" class="id">string</span> <span class="id">FindResult</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs2', 6)" onmouseover="showTip(event, 'fs2', 6)" class="id">int</span> <span class="id">x</span><span class="pn">)</span>
    <span class="pn">{</span>
        <span class="k">if</span> <span class="pn">(</span><span class="id">x</span> <span class="o">==</span> <span class="n">42</span><span class="pn">)</span> 
            <span class="k">return</span> <span class="s">"42"</span><span class="pn">;</span>
        <span class="k">else</span> 
            <span class="k">return</span> <span class="s">"unlucky"</span><span class="pn">;</span>
    <span class="pn">}</span>

    <span class="id">var</span> <span class="id">result</span> <span class="o">=</span> <span class="id">FindResult</span><span class="pn">(</span><span class="n">42</span><span class="pn">)</span><span class="pn">;</span>
    <span class="id">var</span> <span class="id">output</span> <span class="o">=</span> <span class="s">"Result: "</span> <span class="o">+</span> <span class="id">result</span><span class="pn">;</span>
    <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="id">output</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The inner method now has an appropriate scope and is close to where it is being used, much more readable. This pattern can also be used for a <code>try .. catch</code> as well when a value is needed.

## Avoid null - use domain modelling
The above pattern works very well, as long as there is a default value that can be used (initially or in the <code>else</code>/<code>catch</code> block). However, consider the case for finding a user from a DB given their ID. The user may not be there, and it would be silly to create a default user. In the past, this is usually done with exceptions or null.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
<span class="l">5: </span>
<span class="l">6: </span>
<span class="l">7: </span>
<span class="l">8: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">private</span> <span class="id">User</span> <span class="id">FindUser</span><span class="pn">(</span><span class="id">Guid</span> <span class="id">userID</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="c">// pseudo code for db lookup</span>
    <span class="k">if</span> <span class="pn">(</span><span class="id">userExists</span><span class="pn">)</span> 
        <span class="k">return</span> <span class="id">user</span><span class="pn">;</span>

    <span class="id">throw</span> <span class="k">new</span> <span class="id">MissingUserException</span><span class="pn">(</span><span class="o">$</span> <span class="s">"User with ID {userID} was not found"</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
or
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
<span class="l">4: </span>
<span class="l">5: </span>
<span class="l">6: </span>
<span class="l">7: </span>
<span class="l">8: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">private</span> <span class="id">User</span> <span class="id">FindUser</span><span class="pn">(</span><span class="id">Guid</span> <span class="id">userID</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="c">// pseudo code for db lookup</span>
    <span class="k">if</span> <span class="pn">(</span><span class="id">userExists</span><span class="pn">)</span> 
        <span class="k">return</span> <span class="id">user</span><span class="pn">;</span>

    <span class="k">return</span> <span class="k">null</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
In either case, the application will most likely crash at runtime if care is not taken to handle a missing user. Additionally, the compiler provides no help to tell the developer to check for a missing user.

The trick here is to leverage the compiler and create a default value. This technique is also in line with domain driven design (DDD). It's possible to not have a user so we should have classes and objects that reflect that in code (null does not mean no user).
<table class="pre">
<tbody>
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
<span class="l">18: </span>
<span class="l">19: </span>
<span class="l">20: </span>
<span class="l">21: </span>
<span class="l">22: </span>
<span class="l">23: </span>
<span class="l">24: </span>
<span class="l">25: </span>
<span class="l">26: </span>
<span class="l">27: </span>
<span class="l">28: </span>
<span class="l">29: </span>
<span class="l">30: </span>
<span class="l">31: </span>
<span class="l">32: </span>
<span class="l">33: </span>
<span class="l">34: </span>
<span class="l">35: </span>
<span class="l">36: </span>
<span class="l">37: </span>
<span class="l">38: </span>
<span class="l">39: </span>
<span class="l">40: </span>
<span class="l">41: </span>
<span class="l">42: </span>
<span class="l">43: </span>
<span class="l">44: </span>
<span class="l">45: </span>
<span class="l">46: </span>
<span class="l">47: </span>
<span class="l">48: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">public</span> <span class="k">class</span> <span class="id">User</span> 
<span class="pn">{</span>
    <span class="k">public</span> <span class="id">Guid</span> <span class="id">ID</span> <span class="pn">{</span><span class="id">get</span><span class="pn">;</span> <span onmouseout="hideTip(event, 'fs3', 7)" onmouseover="showTip(event, 'fs3', 7)" class="id">set</span><span class="pn">;</span><span class="pn">}</span>
    <span class="k">public</span> <span onmouseout="hideTip(event, 'fs4', 8)" onmouseover="showTip(event, 'fs4', 8)" class="id">String</span> <span class="id">Name</span> <span class="pn">{</span><span class="id">get</span><span class="pn">;</span> <span onmouseout="hideTip(event, 'fs3', 9)" onmouseover="showTip(event, 'fs3', 9)" class="id">set</span><span class="pn">;</span><span class="pn">}</span>
    <span class="c">// etc</span>
<span class="pn">}</span>

<span class="k">public</span> <span class="k">class</span> <span class="id">OptionUser</span> 
<span class="pn">{</span>
    <span class="k">private</span> <span class="id">User</span> <span class="id">user</span><span class="pn">;</span>

    <span class="k">public</span> <span class="id">OptionUser</span><span class="pn">(</span><span class="id">User</span> <span class="id">user</span><span class="pn">)</span>
    <span class="pn">{</span>
        <span class="id">this</span><span class="pn">.</span><span class="id">user</span> <span class="o">=</span> <span class="id">user</span><span class="pn">;</span>
    <span class="pn">}</span>

    <span class="k">public</span> <span class="id">OptionUser</span><span class="pn">(</span><span class="pn">)</span> <span class="pn">{</span><span class="pn">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="id">GetUser</span><span class="pn">(</span><span class="id">Action</span><span class="pn">&lt;</span><span class="id">User</span><span class="pn">&gt;</span> <span class="id">hasUser</span><span class="pn">,</span> <span class="id">Action</span> <span class="id">noUser</span><span class="pn">)</span>
    <span class="pn">{</span>
        <span class="k">if</span> <span class="pn">(</span><span class="id">user</span> <span class="o">==</span> <span class="k">null</span><span class="pn">)</span>
            <span class="id">noUser</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>
        <span class="k">else</span> 
            <span class="id">hasUser</span><span class="pn">(</span><span class="id">user</span><span class="pn">)</span><span class="pn">;</span>
    <span class="pn">}</span>

    <span class="c">// Repeat GetUser for Func&lt;User, T&gt;</span>
<span class="pn">}</span>

<span class="k">private</span> <span class="k">static</span> <span class="id">OptionUser</span> <span class="id">FindUser</span><span class="pn">(</span><span class="id">Guid</span> <span class="id">userID</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="c">// pseudo code for db lookup</span>
    <span class="k">if</span> <span class="pn">(</span><span class="id">userExists</span><span class="pn">)</span> 
        <span class="k">return</span> <span class="k">new</span> <span class="id">OptionUser</span><span class="pn">(</span><span class="id">user</span><span class="pn">)</span><span class="pn">;</span>

    <span class="k">return</span> <span class="k">new</span> <span class="id">OptionUser</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>

<span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="id">Main</span><span class="pn">(</span><span class="pn">)</span>
<span class="pn">{</span>
    <span class="id">var</span> <span class="id">userID</span> <span class="o">=</span> <span class="id">Guid</span><span class="pn">.</span><span class="id">NewGuid</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>
    <span class="id">var</span> <span class="id">optionUser</span> <span class="o">=</span> <span class="id">FindUser</span><span class="pn">(</span><span class="id">userID</span><span class="pn">)</span><span class="pn">;</span>
    <span class="id">optionUser</span><span class="pn">.</span><span class="id">GetUser</span><span class="pn">(</span><span class="id">user</span> <span class="o">=&gt;</span> <span class="pn">{</span>
        <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="o">$</span> <span class="s">"Found User: {user.Name}"</span><span class="pn">)</span><span class="pn">;</span>
    <span class="pn">}</span><span class="pn">,</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=&gt;</span> <span class="pn">{</span>
        <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="o">$</span> <span class="s">"Could not find user with ID {userID}, did you use the correct id?"</span><span class="pn">)</span><span class="pn">;</span>
    <span class="pn">}</span><span class="pn">)</span><span class="pn">;</span>
<span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
First, we are defining a class that captures the idea of a missing user - an option that we don't know. To get the user out of the class, a delegate/lambda must be passed in, one for when there is a user, and one for when there is not. No more exceptions, no more null (hang tight on the <code>OptionUser</code> class), and the compiler is now checking our work too:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp">    <span class="id">optionUser</span><span class="pn">.</span><span class="id">GetUser</span><span class="pn">(</span>
        <span class="id">user</span> <span class="o">=&gt;</span> <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="o">$</span> <span class="s">"Found User: {user.Name}"</span><span class="pn">)</span><span class="pn">)</span><span class="pn">;</span>
        <span class="c">// won't compile </span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Things are looking good. We did need to write a lot of code though. Much of that code will always be the same. OptionUser could really be <code>Option&lt;T&gt;</code> since it would apply to any table in the DB. There are a few ways to address this. You could define your own <code>Option&lt;T&gt;</code>, and reuse it, solving the null check only once (much better than all throughout the application). Another alternative is to use a library that has implemented <code>Option&lt;T&gt;</code> already. <a href="" title="https://github.com/nlkl/Optional">Optional</a> is one library another, library is <a href="" title="https://github.com/louthy/language-ext">language-ext</a> which has a lot more functionality. If you're considering <a href="" title="https://github.com/louthy/language-ext">language-ext</a> then I would suggest considering F#. F# provides <code>Option&lt;T&gt;</code> and much of <code>language-ext</code> as part of the language with a cleaner syntax (F# is probably already installed on your machine, unlike the <code>language-ext</code> nuget package).
Stretch goal: If you want to use an option but need to send an error type back then consider the <code>Result&lt;S, E&gt;</code> which is available in F# or <code>language-ext</code>. <code>Result</code> is the next logical extension from <code>Option</code>.

## Avoid encapsulation aka mixing state and behaviour
OOP was sold to us around the concept of encapsulation, connecting state and behaviour. It turns out in practice though, this does not work as well as expected. Just like with inheritance discussed above, encapsulation doesn't compose very well as the software grows. The open/closed principle from SOLID design suggests that once a class is made (using encapsulation for the state of the application), the class should not need to be changed. The software should continue to grow by simply extending the class (either through inheritance or composition).

This promise does not hold up (or is much easier said than done). Testing stateful objects is hard. Finding stateful bugs is even hard. There are alternative ways that are easier to understand, easier to test, and easier to maintain. They my not be suitable for high performance computing.
If an idea is not helpful, even if everyone else is using it, we should abandon it for a better one.

The alternative is to write two kinds of classes. In most cases we should write classes that only have behaviour, let's call them stateless-services. Services should operate on the second kind of class, immutable data structures. State and behaviour have now been separated.

For a web service, it is almost possible to write an entire application like this, a web service is receiving some date via an API, storing it into a DB, or reading the DB and returning the response. All services could be stateless, passing immutable data between them.

For a GUI application (desktop or mobile) it's not so easy, and impossible you're following MVVM. All is not lost though. View models must be stateful, and that is fine. The view can be stateful as well since it might need to do interesting things too. For everything else, it should be possible to either push the state to one service or have no state at all. By having no state at all, it means the state is either in the local app database, or it is in the view models.
why would we actually want to do this?

By doing this you will find a couple of easy wins early on. The best gains are seen over time though when refactoring comes into play. The upfront gains are seen when writing unit tests. The stateless classes are very very easy to test. It requires creating some classes, passing them through the method under test, and then some assertions. If your stateless classes separate dependencies from business logic, then testing is even easier, as you won't need any mocks or stubs for most classes. It's even possible to throw randomness at the static method to prove it works using a tool like <a href="" title="https://fscheck.github.io/FsCheck/">FsCheck</a>. For the classes that do have dependencies, you will only be using stubs and mocks to verify the correct flow of the code.

Another benefit to using stateless services with immutable data is that the compiler can check your work for you. This is the same idea that was presented earlier with the local function returning a value. Using the same approach for a service means that objects must be passed around, and the code simply won't compile if you forget a line (the types won't match for the methods). This approach actually speeds up development as there is less time spent debugging strange exceptions (usually because a method was not called that reset state on some class).
One place where C# does this really well is with LINQ. The whole system is a collection of static/extension methods that operate on immutable data, <code>IEnumerable</code>.

## Wrap up
Avoid subclassing - use composition/static methods with no state instead

avoid for loops - use LINQ instead

prefer stateless services with immutable data structures over encapsulation
Your code will be easier to maintain, faster to write, and you'll have a happier time with less erroneous exceptions.
F# supports this as the preferred style of software development, and is probably installed on your machine already (including your Mac if you have VS4Mac).