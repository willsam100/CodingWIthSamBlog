---
layout: post
title: Are async animations not working? It's easy and fun with F#
date: 2017-07-06 19:03:33.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Animations, F#, Mobile, Threading, XamarinForms]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1591014519'
  mashsb_shares: '2'
  mashsb_jsonshares: '{"total":2,"error":"","facebook_shares":2,"twitter":0,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/gui-animations-asyncawait-to-fs-async/
  _yoast_wpseo_focuskw_text_input: async animations
  _yoast_wpseo_focuskw: async animations
  _yoast_wpseo_title: Are async animations giving you trouble? It's easy with F# |
    %%category%%
  _yoast_wpseo_metadesc: Async code is hard to write, and async/await in C# can be
    hard to grok. Find out how to write async animations for Xamarin Forms using F#
  _yoast_wpseo_linkdex: '71'
  _yoast_wpseo_content_score: '30'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '29'
  dsq_thread_id: '6133611476'
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2017/07/06/gui-animations-asyncawait-to-fs-async/"
---
## Prerequisites
- An understanding of Xamarin
- An simple understanding of Threads
- An understanding of F#

Recently I attended a Winter of Xamarin event hosted by Microsoft. It was a great event, but as always the code examples all in C#. I thought it would be a great a exercise to translate the C# app into F#. This post focuses on one particular section that I came across, some Xamarin Forms' animations, that I found less than trivial to translate. The various attempts that I tried will be outlined, with the final solution at the end.

