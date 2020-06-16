---
layout: post
title: ".NET Standard for mobile is here!"
date: 2017-09-01 00:44:26.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Mobile]
tags: [F#, Xamarin, XamarinForms]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1580735619'
  mashsb_shares: '6'
  mashsb_jsonshares: '{"total":6,"error":"","facebook_shares":5,"twitter":1,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/net-standard-for-mobile-is-here/
  dsq_thread_id: '6132466157'
  _yoast_wpseo_content_score: '60'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: ''
  ampforwp_custom_content_editor: ''
  ampforwp_custom_content_editor_checkbox: ''
  ampforwp-amp-on-off: default
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2017/09/01/net-standard-for-mobile-is-here/"
---

## Background: I came from Java

Coming from a Java background, one of the hardest things I struggled with was understanding PCLs (Portable Class Libraries). It took a long time to figure out what the right version was, yea that magic number :(. Finding out if a Nuget package was supported in my PCL was also difficult. And then there was the case when a Nuget package could be supported, but the package author has not set things up correctly (generally because they were thinking of the desktop, but I wanted to code on the mobile). It turns out that I was not alone in this and the solution is here!

## What's the problem

Consider building a mobile app for both Android and iOS. Some of the app will be exactly the same, like talking to a REST server. Sharing that code (rather than writing it twice) is what PCLs and .NET Standard is all about. This applies to much more than mobile apps, but all places where code sharing is relevant, including libraries.

PCLs work by using magic numbers that target the intersection of APIs for a given set of devices. Yep, thats sounds confusing, I'm with you. So what is the alternative: .NET Standard!

PCLs and .Net Standard both talk about a platform. This refers to the 'thing' that will be running the code. A platform could be a desktop computer running Windows 10, another platform might be a laptop running Windows 8. It could be even be a chip running some IoT code. A platform is any chip that can run .NET code. A mobile phone can do more things than an IoT chip, and and desktop computer with Windows 10 can do more things than a mobile phone. It's a sliding scale.

## The Solution

.NET Standard uses sensible numbers. The higher the number the more APIs that can be accessed. .NET Standard starts with the most basic APIs at 1.0 (.NET Standard 1.0) and as the numbers increase, more APIs are available. As a result the project is less portable (since the platform must support the capabilities). .NET Standard has been out for a while (approx a year as of this writing), but that first started at .NET Standard 1.0. For those who want to build mobile apps, there are a few more APIs that are needed. Accessing the file system and network being the two of the most important things for a mobile device.

In Visual Studio for Mac (on the alpha channel), it is now possible to create a .NET Standard package that targets .NET Standard 2.0. The API surface for 2.0 is a big jump, and doubles the number of APIs available from 1.6 (the next level down). This means we can code lots of great things, with much less pain. So let's create an app to do that. The current templates for creating a Xamarin Forms app do not support it yet, so here are the steps broken down:

## Trying it out

Open VS4Mac and create a new project:
<img src="{{ site.baseurl }}/assets/img/NewApp.png" alt="&quot;Create new app&quot;" title="" />

Note that I'm using F# because I love it! C# can be used too
Give the app a name, I'm going with <code>MyFirstNetStandard</code>

<img src="{{ site.baseurl }}/assets/img/NewAppName.png" alt="&quot;New App Name&quot;" title="" />
As per normal, this project will have a PCL for the shared code. Right-click and delete project. Your project structure should now look as follows:
<img src="{{ site.baseurl }}/assets/img/ProjectStructure.png" alt="&quot;Project Structure&quot;" title="" />
It's now time for the exciting part; add the .NET Standard project. Right click on the solution folder, and select <code>Add New Project</code>
<img src="{{ site.baseurl }}/assets/img/AddNewProject.png" alt="&quot;Add New Project&quot;" title="" />
From the side menu on the left, select <code>Library</code> under the heading .<code>NET Core</code>, and then select <code>.NET Standard Library</code>, Again I'm using selecting F# as my language because it is fantastic, but C# will work. Complete the prompts, ensuring that <code>.NET Standard 2.0</code> is selected for the target platform
<img src="{{ site.baseurl }}/assets/img/AddNetStandard.png" alt="&quot;Add Net Standard&quot;" title="" />

<img src="{{ site.baseurl }}/assets/img/TargetPlatform.png" alt="&quot;Target Platform&quot;" title="" />
And give it name, I am using the same name as the PCL <code>MyFirstNetStandard</code>

<img src="{{ site.baseurl }}/assets/img/NewAppComplete.png" alt="&quot;New App Complete&quot;" title="" />
Excellent, the new project should not appear in the solution explorer along with the Droid and iOS projects. Next we need to add the <code>Xamarin.Forms</code> nuget package the .Net Standard project. Expand the project structure, right click on <code>Dependencies</code> and select `Add Packages`, then follow the prompts as with any nuget package addition.
<img src="{{ site.baseurl }}/assets/img/AddXamarinForms.png" alt="&quot;Add Xamarin Forms&quot;" title="" />
The final step to writing this up is updating the Droid and iOS projects with a reference to the .NET Standard package. The following steps will need to be repeated for both platforms. Expand the platform, right click on 'Reference` and select 'Edit References...'
<img src="{{ site.baseurl }}/assets/img/EditReferences.png" alt="&quot;Edit References&quot;" title="" />
In the dialog that will display, check the box for the shiny new .NET Standard project:
<img src="{{ site.baseurl }}/assets/img/PlatformAddReference.png" alt="&quot;Platform Add Reference&quot;" title="" />
Everything is all wired up now, but there is no code in the .NET Standard project. In the .NET Standard project, <code>MyFirstNetStandard</code> open the main file and add the required code to create and app. My examples will be in F#, but are easy enough to follow along if you haven't used the language yet.
Open 'Library.fs', and clear out anything in th file then add the namespace and open statement for Xamarin Forms
<table class="pre">
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">namespace</span> <span class="id">MyFirstNetStandard</span>
<span class="k">open</span> <span class="id">Xamarin</span><span class="pn">.</span><span class="id">Forms</span>
</code></pre>
</td>
</tr>
</table>
Below that, add a page with a label:
<table class="pre">
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">MainPage</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
<span class="k">inherit</span> <span class="id">ContentPage</span><span class="pn">(</span><span class="pn">)</span>

<span class="k">let</span> <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="id">label</span> <span class="o">=</span> <span class="id">Label</span><span class="pn">(</span><span class="id">Text</span> <span class="o">=</span> <span class="s">&quot;.NET Standard is the solution to all our problems!&quot;</span><span class="pn">,</span> 
                  <span class="id">VerticalTextAlignment</span> <span class="o">=</span> <span class="id">TextAlignment</span><span class="pn">.</span><span class="id">Center</span><span class="pn">)</span>
<span class="k">do</span> <span class="k">base</span><span class="pn">.</span><span class="id">Content</span> <span class="k">&lt;-</span> <span onmouseout="hideTip(event, 'fs1', 2)" onmouseover="showTip(event, 'fs1', 2)" class="id">label</span>
</code></pre>
</td>
</tr>
</table>
And finally, below the new page, create the application:
<table class="pre">
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span onmouseout="hideTip(event, 'fs2', 3)" onmouseover="showTip(event, 'fs2', 3)" class="rt">App</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
<span class="k">inherit</span> <span class="id">Application</span><span class="pn">(</span><span class="id">MainPage</span> <span class="o">=</span> <span class="id">MainPage</span><span class="pn">(</span><span class="pn">)</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</table>
Everything should be working now, build and run you app on both Android and iOS and enjoy
<img src="{{ site.baseurl }}/assets/img/Solution.png" alt="&quot;Solution&quot;" title="" />
For more details, and those on windows, here is the <a href="https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-standard-2-0/">MSDN announcement</a>
Happy Coding!