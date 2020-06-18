---
layout: post
title: BDD and gherkin without IDE integrations
date: 2019-02-15 19:00:37.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI, Testing]
tags: [BDD, F#, Mobile, Testing, UITest]
meta:
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591284250'
  _yoast_wpseo_primary_category: '36'
  _yoast_wpseo_content_score: '60'
  mashsb_jsonshares: '{"facebook_total":1,"facebook_likes":1,"facebook_comments":0}'
  mashsb_shares: '1'

# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
description: SpecFlow uses an IDE integration/extension. Instead use TickSpec.
NO IDE addon required. Supports F# / C#. Full example listed below
permalink: "/2019/02/15/bdd-and-gherkin-without-ide-integrations/"
---
SpecFlow uses an IDE integration (extension, addon, plugin or whatever it's called). Instead use TickSpec. NO IDE extension/addon required. Supports F# / C#. Full example listed below

## Cucumber, Specflow and Gherkin
Cucumber is the most the most popular library for this, supporting Ruby, Java, JavaScript. For .NET though, Specflow has become the most popular.

Both libraries tend to agree on the same syntax plain text file syntax though; gherkin.

### BDD with gherkin recap
A quick re-cap, BDD is the act of defining automated tests by the behaviour of the system (an alternative to TDD). Because BDD focuses on behaviour, tests are typically called

outside-in. Said another way, they sit higher up on the test pyramid and they test much of the application when compared to TDD tests. BDD style tests could be written in source code, however, because they involve writing out what the application should do, this tends to involve other people (those who don't write code). As a result, gherkin was introduced as a simple plain text language to describe the tests so those without little coding experience could read them.

### Gherkin language
The language is very simple and has 3 main keywords, <code>given</code>, <code>when</code>, <code>then</code>. The full spec is here: <a href="https://docs.cucumber.io/gherkin/reference/">https://docs.cucumber.io/gherkin/reference/</a>
here is a simple example (taken from the docs):
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">Feature</span><span class="pn">:</span> <span class="id">Guess</span> <span class="id">the</span> <span class="id">word</span>

<span class="pn">#</span><span class="id">The</span> <span class="id">first</span> <span class="id">example</span> <span class="id">has</span> <span class="id">two</span> <span class="id">steps</span>
<span class="id">Scenario</span><span class="pn">:</span> <span class="id">Maker</span> <span class="id">starts</span> <span class="id">a</span> <span class="id">game</span>
    <span class="id">When</span> <span class="id">the</span> <span class="id">Maker</span> <span class="id">starts</span> <span class="id">a</span> <span class="id">game</span>
    <span class="id">Then</span> <span class="id">the</span> <span class="id">Maker</span> <span class="id">waits</span> <span class="k">for</span> <span class="id">a</span> <span class="id">Breaker</span> <span class="k">to</span> <span class="id">join</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

### SpecFlow makes this hard
Specflow makes all of this possible by having a plugin to the IDE, Visual Studio and community support for Visual Studio For Mac (thanks @jimbobbennett). This is required, as Specflow emits code when the user saves the gherkin file (called the feature file). Sometimes things change in the IDE, and these tools are broken, this is more of an issue on VS4Mac than windows. I also don't like code gen when it can be avoided - it feels like magic and code should be understood.

## TickSpec as the alternative
There is a lesser known library that provides the same functionality called <a href="https://github.com/fsprojects/TickSpec">https://github.com/fsprojects/TickSpec</a>. It supports the gherkin language and does not require an IDE plugin. Without the plugin, there is no syntax highlighting, but these style of tests are not for developers.
Best of all, with TickSpec, there is not code gen. No magic. Nothing to go stale.
With a few code tweaks, this library can be used to build out feature files (plain text gherkin language tests) for a Xamarin app, running on either Mac or Windows.

### Creating a Xamarin UI test with TickSpec
- Given an existing app that needs testing
- Add a new UI test project
- Update Xamarin.UITest to the latest
- Add TickSpec NuGet package

All code snippets will be in F# (because it's an awesome language), C# is supported too though. You can even write your app in C#, and make just this UITest project in F#.

It's available in the drop-down when you create the project.
To bootstrap TickSpec, a bit of glue code is required for the tests to be discovered for each platform. Add the following to your <code>AppInitializer</code>
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">module</span> <span class="id">AppInitializer</span> <span class="o">=</span>

    <span class="c">// Sadly we need a variable and null :(. It will be the only one though. </span>
    <span class="k">let</span> <span class="k">mutable</span> <span class="id">app</span><span class="pn">:</span> <span class="id">IApp</span> <span class="o">=</span> <span class="k">null</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If you're using C#, then create a new file and translate the following code to C#. If you're us F#, then add the following to the bottom of your <code>AppInitializer.fs</code> file.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">/// Class containing all BDD tests in current assembly as NUnit unit tests</span>
<span class="pn">[&lt;</span><span class="id">TestFixture</span><span class="pn">&gt;]</span>
<span class="k">type</span> <span class="id">FeatureFixture</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
    <span class="c">/// Test method for all BDD tests in current assembly as NUnit unit tests</span>
    <span class="pn">[&lt;</span><span class="id">Test</span><span class="pn">&gt;]</span>
    <span class="pn">[&lt;</span><span class="id">TestCaseSource</span><span class="pn">(</span><span class="s">"Scenarios"</span><span class="pn">)</span><span class="pn">&gt;]</span>
    <span class="k">member</span> <span class="id">__</span><span class="pn">.</span><span class="id">Bdd</span> <span class="pn">(</span><span class="id">scenario</span><span class="pn">:</span><span class="id">Scenario</span><span class="pn">)</span> <span class="o">=</span>
        <span class="k">if</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Tags</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">exists</span> <span class="pn">(</span><span class="pn">(</span><span class="o">=</span><span class="pn">)</span> <span class="s">"ignore"</span><span class="pn">)</span> <span class="k">then</span>
            <span class="id">raise</span> <span class="pn">(</span><span class="k">new</span> <span class="id">IgnoreException</span><span class="pn">(</span><span class="s">"Ignored: "</span> <span class="o">+</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">ToString</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span><span class="pn">)</span>
        <span class="k">try</span>
            <span class="k">let</span> <span class="id">platform</span> <span class="o">=</span> 
                <span class="k">match</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Tags</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">contains</span> <span class="s">"android"</span><span class="pn">,</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Tags</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">contains</span> <span class="s">"ios"</span> <span class="k">with</span> 
                <span class="pn">|</span> <span class="k">true</span><span class="pn">,</span> <span class="k">true</span> <span class="k">-&gt;</span> <span class="id">failwith</span> <span class="s">"Can't run both ios and android. Check your spelling for the tags"</span> 
                <span class="pn">|</span> <span class="k">false</span><span class="pn">,</span> <span class="k">false</span><span class="k">-&gt;</span> <span class="id">failwith</span> <span class="s">"Must run with platform either ios or android. Add one of: @android, @ios, @android_ios"</span> 
                <span class="pn">|</span> <span class="k">true</span><span class="pn">,</span> <span class="k">false</span> <span class="k">-&gt;</span> <span class="id">Platform</span><span class="pn">.</span><span class="id">Android</span>
                <span class="pn">|</span> <span class="k">false</span><span class="pn">,</span> <span class="k">true</span> <span class="k">-&gt;</span> <span class="id">Platform</span><span class="pn">.</span><span class="id">iOS</span>

            <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span> <span class="k">&lt;-</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">startApp</span> <span class="id">platform</span>
            <span class="id">scenario</span><span class="pn">.</span><span class="id">Action</span><span class="pn">.</span><span class="id">Invoke</span><span class="pn">(</span><span class="pn">)</span>
        <span class="k">with</span>
        <span class="pn">|</span> <span class="id">ex</span> <span class="k">-&gt;</span> 
            <span class="id">eprintf</span> <span class="s">"Failed: %s\n%s"</span> <span class="id">ex</span><span class="pn">.</span><span class="id">Message</span> <span class="id">ex</span><span class="pn">.</span><span class="id">StackTrace</span>
            <span class="id">sprintf</span> <span class="s">"Failed: %s\n%s"</span> <span class="id">ex</span><span class="pn">.</span><span class="id">Message</span> <span class="id">ex</span><span class="pn">.</span><span class="id">StackTrace</span> <span class="o">|&gt;</span> <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span> 
            <span class="id">raise</span> <span class="id">ex</span>


    <span class="c">/// All test scenarios from feature files in current assembly</span>
    <span class="k">static</span> <span class="k">member</span> <span class="id">Scenarios</span> <span class="o">=</span>
    
        <span class="k">let</span> <span class="id">enhanceScenarioName</span> <span class="id">parameters</span> <span class="id">scenarioName</span> <span class="o">=</span>
            <span class="k">let</span> <span class="id">replaceParameterInScenarioName</span> <span class="pn">(</span><span class="id">scenarioName</span><span class="pn">:</span><span class="id">string</span><span class="pn">)</span> <span class="id">parameter</span> <span class="o">=</span>
                <span class="id">scenarioName</span><span class="pn">.</span><span class="id">Replace</span><span class="pn">(</span><span class="s">"&lt;"</span> <span class="o">+</span> <span class="id">fst</span> <span class="id">parameter</span> <span class="o">+</span> <span class="s">"&gt;"</span><span class="pn">,</span> <span class="id">snd</span> <span class="id">parameter</span><span class="pn">)</span>
            <span class="id">parameters</span>
            <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">fold</span> <span class="id">replaceParameterInScenarioName</span> <span class="id">scenarioName</span>

        <span class="k">let</span> <span class="id">splitTags</span> <span class="pn">(</span><span class="id">tags</span><span class="pn">:</span> <span class="id">string</span><span class="pn">[</span><span class="pn">]</span><span class="pn">)</span> <span class="o">=</span> 
            <span class="id">tags</span>
            <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span><span class="pn">.</span><span class="id">Split</span><span class="pn">(</span><span class="s">"_"</span><span class="pn">)</span><span class="pn">)</span> 
            <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">concat</span>
            <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span><span class="pn">.</span><span class="id">Replace</span><span class="pn">(</span><span class="s">"_"</span><span class="pn">,</span> <span class="s">""</span><span class="pn">)</span><span class="pn">.</span><span class="id">Trim</span><span class="pn">(</span><span class="pn">)</span><span class="pn">.</span><span class="id">ToLower</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

        <span class="k">let</span> <span class="id">isPlatform</span> <span class="pn">(</span><span class="id">name</span><span class="pn">:</span><span class="id">string</span><span class="pn">)</span> <span class="id">tags</span> <span class="pn">(</span><span class="id">scenario</span><span class="pn">:</span><span class="id">Scenario</span><span class="pn">)</span> <span class="id">feature</span> <span class="o">=</span> 
            <span class="k">if</span> <span class="id">tags</span> <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">contains</span> <span class="pn">(</span><span class="id">name</span><span class="pn">.</span><span class="id">ToLower</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span> <span class="k">then</span> 
                <span class="k">let</span> <span class="id">scenario</span> <span class="o">=</span> 
                    <span class="pn">{</span><span class="id">scenario</span> <span class="k">with</span> 
                        <span class="id">Name</span> <span class="o">=</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Name</span> <span class="o">|&gt;</span> <span class="id">sprintf</span> <span class="s">"%s: %s"</span> <span class="id">name</span>
                        <span class="id">Tags</span> <span class="o">=</span> <span class="id">tags</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">filter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span> <span class="o">=</span> <span class="pn">(</span><span class="id">name</span><span class="pn">.</span><span class="id">ToLower</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span><span class="pn">)</span> <span class="pn">}</span>
                <span class="pn">(</span><span class="k">new</span> <span class="id">TestCaseData</span><span class="pn">(</span><span class="id">scenario</span><span class="pn">)</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetName</span><span class="pn">(</span><span class="id">enhanceScenarioName</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Parameters</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Name</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetProperty</span><span class="pn">(</span><span class="s">"Feature"</span><span class="pn">,</span> <span class="id">feature</span><span class="pn">.</span><span class="id">Name</span><span class="pn">.</span><span class="id">Substring</span><span class="pn">(</span><span class="n">9</span><span class="pn">)</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetCategory</span><span class="pn">(</span><span class="id">name</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">Some</span>
            <span class="k">else</span> <span class="id">None</span>

        <span class="k">let</span> <span class="id">createTestCaseData</span> <span class="pn">(</span><span class="id">feature</span><span class="pn">:</span><span class="id">Feature</span><span class="pn">)</span> <span class="pn">(</span><span class="id">scenario</span><span class="pn">:</span><span class="id">Scenario</span><span class="pn">)</span> <span class="o">=</span>
            <span class="k">let</span> <span class="id">tags</span> <span class="o">=</span> <span class="id">splitTags</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Tags</span>  

            <span class="pn">[</span><span class="id">isPlatform</span> <span class="s">"Android"</span><span class="pn">;</span> <span class="id">isPlatform</span> <span class="s">"iOS"</span><span class="pn">]</span>
            <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">choose</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">f</span> <span class="k">-&gt;</span> <span class="id">f</span> <span class="id">tags</span> <span class="id">scenario</span> <span class="id">feature</span><span class="pn">)</span>
            <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">foldBack</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="id">tag</span><span class="pn">:</span><span class="id">string</span><span class="pn">)</span> <span class="id">tests</span> <span class="k">-&gt;</span> 
                <span class="id">tests</span> <span class="o">|&gt;</span> <span class="id">List</span><span class="pn">.</span><span class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">data</span> <span class="k">-&gt;</span> <span class="id">data</span><span class="pn">.</span><span class="id">SetProperty</span><span class="pn">(</span><span class="s">"Tag"</span><span class="pn">,</span> <span class="id">tag</span><span class="pn">)</span><span class="pn">)</span><span class="pn">)</span> <span class="id">scenario</span><span class="pn">.</span><span class="id">Tags</span>

        <span class="k">let</span> <span class="id">createFeatureData</span> <span class="pn">(</span><span class="id">feature</span><span class="pn">:</span><span class="id">Feature</span><span class="pn">)</span> <span class="o">=</span>
            <span class="id">feature</span><span class="pn">.</span><span class="id">Scenarios</span>
            <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">map</span> <span class="pn">(</span><span class="id">createTestCaseData</span> <span class="id">feature</span><span class="pn">)</span>
            <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">concat</span>
        
        <span class="k">let</span> <span class="id">assembly</span> <span class="o">=</span> <span class="id">Assembly</span><span class="pn">.</span><span class="id">GetExecutingAssembly</span><span class="pn">(</span><span class="pn">)</span>
        <span class="k">let</span> <span class="id">definitions</span> <span class="o">=</span> <span class="k">new</span> <span class="id">StepDefinitions</span><span class="pn">(</span><span class="id">assembly</span><span class="pn">.</span><span class="id">GetTypes</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>

        <span class="id">assembly</span><span class="pn">.</span><span class="id">GetManifestResourceNames</span><span class="pn">(</span><span class="pn">)</span>
        <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">filter</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="id">n</span><span class="pn">:</span><span class="id">string</span><span class="pn">)</span> <span class="k">-&gt;</span> <span class="id">n</span><span class="pn">.</span><span class="id">EndsWith</span><span class="pn">(</span><span class="s">".feature"</span><span class="pn">)</span> <span class="pn">)</span>
        <span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">collect</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">n</span> <span class="k">-&gt;</span>
            <span class="id">definitions</span><span class="pn">.</span><span class="id">GenerateFeature</span><span class="pn">(</span><span class="id">n</span><span class="pn">,</span> <span class="id">assembly</span><span class="pn">.</span><span class="id">GetManifestResourceStream</span><span class="pn">(</span><span class="id">n</span><span class="pn">)</span><span class="pn">)</span>
            <span class="o">|&gt;</span> <span class="id">createFeatureData</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The code snippet above primarily does two things.
- create NUnit tests from the plain text feature files
- create the test for each platform specified in the test

No tests have been added yet, so there won't be anything to see. Let's add some.

- add a plain text file to the project with the suffix <code>.feature</code>
- set the build action to EmbeddedResource
- for each test (<code>scenario</code>) add the following on the line above <code>@android_ios</code>

Here is an example of a feature file with one test that will run on both Android and iOS. If you want only one platform then use only that name ie @android or @ios
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">Feature</span><span class="pn">:</span> <span class="id">Login</span> <span class="id">Screen</span> <span class="id">works</span> <span class="id">correctly</span> 

<span class="id">@</span><span class="id">android_ios</span>
<span class="id">Scenario</span> <span class="n">1</span><span class="pn">:</span> <span class="id">I</span> <span class="id">can</span> <span class="id">login</span> <span class="k">to</span> <span class="id">the</span> <span class="id">app</span> <span class="k">with</span> <span class="id">the</span> <span class="id">correct</span> <span class="id">credentials</span>
    <span class="id">Given</span> <span class="id">I</span> <span class="id">enter</span> <span class="id">the</span> <span class="id">username</span> <span class="id">admin</span>
    <span class="id">And</span> <span class="id">I</span> <span class="id">enter</span> <span class="id">the</span> <span class="id">password</span> <span class="id">admin</span>
    <span class="id">When</span> <span class="id">I</span> <span class="id">tap</span> <span class="id">login</span>
    <span class="id">Then</span> <span class="id">I</span> <span class="id">should</span> <span class="id">be</span> <span class="id">on</span> <span class="id">the</span> <span class="id">Notes</span> <span class="id">screen</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If you compile your code, you should see the test(s) show up in the test windows.

### Adding steps
Following the Page-Object-Model, we need a static class name to hold our steps. The following code implements the required steps for the example test above.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">module</span> <span class="id">LoginDefinitions</span> <span class="o">=</span> 
    
    <span class="pn">[&lt;</span><span class="id">Given</span><span class="pn">&gt;]</span>
    <span class="k">let</span>  <span class="id">``I enter the username (.*)``</span> <span class="id">username</span> <span class="o">=</span>
        <span class="id">LoginScreen</span><span class="pn">.</span><span class="id">enterTextForUsername</span> <span class="id">username</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span>

    <span class="k">let</span> <span class="pn">[&lt;</span><span class="id">Given</span><span class="pn">&gt;]</span> <span class="id">``I enter the password (.*)``</span> <span class="id">password</span> <span class="o">=</span>
        <span class="id">LoginScreen</span><span class="pn">.</span><span class="id">enterTextForPassword</span> <span class="id">password</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span>

    <span class="k">let</span> <span class="pn">[&lt;</span><span class="id">When</span><span class="pn">&gt;]</span> <span class="id">``I tap login``</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">LoginScreen</span><span class="pn">.</span><span class="id">login</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span>

    <span class="k">let</span> <span class="pn">[&lt;</span><span class="id">Then</span><span class="pn">&gt;]</span> <span class="id">``I should be on the Notes screen``</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">NotesScreen</span><span class="pn">.</span><span class="id">canSeeGetNotesButton</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span>

    <span class="k">let</span> <span class="pn">[&lt;</span><span class="id">Then</span><span class="pn">&gt;]</span> <span class="id">``I am still on the Login page``</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="id">LoginScreen</span><span class="pn">.</span><span class="id">canSeeLoginButton</span> <span class="id">AppInitializer</span><span class="pn">.</span><span class="id">app</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Note the in F# we can use the double backticks to write the name of a function with spaces. This makes writing the test names really easy and avoids naming wars - PascalCase, CamelCase, SnakeCase - just use English now!

<code>[&lt;given&gt;]</code> is an attribute (<code>[Given]</code> in C#). The first test shows how attributes are used in a C# style fashion. The remaining tests use the inlined style, to make the steps extremely readable.

Each step delegates the work to a page and passes in the <code>app</code>. Here is the <code>NotesScreen</code> as an example:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">module</span> <span class="id">NotesScreen</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">getNotesButton</span> <span class="o">=</span> <span class="s">"Get Notes"</span>

    <span class="k">let</span> <span class="id">canSeeGetNotesButton</span> <span class="pn">(</span><span class="id">app</span><span class="pn">:</span><span class="id">IApp</span><span class="pn">)</span> <span class="o">=</span> 
        <span class="id">app</span><span class="pn">.</span><span class="id">WaitForElement</span><span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span><span class="pn">.</span><span class="id">Marked</span> <span class="id">getNotesButton</span><span class="pn">)</span> 
        <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">filter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">x</span><span class="pn">.</span><span class="id">Text</span> <span class="o">=</span> <span class="id">getNotesButton</span><span class="pn">)</span>
        <span class="o">|&gt;</span> <span class="k">function</span>
        <span class="pn">|</span> <span class="pn">[|</span><span class="id">x</span><span class="pn">|]</span> <span class="k">-&gt;</span> <span class="pn">(</span><span class="pn">)</span>
        <span class="pn">|</span> <span class="id">xs</span> <span class="k">-&gt;</span> 
            <span class="id">xs</span> <span class="o">|&gt;</span> <span class="id">Array</span><span class="pn">.</span><span class="id">iter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-&gt;</span> <span class="id">sprintf</span> <span class="s">"%A %s"</span> <span class="id">x</span><span class="pn">.</span><span class="id">Class</span> <span class="id">x</span><span class="pn">.</span><span class="id">Text</span> <span class="o">|&gt;</span> <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">)</span>
            <span class="id">sprintf</span> <span class="s">"failed to find button: %s, %A"</span> <span class="id">getNotesButton</span> <span class="id">xs</span> <span class="o">|&gt;</span> <span class="id">failwith</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
<code>NotesScreen</code> is just a static class (<code>module</code> in F#). We can then define each of the text using <code>let</code>. No need for extra keywords as <code>let</code> bindings are immutable by default.

A simple function <code>canSeeGetNotesButton</code> then does the work to read the app's UI and check that the item exists. If it is missing, an exception is thrown.

### Build and run
With those items in place, you should now be able to build and run. For each tests, there should be an Android and iOS test.
<h1>The following gists show full examples of each file:</h1>
<a href="https://gist.github.com/willsam100/53375a4b94b689f3feb8f989414a9ff8">AppInitializer.fs</a>

<a href="https://gist.github.com/willsam100/91ffd1d74db9f72e6ee9772b098cc46d">Tests.fs</a>

<a href="https://gist.github.com/willsam100/86987eb493da326cdb84a8549ad0b900">Login.feature</a>

## Help - when I build the tests don't show up

### Looking for the wrong file type

This can be caused by a few things. At the start of this blog post, the code to setup things up was added. The end of that code had a the following lines:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">assembly</span><span class="pn">.</span><span class="id">GetManifestResourceNames</span><span class="pn">(</span><span class="pn">)</span>
<span class="o">|&gt;</span> <span class="id">Seq</span><span class="pn">.</span><span class="id">filter</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="id">n</span><span class="pn">:</span><span class="id">string</span><span class="pn">)</span> <span class="k">-&gt;</span> <span class="id">n</span><span class="pn">.</span><span class="id">EndsWith</span><span class="pn">(</span><span class="s">".feature"</span><span class="pn">)</span> <span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Make sure that it still says <code>.feature</code> or it won't find the right file.


### Wrong file type name

The plain test file must end with <code>.feature</code> if it doesn't, then it won't be found


### Build action must be EmbeddedResource
This must be set, as TickSpec scans the assembly looking for feature files (rather than using magic to generate backings files). If not set, then the feature fiel wont' be in the assembly, and TickSpec won't be able to find it.


### Tag each test with @android_ios or @android or @ios
Each test is named with <code>Scenario</code>. Before that line, there must be a tag with what platform the test will run on. The code added at the top of this post describes how to generate two tests (one for each platform) from the single test. No tag, no tests


### Tag each test with @ignore
If you have this tag before your test, TickSpec will ignore the test and not run it.

## Have any questions?
Leave a comment or message me on twitter @willsam100 - I'm happy to help
Happy [type safe] coding
CodingWithSam