## The animation code
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp">    <span class="k">public</span> <span class="id">async</span> <span class="id">Task</span> <span class="id">Shake</span><span class="pn">(</span><span class="pn">)</span>
    <span class="pn">{</span>
        <span class="k">for</span> <span class="pn">(</span><span class="id">int</span> <span class="id">i</span> <span class="o">=</span> <span class="n">0</span><span class="pn">;</span> <span class="id">i</span> <span class="pn">&lt;</span> <span class="n">8</span><span class="pn">;</span> <span class="id">i</span><span class="o">++</span><span class="pn">)</span>
        <span class="pn">{</span>
            <span class="id">await</span> <span class="id">Task</span><span class="pn">.</span><span class="id">WhenAll</span><span class="pn">(</span>
                    <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span><span class="pn">,</span>
                    <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="pn">)</span><span class="pn">;</span>

            <span class="id">await</span> <span class="id">Task</span><span class="pn">.</span><span class="id">WhenAll</span><span class="pn">(</span>
                    <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="pn">)</span><span class="pn">;</span>

            <span class="id">await</span> <span class="id">Task</span><span class="pn">.</span><span class="id">WhenAll</span><span class="pn">(</span>
                    <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0</span><span class="pn">,</span> <span class="o">-</span><span class="n">30</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="pn">)</span><span class="pn">;</span>

            <span class="id">await</span> <span class="id">Task</span><span class="pn">.</span><span class="id">WhenAll</span><span class="pn">(</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span><span class="pn">,</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">)</span><span class="pn">;</span>
        <span class="pn">}</span>
    <span class="pn">}</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Here is the main gist of the code. The method is part of a ContentPage. I'm assuming that most readers will understanding this, so will only go over this briefly. <code>Task.WhenAll</code> executes the animations in parallel. The <code>await</code> after each <code>Task.WhenAll</code> means wait for it to complete, so the inside of the loop, contains 4 sequential animations. Finally the loop just runs the 4 animations, sequentially 8 times.

## Async in F#
Understanding Async in F# can be a little tricky. This is not because is it complicated, rather it is more explicit. Tomas Petricek has a great <a href="http://tomasp.net/blog/csharp-fsharp-async-intro.aspx/">'series on Async'</a>, and another post on the <a href="http://tomasp.net/blog/csharp-async-gotchas.aspx/">'Async in C# and F#: gotchas in C#'</a>. Armed with this knowledge here is my first attempt at translating the inner loop.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// Nested inside a ContentPage</span>
<span class="k">let</span> <span class="id">show</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="pn">[</span>
        <span class="pn">[|</span>
            <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
        <span class="pn">|]</span>
        <span class="pn">[|</span> <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> <span class="pn">|]</span>
        <span class="pn">[|</span> <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> <span class="pn">|]</span>
        <span class="pn">[|</span>
            <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
        <span class="pn">|]</span>
    <span class="pn">]</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">iter</span><span class="pn">(</span><span class="k">fun</span> <span class="id">animations</span> <span class="k">-&gt;</span> 
        <span class="id">animations</span>
        <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> 
        <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> 
        <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span> 
        <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Here's some F# code that compiles and was my first attempt. Preferring a functional style, I wanted to treat the animations as data, and then <code>iter</code> (map with no result) over them. The function inside the iter contains the relevant transformations to make the code compile with F#. Let's walk through it. First <code>animations</code> is an array of <code>Task</code> and F# needs <code>Async</code>, so lets <code>map</code> over them and convert them to <code>Async</code>s. Next we we want to run them in parallel, <code>Async.Parallel</code> for the job. This is the same as C#'s <code>Task.WhenAll</code>. On to the next line, F# checks return types, and each of the animations return a bool. <code>Async.Ignore</code> will fix up the return type since there is nothing to check. Finally, Async.StartImmediate starts on the main thread an returns immediately so it won't block the main thread, but also doesn't create a new thread. (Async.RunSynchronously can't be used here since the animations must be run on the main thread and will wait for. The result: a deadlock). I'll save you time of trying the <code>show</code> function out; it doesn't work. When you run it, nothing happens. No crash, no animation.

## But it compiled
When dealing with side effects, all bets are off as to whether the code works; animations are side effects. A careful read on Thomas' blog can give us an understanding of what is happening here. On the C# side, static methods for Tasks start immediately. This means that treating animations as data in a list (array) won't work as they will be started immediately. Armed with this knowledge here is another attempt.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">shake</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>  
    <span class="id">async</span> <span class="pn">{</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This looks a little better and this does in fact work. The animation runs. As before, for each of the arrays, they need to be converted for <code>Task</code>s, set to run in parallel and have their result ignored. This time, instead of putting all the arrays in a list, they are inside an async block, and the <code>do!</code> tells F# to wait for each one to finish before starting the next one. <code>Async.StartImmediate</code> starts this off nicely on the main thread. I'll come back to this later and refactor out some of the duplicated code, but now let's get this running 8 times.

## Running 8 times
Surely it can't be hard to run an animation 8 times. Also were using F# so variables and loops are beneath us. Here is my (failed) first attempt at running the above code 8 times.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">shake</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>  
    <span class="id">async</span> <span class="pn">{</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">replicate</span> <span class="n">8</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">iter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">animation</span> <span class="k">-&gt;</span> <span class="id">async</span> <span class="pn">{</span><span class="k">do!</span> <span class="id">animation</span><span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The last line is all that's changed. An attempt was made to replicate the <code>Async</code> action 8 times and then iter over the resulting list wrapping the animation in <code>do!</code> and executing as <code>Async.StartImmediate</code>. I had assuming that <code>Async.StartImmediate</code> was the same as C# and that they could be chained together. As already stated, this doesn't work. In F#, any async work must be enclosed in an async block, and started only once. <code>Async.StartImmediate</code> is analogous with an a C# method of <code>public async void</code> meaning you can not continue after the task has been started.

## A working version
I still didn't want to put a variable with a for loop in, so I found the best compromise I could handle. Here is the first working solution:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">shakeLandscapeAsync</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">shake</span> <span class="o">=</span>  <span class="id">async</span> <span class="pn">{</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
    <span class="pn">}</span>
    <span class="id">async</span> <span class="pn">{</span> 
        <span class="k">for</span> <span class="id">action</span> <span class="k">in</span> <span class="id">List</span><span class="pn">.</span><span class="id">replicate</span> <span class="n">8</span> <span class="id">shake</span> <span class="k">do</span> 
            <span class="k">do!</span> <span class="id">action</span> 
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The single shake action is bound to <code>shake</code>, and then the method returns an async block. In the block, the <code>shake</code> binding can then be replicated 8 times, and thanks to F#'s awesome <code>for..in..do</code> syntax, we have a loop but without a counter variable. We're also still inside the async block so <code>do!</code> waits for the animation to completed on each run through the list of animations.

## Refactoring
This working version has some repeated code that we could clean up a little bit. We can take that out and put that into a function:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">awaitParallel</span> <span class="o">=</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="id">Async</span><span class="pn">.</span><span class="id">AwaitTask</span> <span class="pn">&gt;</span><span class="pn">&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Parallel</span> <span class="pn">&gt;</span><span class="pn">&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">Ignore</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If that looks confusing, check out my post on [point free notation]({{ "point-free-notation/" | relative_url }}) and also this post on [function composition]({{ "function-composition" | relative_url }}). With the helper function the rest of code now looks as follows:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">shakeLandscapeAsync</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">shake</span> <span class="o">=</span>  <span class="id">async</span> <span class="pn">{</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
    <span class="pn">}</span>
    <span class="id">async</span> <span class="pn">{</span> 
        <span class="k">for</span> <span class="id">action</span> <span class="k">in</span> <span class="id">List</span><span class="pn">.</span><span class="id">replicate</span> <span class="n">8</span> <span class="id">shake</span> <span class="k">do</span> 
            <span class="k">do!</span> <span class="id">action</span> 
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This is the final version that I left in my code base. I will mention one last step that could be done.
Running animations sequentially like this might be rather common in a mobile app. So the async loop could be refactored out. An easy way to do this would be as an extension method on Async. Here's an example:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// In AsyncExtensions.fs</span>
<span class="k">module</span> <span class="id">Async</span>

<span class="k">let</span> <span class="id">RunSequentially</span> <span class="id">asyncs</span> <span class="o">=</span> 
    <span class="id">async</span> <span class="pn">{</span>
        <span class="k">for</span> <span class="id">a</span> <span class="k">in</span> <span class="id">asyncs</span> <span class="k">do</span> 
            <span class="k">do!</span> <span class="id">a</span>
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">StartImmediate</span>   
</code></pre>
</td>
</tr>
</tbody>
</table>
Simple enough, it's just pulled out the loop logic, (similar to how map has factored our the logic of a loop). The caller would then be as follows:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">shakeLandscapeAsync</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="id">async</span> <span class="pn">{</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.1</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="o">-</span><span class="n">30.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
        <span class="k">do!</span> <span class="pn">[|</span>
                <span class="id">this</span><span class="pn">.</span><span class="id">ScaleTo</span><span class="pn">(</span><span class="n">1.0</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">TranslateTo</span><span class="pn">(</span><span class="n">0.</span><span class="pn">,</span> <span class="n">0.</span><span class="pn">,</span> <span class="n">20u</span><span class="pn">,</span> <span class="id">Easing</span><span class="pn">.</span><span class="id">Linear</span><span class="pn">)</span>
            <span class="pn">|]</span> <span class="o">|&gt;</span> <span class="id">awaitParallel</span>
    <span class="pn">}</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">replicate</span> <span class="n">8</span> <span class="o">|&gt;</span> <span class="id">RunSequentially</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Because we are using the shake directly, we no longer need to bind it to <code>shake</code>. The piping operator is once again our friend and the code is easy to follow.

## Summary
F#'s Async has all the power you need. let your codebase run wild with animations. Just remember that static methods on a C# <code>Task</code> are started immediately so they will need to be coupled with appropriate <code>!</code> op (ie <code>do!</code>, <code>let!</code> or <code>use!</code>).
