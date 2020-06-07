---
layout: post
title: How to free state and have fun with an F# Mailbox
date: 2017-06-13 09:53:31.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [State]
tags: [Database, Functional-Programming, MailboxProcessor, REPL, Scripting, State, Xamarin]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1591290682'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/pulling-state-from-f-mailboxes/
  _yoast_wpseo_focuskw_text_input: free state F# mailbox
  _yoast_wpseo_focuskw: F# mailbox reply
  _yoast_wpseo_metadesc: Getting state into an F# Mailbox is easy. Getting it back
    out is a little hard. Learn how to use F# mailbox reply to get state back and
    stop using mutables.
  _yoast_wpseo_linkdex: '77'
  _yoast_wpseo_content_score: '30'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '39'
  dsq_thread_id: '6132743194'
  _wp_old_slug: pulling-state-from-f-mailboxes
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2017/06/13/f-mailboxes-reply/"
---

## Prerequisites
- A simple understanding of F#
- A recap of F# Mailboxes: <a href="https://fsharpforfunandprofit.com/posts/concurrency-actor-model/">https://fsharpforfunandprofit.com/posts/concurrency-actor-model/</a>
- A basic understanding of an SQLite database

This post aims to build on the discussion started in the prerequisites. Mailbox processors are a great alternative to use variables and locks. There are many blog other posts on MailboxProcessors, however, few of them mention how to retrieve the state they hold rather than just printing it to console. This post will explain the different approaches while applying this to a typical Xamarin app problem; a database connection. Understanding how to use the F# mailbox reply methods will be easy after reading this post!

## Background
Before jumping in to the details, it is important to establish why this is important. State is a requirement of a program, but how that state is modelled is up to the programmer. Most main-stream programming languages (those OOP ones) model state with many variables. An alternative approach (and better in my opinion) is to eliminate most of the variables by modelling state through functions (how that is done is not the focus of this post). For those few variables that remain, a high quality can now taken to make sure everything is thread safe and protected. F# mailboxes are a nice way of doing that. As already stated, one of those variables is the database connection. With that out of the way, time for some code!

## A database connection: The standard approach
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">open</span> <span class="id">SQLite</span>
<span class="k">open</span> <span class="id">System</span>
<span class="k">open</span> <span class="id">System</span><span class="pn">.</span><span class="id">IO</span>

<span class="pn">[&lt;</span><span class="rt">CLIMutable</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="rt">UserName</span> <span class="o">=</span> <span class="pn">{</span><span class="id">Id</span><span class="pn">:</span> <span class="vt">Guid</span><span class="pn">;</span> <span class="id">FirstName</span><span class="pn">:</span> <span class="rt">string</span><span class="pn">;</span> <span class="id">LastName</span><span class="pn">:</span><span class="rt">string</span> <span class="pn">}</span>

<span class="k">type</span> <span class="rt">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">monitor</span> <span class="o">=</span> <span class="k">new</span> <span class="rt">Object</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">let</span> <span class="id">dbName</span> <span class="o">=</span> <span class="s">"database.db3"</span>
    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">connection</span><span class="pn">:</span> <span class="id">SQLiteConnection</span> <span class="o">=</span> <span class="k">null</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="fn">Init</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="fn">lock</span> <span class="id">monitor</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> 
            <span class="mv">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span> <span class="pn">)</span>
            <span class="mv">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="fn">ignore</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="fn">GetAllUsers</span><span class="pn">(</span><span class="pn">)</span><span class="pn">:</span> <span class="id">Seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span> <span class="o">=</span> 
        <span class="fn">lock</span> <span class="id">monitor</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> 
            <span class="mv">connection</span><span class="pn">.</span><span class="id">Table</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

    <span class="c">//Rest of read write data</span>

