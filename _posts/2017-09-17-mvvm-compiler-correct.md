---
layout: post
title: How to MVVM that you know is compiler correct
date: 2017-09-17 09:35:59.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Design-Patterns, F#, MVVM]
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '60'
  mashsb_timestamp: '1587694782'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '29'
  mashsb_shares: '6'
  mashsb_jsonshares: '{"total":6,"error":"","facebook_shares":4,"twitter":2,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/mvvm-compiler-correct/
  _yoast_wpseo_focuskw: MVVM compiler checked
  _yoast_wpseo_focuskw_text_input: MVVM compiler checked
  _yoast_wpseo_title: "%%title%% %%sep%% %%sitename%%"

  _yoast_wpseo_linkdex: '68'
  dsq_thread_id: '6149664161'
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
description: MVVM can be hard to maintain. Using a compiler to enforce MVVM would be effective. See how MVVM can be compiler checked with an example in just 25 lines
permalink: "/2017/09/17/mvvm-compiler-correct/"
---
MVVM is considered by many developers as the defacto design pattern to be used for creating GUI apps. Due to the nature of it being a design pattern, sometimes developers do not implement it correctly, or the architecture of the app simply deteriorates over time. Utilising the hierarchical nature of the pattern, a single pass compiler enforces the pattern, which is the type of compiler that F# adopted. If a developer wrote a GUI app with F#, than the MVVM implementation will be compiler checked. This post highlights why a code base might not be following MVVM correctly. Secondly, an MVVM compiler checked example is then shown. The last section covers a few practical steps that can be taken for individuals that have a C# code base.

## What is MVVM?
MVVM is a design pattern for UI applications. Check out the <a title="MVVM" href="https://developer.xamarin.com/guides/xamarin-forms/enterprise-application-patterns/mvvm/">documentation from Xamarin</a> for a full explanation.

## Who has a code base that implements MVVM correctly?
Developers that have had some experience with MVVM will know that not every code base follows the pattern. Developers sometimes make mistakes (also known as bugs), and unless the developer writes unit tests for every line of code, it can can be hard for the developer to know that code base adheres to MVVM. Unfortunately the lack of enforcement is that nature of design patterns, since the compiler does not understand the pattern.

Unit tests increase the developer's confidence that the code base correctly implements MVVM. Although, this assumes that the developer of the unit tests wrote them correctly. Additionally, many developers use a framework to abstract away the boilerplate code. Sometimes the frameworks can make it hard to correctly unit test.
To the individuals that have a code base covered with unit tests great work, but consider the event where another developer is brought into the team. The lead developer must now check the new developer's code as well as his/her own. Time for a tech lead/management to add some more process with pull requests (PR).
The teammate is now now writing unit tests and following MVVM with a PRs for the senior dev to keep and eye on. However, the senior dev needs to take a holiday, it's been a while! Who will keep an eye on MVVM now?

## How to invert the problem to make it easier
All of the above, though being very good, are process heavy tools. At any stage of development, the architecture of the code base can deteriorate, without a clear sign to the developer. As an alternative, it would be better if the language/APIs/compiler guided the developer to write the app with the correct architecture. At a high level, Mark Seemann addresses the architecture problem and highlights a solution known as the pit of success. Here is the talk that Mark gave on this and is worth the watch: <a href="https://skillsmatter.com/skillscasts/7355-functional-architecture-the-pit-of-success">Pit of success</a>.

## A compiler can check an MVVM implementation
MVVM has a clear hierarchy, resulting in a layered architecture. Each layer only has access to the items above it. The model layer only has access to itself, including Services/Managers and any data objects. View models have access to all the models and services. Finally the UI container element (Page/Window) has access to everything.

Given the hierarchy of the design pattern, one way to enforce the pattern is to require that each layer be declared before using it. It turns out that a single pass compiler enforces this. For example, a class named <code>Foo</code> must first be declared. After it has been declared, only then, can an instance be created.
Searching for an appropriate compiler, highlights that F# utilizes a single pass compiler. F# runs on .NET as well as having Object-Oriented support.

## MVVM that is compiler checked - 25 lines of code
To demonstrate a correct implementation of MVVM, the example will need to include all the right pieces. The following will be included:

- a model as a data type

- a service that operates on the model and has an interface with an implementation

- a view model that takes in the interface

- a page element to glue it all together
Xamarin Forms has been used to demonstrate this. A link to the repository with all of the code is at the end of this post. Here is the code:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">namespace</span> <span class="id">CompilerCheckedMvvm</span>
<span class="k">open</span> <span class="id">Xamarin</span><span class="pn">.</span><span class="id">Forms</span>
<span class="k">open</span> <span class="id">Xamarin</span><span class="pn">.</span><span class="id">Forms</span><span class="pn">.</span><span class="id">Xaml</span>