<span class="c">// In app startup </span>
<span class="k">let</span> <span class="id">deviceDatabasePath</span> <span class="o">=</span> <span class="s">""</span> <span class="c">// real implementation would db path from IOC container</span>
<span class="k">let</span> <span class="id">databaseManager</span> <span class="o">=</span> <span class="k">new</span> <span class="rt">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span>
<span class="fn">databaseManager</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">deviceDatabasePath</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As a starting point, this F# code is a very typical approach to creating a database connection that is thread safe. To explain the main points of the code, UserName is the DTO class that we're using for a single table. For the connection, a synchronous connection is being used. Normally an async connection is used, but in the context of this post, an async connection offers little benefits. I've also skipped any form of interfaces for testing and will leave adding that in as an exercise for the reader.

##  Step one: A mailbox
First, let's take out the lock and wrap the connection in a mailbox.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">open</span> <span class="id">SQLite</span>
<span class="k">open</span> <span class="id">System</span>
<span class="k">open</span> <span class="id">System</span><span class="pn">.</span><span class="id">IO</span>

<span class="pn">[&lt;</span><span class="id">CLIMutable</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="id">UserName</span> <span class="o">=</span> <span class="pn">{</span><span class="id">Id</span><span class="pn">:</span> <span class="id">Guid</span><span class="pn">;</span> <span class="id">FirstName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">;</span> <span class="id">LastName</span><span class="pn">:</span><span class="id">string</span> <span class="pn">}</span>
<span class="k">type</span> <span class="rt">Message</span> <span class="o">=</span> <span class="uc">Init</span> <span class="k">of</span> <span class="rt">string</span>

<span class="k">type</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="k">mutable</span> <span class="id">connection</span><span class="pn">:</span> <span class="id">SQLiteConnection</span> <span class="o">=</span> <span class="k">null</span>
    <span class="k">let</span> <span class="id">dbName</span> <span class="o">=</span> <span class="s">"database.db3"</span>
    <span class="k">let</span> <span class="id">mailbox</span> <span class="o">=</span> <span class="id">MailboxProcessor</span><span class="pn">.</span><span class="id">Start</span><span class="pn">(</span><span class="k">fun</span> <span class="id">inbox</span> <span class="k">-&gt;</span> 

        <span class="k">let</span> <span class="k">rec</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">async</span> <span class="pn">{</span>
            <span class="k">let!</span> <span class="id">msg</span> <span class="o">=</span> <span class="id">inbox</span><span class="pn">.</span><span class="id">Receive</span><span class="pn">(</span><span class="pn">)</span>
            <span class="k">match</span> <span class="id">msg</span> <span class="k">with</span> 
            <span class="pn">|</span> <span class="id">Init</span> <span class="id">path</span> <span class="k">-&gt;</span> 
                <span class="id">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span>
                <span class="id">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>

            <span class="k">return!</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span>
        <span class="pn">}</span>
        <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">path</span><span class="pn">:</span> <span class="id">string</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span><span class="pn">(</span><span class="id">Init</span> <span class="id">path</span><span class="pn">)</span>    

    <span class="c">// rest of methods to read/write data</span>

<span class="c">// In app startup </span>
<span class="k">let</span> <span class="id">deviceDatabasePath</span> <span class="o">=</span> <span class="s">""</span> <span class="c">// real implementation would db path from IOC container</span>
<span class="k">let</span> <span class="id">databaseManager</span> <span class="o">=</span> <span class="k">new</span> <span class="rt">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span>
<span class="fn">databaseManager</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">deviceDatabasePath</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Now we have a <code>MailboxProcessor</code> holding our <code>SQLiteConnection</code> and we also have a message type to create the connection. Because the connection is stored inside the mailbox, we know that things are thread safe. We're not finished yet though since there is no way to get data out of the <code>MailboxProcessor</code>

## Getting data out: The many possibilities
There are a few ways to get data out of the mailbox. They are:
- Reply synchronously
- Reply asynchronous
- use events

let's use each approach to get a feel for what works best. Here are the code sections that need to be added/updated for replying synchronously:

## Reply to me synchronously
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// Update the type to handle replying</span>
<span class="k">type</span> <span class="id">Message</span> <span class="o">=</span> <span class="id">Init</span> <span class="k">of</span> <span class="id">string</span> <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="k">of</span> <span class="id">AsyncReplyChannel</span><span class="pn">&lt;</span><span class="id">seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">&gt;</span>

    <span class="c">// update mailbox to handle the message and reply with the data</span>
    <span class="k">match</span> <span class="id">msg</span> <span class="k">with</span> 
    <span class="pn">|</span> <span class="id">Init</span> <span class="id">path</span> <span class="k">-&gt;</span> 
        <span class="id">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span>
        <span class="id">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>
    <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="id">replyChannel</span> <span class="k">-&gt;</span> 
        <span class="id">replyChannel</span><span class="pn">.</span><span class="id">Reply</span><span class="pn">(</span><span class="id">connection</span><span class="pn">.</span><span class="id">Table</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span> 

<span class="c">// Add a method on the class to use the mailbox and reply synchronously with the data</span>
<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">GetAllUsers</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
    <span class="id">mailbox</span><span class="pn">.</span><span class="id">PostAndReply</span> <span class="id">GetUserNames</span> 
</code></pre>
</td>
</tr>
</tbody>
</table>
<code>Message</code> now includes a branch with <code>GetUserNames</code> that also carries some data with it; the reply channel. The reply channel is also typed with the response data, in this case <code>seq&lt;UserName&gt;</code>. Once the message type has been updated, the pattern match inside the mailbox will give a warning till it has been updated to handle the <code>GetUserNames</code> case. The matching case is very simple. Just use the connection to pull out the data and pass it into the replyChannel's reply method. The last section of code is added to <code>DatabaseManager</code> as a public method. In here, we specify that the request should be made synchronously, ie post the message and block on the same thread for the response.

Here is the full output:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">open</span> <span class="id">SQLite</span>
<span class="k">open</span> <span class="id">System</span>
<span class="k">open</span> <span class="id">System</span><span class="pn">.</span><span class="id">IO</span>

<span class="pn">[&lt;</span><span class="id">CLIMutable</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="id">UserName</span> <span class="o">=</span> <span class="pn">{</span><span class="id">Id</span><span class="pn">:</span> <span class="id">Guid</span><span class="pn">;</span> <span class="id">FirstName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">;</span> <span class="id">LastName</span><span class="pn">:</span><span class="id">string</span> <span class="pn">}</span>
<span class="k">type</span> <span class="id">Message</span> <span class="o">=</span> <span class="id">Init</span> <span class="k">of</span> <span class="id">string</span> <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="k">of</span> <span class="id">AsyncReplyChannel</span><span class="pn">&lt;</span><span class="id">seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">&gt;</span>

<span class="k">type</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="k">mutable</span> <span class="id">connection</span><span class="pn">:</span> <span class="id">SQLiteConnection</span> <span class="o">=</span> <span class="k">null</span>
    <span class="k">let</span> <span class="id">dbName</span> <span class="o">=</span> <span class="s">"database.db3"</span>
    <span class="k">let</span> <span class="id">mailbox</span> <span class="o">=</span> <span class="id">MailboxProcessor</span><span class="pn">.</span><span class="id">Start</span><span class="pn">(</span><span class="k">fun</span> <span class="id">inbox</span> <span class="k">-&gt;</span> 

        <span class="k">let</span> <span class="k">rec</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">async</span> <span class="pn">{</span>
            <span class="k">let!</span> <span class="id">msg</span> <span class="o">=</span> <span class="id">inbox</span><span class="pn">.</span><span class="id">Receive</span><span class="pn">(</span><span class="pn">)</span>
            <span class="k">match</span> <span class="id">msg</span> <span class="k">with</span> 
            <span class="pn">|</span> <span class="id">Init</span> <span class="id">path</span> <span class="k">-&gt;</span> 
                <span class="id">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span>
                <span class="id">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>
            <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="id">replyChannel</span> <span class="k">-&gt;</span> 
                <span class="id">replyChannel</span><span class="pn">.</span><span class="id">Reply</span><span class="pn">(</span><span class="id">connection</span><span class="pn">.</span><span class="id">Table</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span> 
            
            <span class="k">return!</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span>
        <span class="pn">}</span>
        <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">path</span><span class="pn">:</span> <span class="id">string</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span><span class="pn">(</span><span class="id">Init</span> <span class="id">path</span><span class="pn">)</span>   

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">GetAllUsers</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">PostAndReply</span> <span class="id">GetUserNames</span> 

    <span class="c">// rest of methods to read/write data</span>

<span class="c">// In app startup </span>
<span class="k">let</span> <span class="id">deviceDatabasePath</span> <span class="o">=</span> <span class="s">""</span> <span class="c">// real implementation would db path from IOC container</span>
<span class="k">let</span> <span class="id">databaseManger</span> <span class="o">=</span> <span class="k">new</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">deviceDatabasePath</span><span class="pn">)</span>
<span class="k">let</span> <span class="id">usernames</span> <span class="o">=</span> <span class="id">databaseManger</span><span class="pn">.</span><span class="id">GetAllUsers</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## Doing things asynchronously
For the next variation, we have to reply asynchronously. To change the synchronous example to asynchronously, only one method needs to changed:
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
<pre class="fssnip highlighted"><code lang="fsharp">    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">GetAllUsers</span> <span class="pn">(</span><span class="pn">)</span><span class="pn">:</span> <span class="id">Async</span><span class="pn">&lt;</span><span class="id">seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">&gt;</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">PostAndAsyncReply</span> <span class="id">GetUserNames</span> 

    <span class="k">let</span> <span class="id">usernames</span> <span class="o">=</span> <span class="id">databaseManger</span><span class="pn">.</span><span class="id">GetAllUsers</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">Async</span><span class="pn">.</span><span class="id">RunSynchronously</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As stated, the change is very simple, just use <code>PostAndAsyncReply</code>, and the response will be asynchronous. For the example invoking the method, I have called it synchronously (<code>RunSynchronously</code>), but this should be avoided to get the benefits from asynchronous execution.

## An event to rule them all
The final choice for getting data out of a <code>MailboxProcessor</code> is by using events. Don Syme wrote a great a post on this <a title="https://blogs.msdn.microsoft.com/dsyme/2010/02/15/async-and-parallel-design-patterns-in-f-agents/">here</a>. Let's apply that solution to the <code>SQLiteConnection</code>. First off let's update the <code>Message</code> to have the values we need.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">Message</span> <span class="o">=</span> <span class="id">Init</span> <span class="k">of</span> <span class="id">string</span> <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="pn">|</span> <span class="id">Create</span> <span class="k">of</span> <span class="id">UserName</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The <code>GetUserNames</code> is now used as an enum since we'll use an event to return the data. I've also taken the liberty of adding another choice to the message that will allow a <code>UserName</code> to be added to the database. This choice can be added to any/all of the examples if you want to test them with some data. Now we need to add the event along with a few helper methods to the <code>DatabaseManager</code>
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
<span class="l">9: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp">    <span class="c">// Declared at the top, in DatabaseManager</span>
    <span class="k">let</span> <span class="id">event</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Event</span><span class="pn">&lt;</span><span class="id">seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">let</span> <span class="id">context</span> <span class="o">=</span> 
        <span class="k">match</span> <span class="id">SynchronizationContext</span><span class="pn">.</span><span class="id">Current</span> <span class="k">with</span> 
        <span class="pn">|</span> <span class="k">null</span> <span class="k">-&gt;</span> <span class="k">new</span> <span class="id">SynchronizationContext</span><span class="pn">(</span><span class="pn">)</span>
        <span class="pn">|</span> <span class="id">ctx</span> <span class="k">-&gt;</span> <span class="id">ctx</span>

    <span class="k">let</span> <span class="id">raiseEvent</span> <span class="id">args</span> <span class="o">=</span> 
        <span class="id">context</span><span class="pn">.</span><span class="id">Post</span><span class="pn">(</span><span class="pn">(</span><span class="k">fun</span> <span class="id">_</span> <span class="k">-&gt;</span> <span class="id">event</span><span class="pn">.</span><span class="id">Trigger</span> <span class="id">args</span><span class="pn">)</span><span class="pn">,</span> <span class="id">state</span><span class="o">=</span><span class="k">null</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As discussed earlier, the event is typed with our return type, in this case <code>seq&lt;UserName&gt;</code>. We also capture a context that we can post back on, generally this will be the main thread of a GUI app. Finally, a helper <code>raiseEvent</code> was added that raises the event on the captured context. Next up is to update the match block in our mailbox:
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
<span class="l">9: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// Inside the mailbox</span>
<span class="k">match</span> <span class="id">msg</span> <span class="k">with</span> 
<span class="pn">|</span> <span class="id">Init</span> <span class="id">path</span> <span class="k">-&gt;</span> 
    <span class="id">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span>
    <span class="id">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>
<span class="pn">|</span> <span class="id">GetUserNames</span> <span class="k">-&gt;</span> 
    <span class="id">connection</span><span class="pn">.</span><span class="id">Table</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">raiseEvent</span>
<span class="pn">|</span> <span class="id">Create</span> <span class="id">userName</span> <span class="k">-&gt;</span> 
    <span class="id">connection</span><span class="pn">.</span><span class="id">Insert</span><span class="pn">(</span><span class="id">userName</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The first match is the same as before. <code>GetUserNames</code> now loads all the data and calls the helper method we declared earlier. <code>Create</code> is also very simple, it inserts the userName into the database and ignores the return value of the insert. We're nearly done, we just need to expose the new functionality on the <code>DatabaseManager</code> via some methods.
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
<span class="l">9: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// In DatabaseManager</span>
<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Create</span><span class="pn">(</span><span class="id">firstName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">,</span> <span class="id">lastName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">)</span> <span class="o">=</span>
    <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span> <span class="o">&lt;|</span> <span class="id">Create</span> <span class="pn">{</span><span class="id">Id</span> <span class="o">=</span> <span class="id">Guid</span><span class="pn">.</span><span class="id">NewGuid</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span> <span class="id">FirstName</span> <span class="o">=</span> <span class="id">firstName</span><span class="pn">;</span> <span class="id">LastName</span> <span class="o">=</span> <span class="id">lastName</span><span class="pn">}</span>

<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ReceiveAllUsers</span> <span class="o">=</span> 
    <span class="id">event</span><span class="pn">.</span><span class="id">Publish</span>

<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">RequestAllUsers</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
    <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span> <span class="id">GetUserNames</span> 
</code></pre>
</td>
</tr>
</tbody>
</table>
Adding data is a single method that takes in the required. It simply makes a <code>Post</code> call to the mailbox with the <code>Create</code> tag and the data. Getting the data our now requires two methods. Because we're using events to get the data out, the client will need to subscribe to the event. <code>ReceiveAllUsers</code> allows the client to do this by exposing the <code>Publish</code> method on the event. Once the client has subscribed, the client can call <code>RequestAllUsers</code> and the data will be received on the event subscription. <code>RequestAllUsers</code> just makes a <code>Post</code> call to the mailbox with the <code>GetUserNames</code> choice.
Here is some sample client code I used in an F# repl to test this out:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">deviceDatabasePath</span> <span class="o">=</span> <span class="s">"/Users/sam.williams/Desktop/"</span> <span class="c">// real implementation would be db path from IOC container</span>
<span class="k">let</span> <span class="id">databaseManger</span> <span class="o">=</span> <span class="k">new</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">deviceDatabasePath</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">Create</span><span class="pn">(</span><span class="s">"Sam"</span><span class="pn">,</span> <span class="s">"Williams"</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">ReceiveAllUsers</span><span class="pn">.</span><span class="id">Add</span><span class="pn">(</span><span class="k">fun</span> <span class="id">users</span> <span class="k">-&gt;</span> <span class="id">users</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">iter</span> <span class="pn">(</span><span class="id">printfn</span> <span class="s">"%A"</span><span class="pn">)</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">RequestAllUsers</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
It's rather straight forward but I will go over it quickly. The first three lines are creating an instance and setting up the database as before. We then create a record so we can get something out. We then add our subscription to the <code>databaseManager</code> event. For this trivial example, the users will be printed to the console. Finally, the request is made to get all the users. The full listing for the mailbox with events is listed at the end of the post.

To wrap-up, there are three possible methods to get state out of an F# mailbox, reply synchronously, reply asynchronous and via events. Hopefully one of these will be suitable the next time you need to protect a variable from thread bugs.

## tl;dr: full fsx script example with events
Here is the full listing for the mailbox with events. I've included the required imports to run this as an fsx script on a Mac. For this script I've also included some very crude error handling, useful for getting the script running (but not production quality):
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
<span class="l">49: </span>
<span class="l">50: </span>
<span class="l">51: </span>
<span class="l">52: </span>
<span class="l">53: </span>
<span class="l">54: </span>
<span class="l">55: </span>
<span class="l">56: </span>
<span class="l">57: </span>
<span class="l">58: </span>
<span class="l">59: </span>
<span class="l">60: </span>
<span class="l">61: </span>
<span class="l">62: </span>
<span class="l">63: </span>
<span class="l">64: </span>
<span class="l">65: </span>
<span class="l">66: </span>
<span class="l">67: </span>
<span class="l">68: </span>
<span class="l">69: </span>
<span class="l">70: </span>
<span class="l">71: </span>
<span class="l">72: </span>
<span class="l">73: </span>
<span class="l">74: </span>
<span class="l">75: </span>
<span class="l">76: </span>
<span class="l">77: </span>
<span class="l">78: </span>
<span class="l">79: </span>
<span class="l">80: </span>
<span class="l">81: </span>
<span class="l">82: </span>
<span class="l">83: </span>
<span class="l">84: </span>
<span class="l">85: </span>
<span class="l">86: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pp">#r</span> <span class="s">"../packages/sqlite-net-pcl/lib/portable-net45+wp8+wpa81+win8+MonoAndroid10+MonoTouch10+Xamarin.iOS10/SQLite-net.dll"</span> 
<span class="pp">#r</span> <span class="s">"../packages/SQLitePCLRaw.core/lib/Xamarin.Mac20/SQLitePCLRaw.core.dll"</span>
<span class="pp">#r</span> <span class="s">"../packages/SQLitePCLRaw.provider.e_sqlite3.macos/lib/Xamarin.Mac20/SQLitePCLRaw.provider.e_sqlite3.dll"</span>
<span class="pp">#r</span> <span class="s">"../packages/SQLitePCLRaw.bundle_green/lib/Xamarin.Mac20/SQLitePCLRaw.batteries_v2.dll"</span>
<span class="pp">#r</span> <span class="s">"../packages/SQLitePCLRaw.provider.sqlite3.ios_unified/lib/Xamarin.iOS10/SQLitePCLRaw.provider.sqlite3.dll"</span>
<span class="pp">#r</span> <span class="s">"../packages/SQLitePCLRaw.provider.e_sqlite3.net45/lib/net45/SQLitePCLRaw.provider.e_sqlite3.dll"</span>

<span class="k">open</span> <span class="id">SQLite</span>
<span class="k">open</span> <span class="id">System</span>
<span class="k">open</span> <span class="id">System</span><span class="pn">.</span><span class="id">IO</span>
<span class="k">open</span> <span class="id">System</span><span class="pn">.</span><span class="id">Threading</span>

<span class="pn">[&lt;</span><span class="id">CLIMutable</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="id">UserName</span> <span class="o">=</span> <span class="pn">{</span><span class="id">Id</span><span class="pn">:</span> <span class="id">Guid</span><span class="pn">;</span> <span class="id">FirstName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">;</span> <span class="id">LastName</span><span class="pn">:</span><span class="id">string</span> <span class="pn">}</span>
<span class="k">type</span> <span class="id">Message</span> <span class="o">=</span> <span class="id">Init</span> <span class="k">of</span> <span class="id">string</span> <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="pn">|</span> <span class="id">Create</span> <span class="k">of</span> <span class="id">UserName</span>

<span class="k">type</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">event</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Event</span><span class="pn">&lt;</span><span class="id">seq</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">let</span> <span class="id">mainThread</span> <span class="o">=</span> 
        <span class="k">match</span> <span class="id">SynchronizationContext</span><span class="pn">.</span><span class="id">Current</span> <span class="k">with</span> 
        <span class="pn">|</span> <span class="k">null</span> <span class="k">-&gt;</span> <span class="k">new</span> <span class="id">SynchronizationContext</span><span class="pn">(</span><span class="pn">)</span>
        <span class="pn">|</span> <span class="id">ctx</span> <span class="k">-&gt;</span> <span class="id">ctx</span>

    <span class="k">let</span> <span class="id">raiseEvent</span> <span class="id">args</span> <span class="o">=</span> 
        <span class="id">mainThread</span><span class="pn">.</span><span class="id">Post</span><span class="pn">(</span><span class="pn">(</span><span class="k">fun</span> <span class="id">_</span> <span class="k">-&gt;</span> <span class="id">event</span><span class="pn">.</span><span class="id">Trigger</span> <span class="id">args</span><span class="pn">)</span><span class="pn">,</span> <span class="id">state</span><span class="o">=</span><span class="k">null</span><span class="pn">)</span>

    <span class="k">let</span> <span class="k">mutable</span> <span class="id">connection</span><span class="pn">:</span> <span class="id">SQLiteConnection</span> <span class="o">=</span> <span class="k">null</span>
    <span class="k">let</span> <span class="id">dbName</span> <span class="o">=</span> <span class="s">"database.db3"</span>
    <span class="k">let</span> <span class="id">mailbox</span> <span class="o">=</span> <span class="id">MailboxProcessor</span><span class="pn">.</span><span class="id">Start</span><span class="pn">(</span><span class="k">fun</span> <span class="id">inbox</span> <span class="k">-&gt;</span> 
    
        <span class="c">// Helper to get out any exceptions that may occur</span>
        <span class="k">let</span> <span class="k">rec</span> <span class="id">getException</span> <span class="id">indent</span> <span class="pn">(</span><span class="id">e</span><span class="pn">:</span><span class="id">exn</span><span class="pn">)</span> <span class="o">=</span> 
            <span class="id">printfn</span> <span class="s">"Finding exception message"</span>
            <span class="k">if</span> <span class="id">e</span><span class="pn">.</span><span class="id">InnerException</span> <span class="o">=</span> <span class="k">null</span> <span class="k">then</span> 
                <span class="id">sprintf</span> <span class="s">"%s\n%s"</span> <span class="id">e</span><span class="pn">.</span><span class="id">Message</span> <span class="id">e</span><span class="pn">.</span><span class="id">StackTrace</span>
            <span class="k">else</span> 
                <span class="id">sprintf</span> <span class="s">"%s%s\n%s"</span> <span class="id">indent</span> <span class="pn">(</span><span class="id">getException</span> <span class="pn">(</span><span class="id">sprintf</span> <span class="s">"%s\t"</span> <span class="id">indent</span><span class="pn">)</span> <span class="id">e</span><span class="pn">.</span><span class="id">InnerException</span><span class="pn">)</span> <span class="id">e</span><span class="pn">.</span><span class="id">StackTrace</span> 

        <span class="c">// generalised catch for any exceptions on each branch</span>
        <span class="k">let</span> <span class="id">catch</span> <span class="id">f</span> <span class="o">=</span> 
            <span class="k">try</span> 
                <span class="id">f</span> <span class="pn">(</span><span class="pn">)</span> 
            <span class="k">with</span> 
            <span class="pn">|</span> <span class="id">e</span> <span class="k">-&gt;</span> <span class="id">printfn</span> <span class="s">"Error: %s\n%s"</span> <span class="id">e</span><span class="pn">.</span><span class="id">Message</span> <span class="pn">(</span><span class="id">getException</span> <span class="s">""</span> <span class="id">e</span><span class="pn">)</span>
            
        <span class="k">let</span> <span class="k">rec</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">async</span> <span class="pn">{</span>
            <span class="k">let!</span> <span class="id">msg</span> <span class="o">=</span> <span class="id">inbox</span><span class="pn">.</span><span class="id">Receive</span><span class="pn">(</span><span class="pn">)</span>
            <span class="k">match</span> <span class="id">msg</span> <span class="k">with</span> 
            <span class="pn">|</span> <span class="id">Init</span> <span class="id">path</span> <span class="k">-&gt;</span> 
                <span class="id">catch</span> <span class="o">&lt;|</span> <span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> 
                    <span class="id">connection</span> <span class="k">&lt;-</span> <span class="k">new</span> <span class="id">SQLiteConnection</span><span class="pn">(</span><span class="id">Path</span><span class="pn">.</span><span class="id">Combine</span><span class="pn">(</span><span class="id">path</span><span class="pn">,</span> <span class="id">dbName</span><span class="pn">)</span><span class="pn">,</span> <span class="k">false</span><span class="pn">)</span>
                    <span class="id">connection</span><span class="pn">.</span><span class="id">CreateTable</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>
                    <span class="id">printfn</span> <span class="s">"Connection created!"</span>
            <span class="pn">|</span> <span class="id">GetUserNames</span> <span class="k">-&gt;</span> 
                <span class="id">catch</span> <span class="o">&lt;|</span> <span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> 
                    <span class="id">connection</span><span class="pn">.</span><span class="id">Table</span><span class="pn">&lt;</span><span class="id">UserName</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">raiseEvent</span>
            <span class="pn">|</span> <span class="id">Create</span> <span class="id">userName</span> <span class="k">-&gt;</span> 
                <span class="id">catch</span> <span class="o">&lt;|</span> <span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> 
                    <span class="id">connection</span><span class="pn">.</span><span class="id">Insert</span><span class="pn">(</span><span class="id">userName</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span>

            <span class="k">return!</span> <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span>
        <span class="pn">}</span>
        <span class="id">loop</span> <span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">path</span><span class="pn">:</span> <span class="id">string</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span><span class="pn">(</span><span class="id">Init</span> <span class="id">path</span><span class="pn">)</span>   

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Create</span><span class="pn">(</span><span class="id">firstName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">,</span> <span class="id">lastName</span><span class="pn">:</span> <span class="id">string</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span> <span class="o">&lt;|</span> <span class="id">Create</span> <span class="pn">{</span><span class="id">Id</span> <span class="o">=</span> <span class="id">Guid</span><span class="pn">.</span><span class="id">NewGuid</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span> <span class="id">FirstName</span> <span class="o">=</span> <span class="id">firstName</span><span class="pn">;</span> <span class="id">LastName</span> <span class="o">=</span> <span class="id">lastName</span><span class="pn">}</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">RequestAllUsers</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">mailbox</span><span class="pn">.</span><span class="id">Post</span> <span class="id">GetUserNames</span> 

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ReceiveAllUsers</span> <span class="o">=</span> 
        <span class="id">event</span><span class="pn">.</span><span class="id">Publish</span>

    <span class="c">// rest of methods to read/write data</span>

<span class="c">// In app startup </span>
<span class="k">let</span> <span class="id">deviceDatabasePath</span> <span class="o">=</span> <span class="s">"/Users/sam.williams/Desktop/"</span> <span class="c">// real implementation would db path from IOC container</span>
<span class="k">let</span> <span class="id">databaseManger</span> <span class="o">=</span> <span class="k">new</span> <span class="id">DatabaseManager</span><span class="pn">(</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">Init</span><span class="pn">(</span><span class="id">deviceDatabasePath</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">Create</span><span class="pn">(</span><span class="s">"Sam"</span><span class="pn">,</span> <span class="s">"Williams"</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">ReceiveAllUsers</span><span class="pn">.</span><span class="id">Add</span><span class="pn">(</span><span class="k">fun</span> <span class="id">users</span> <span class="k">-&gt;</span> <span class="id">users</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">iter</span> <span class="pn">(</span><span class="id">printfn</span> <span class="s">"%A"</span><span class="pn">)</span><span class="pn">)</span>
<span class="id">databaseManger</span><span class="pn">.</span><span class="id">RequestAllUsers</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>