<span class="c">// An alias to show where the model as data is used</span>
<span class="k">type</span> <span class="id">Model</span> <span class="o">=</span> <span class="id">int</span>

<span class="c">// Interface</span>
<span class="k">type</span> <span class="id">IDataLoaderService</span> <span class="o">=</span> 
    <span class="k">abstract</span> <span class="k">member</span> <span class="id">LoadData</span><span class="pn">:</span> <span class="id">unit</span> <span class="k">-&gt;</span> <span class="id">Model</span>

<span class="c">// Implementation of interface</span>
<span class="k">type</span> <span class="id">DataLoaderService</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">interface</span> <span class="id">IDataLoaderService</span> <span class="k">with</span> 
        <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">LoadData</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="n">42</span>

<span class="c">// View model with service as a constructor argument</span>
<span class="k">type</span> <span class="id">MainViewModel</span><span class="pn">(</span><span class="id">_dataLoader</span><span class="pn">:</span> <span class="id">IDataLoaderService</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Data</span> <span class="o">=</span> <span class="id">_dataLoader</span><span class="pn">.</span><span class="id">LoadData</span><span class="pn">(</span><span class="pn">)</span><span class="pn">.</span><span class="id">ToString</span><span class="pn">(</span><span class="pn">)</span>

<span class="k">type</span> <span class="id">MainPage</span><span class="pn">(</span><span class="pn">)</span> <span class="k">as</span> <span class="id">this</span> <span class="o">=</span>
    <span class="k">inherit</span> <span class="id">ContentPage</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">let</span> <span class="id">_</span> <span class="o">=</span> <span class="k">base</span><span class="pn">.</span><span class="id">LoadFromXaml</span><span class="pn">(</span><span class="id">typeof</span><span class="pn">&lt;</span><span class="id">MainPage</span><span class="pn">&gt;</span><span class="pn">)</span>
    <span class="k">do</span> <span class="id">this</span><span class="pn">.</span><span class="id">BindingContext</span> <span class="k">&lt;-</span> <span class="id">MainViewModel</span><span class="pn">(</span><span class="id">DataLoaderService</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Re-arranging the order of the code above will result in a compiler error. Additionally with such a short example, most developers will be able to learn that pattern. It starts with the model, then reading further down, the view model is declared. Finally at the bottom of the file, is the UI where everything is put together.
It is important to highlight that for teams and developer that know F#, using F# to correctly implement the MMVM design pattern is more cost effective than adding all of the processes stated earlier.

## 4 things to do if you have a C# app
The simplest solution is to use F# as a resource and training ground. After some practice with F#, take the knowledge back to the existing code base and cary on. Many developers will find this strategy extremely easy to implement, however the benefits are limited.

Another approach is to use F# script files to create simple port of the existing code base. The script file can then be compiled to check that everything is setup correctly. If the compiler identifies any problems, the existing code base can be modified and updated. The developer could also choose to commit the script file into a repo to be maintained on a regular regular cadence.

Building on the approach of a script file, a developer could create a simple port of the existing app. As with the script file, the developer only needs to port the important parts of the app as the goal is for understanding and compiler verification. Porting a basic version of the app to F# has additional benefits as well. This strategy is both safe and practical for any developer to learn and master F#. F# learning will be fast tracked as only the relevant parts of the language need to be understood. For those interested in using MvvmCross find out how to get started: [How to mvvmCross with F# that will increase your productivity]({{ "/2017/05/22/xamarin-forms-mvvmcross-fsharp/" | relative_url }})

The final strategy, though the most challenging, is to replace the C# app with an F# port. In many circumstances this will not be feasible but it may not be as challenging as one might first think. Before a developer starts such a large change, it is important that the developer includes the relevant people, and ensures everyone is in support. F# has many great benefits, but rushing into a port can result in <a href="https://blog.daftcode.pl/hype-driven-development-3469fc2e9b22">Hype Driven Development</a>, which ultimately has a negative effect on everyone involved. The goal is to have a significant positive impact for the end user of the software.

## How to start a port that will be successful
For a developer to get started on such a port, small steps need to be taken. The app does not need to be ported all at once. F# and C# compatibility is very good. Create an F# project, move a subset of code to the project. Then reference the new F# project from the existing C# project. Consider starting with a unit test project as this also reduces the risk. One project at a time, is the key here.

## What to remember
Here's a bullet list of the important stuff

- MVVM is hard to follow, as the compiler can not enforce it in C#
- MVVM is enforced by the F# compiler
- Here is a <a title="GitHub: MVVM in F#" href="http://bit.ly/mvvmfsharpgithub">compiler verified example</a> as a Xamarin Forms app
- If you have a C# code base, here's how to start with F#:

- For learning, to apply to a C# code base
- with a script file port of the existing code base
- create a basic port in F#
- In rare cases: Replace the app with an F# port.
