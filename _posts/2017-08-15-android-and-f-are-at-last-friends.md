---
layout: post
title: Android and F# are at last friends
date: 2017-08-15 18:37:10.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Mobile]
tags: [Android, Xamarin]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1590872847'
  mashsb_shares: '3'
  mashsb_jsonshares: '{"total":3,"error":"","facebook_shares":2,"twitter":1,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/android-and-f-are-at-last-friends/
  dsq_thread_id: '6133782183'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '33'
  _yoast_wpseo_content_score: '30'
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2017/08/15/android-and-f-are-at-last-friends/"
---
A long time ago.., ok sometime at around the end of last year (2016), Android's terrible relationship with F# was about to improve significantly. Xamarin Forms was possible with an F# Core, but many, myself included, resorted to using a C# Droid project.

What was the problem you ask? every time the project is recompiled, or the R file is regenerated, the resource IDs are recalculated. One would assume that an F# project should have F# an F# resource file, but it turns out that some of the names were reserved keywords in F#. Causing the project, not the build. The F# resource file had to be edited manually to put quotes around the reserved keywords to get the project to compile. Everything was fine until that file changed or a recompile was required again.
**1000 attempts were not needed**

The team at Xamarin kept working away at this bug with many attempts, but a bolder move was required. The final solution was to use one of F#'s superpowers, yes, a type provider. The generated resource file was left as a C# file, no more pesky keywords. A Resources type provider was then created to read off the resources file. This addressed all the problems making Android development in F# a pleasant experience. For those not familiar with a type provider, it is like a compiler plugin to the F# compiler,

I wrote an intro here [learn about type providers]({{ "/2016/07/18/yet-another-type-provider-blog-and-other-cool-things/" | relative_url }})
**How do I use it**

In your favourite IDE, it should be Visual Studio since it's now on Mac too, the default templates have been updated to include the resources type provider. To access the resources though, you will need to add one line. The following should be added at the top of your fs file, or before all your activities/fragments
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// assuming the droid namespace is MyGreatAndroidProject.Droid</span>
<span class="k">type</span> <span class="rt">Resources</span> <span class="o">=</span> <span class="id">MyGreatAndroidProject</span><span class="pn">.</span><span class="id">Droid</span><span class="pn">.</span><span class="id">Resource</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
With that line added, it's now possible to access your resources. For example, to get the main page would be
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">Resources</span><span class="pn">.</span><span class="id">Layout</span><span class="pn">.</span><span class="id">Main</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If for some reason the resources are not appearing, then restart the IDE as it is most likely that the file was not added to the solution correctly (an error with the IDE).
Go forth with Android and F# in harmony - for example, you could convert a Xamarin Forms project to 100% F#. Here is how to set up the core project with [MvvmCross and FSharp]({{ "2017/05/22/xamarin-forms-mvvmcross-fsharp/" | relative_url }})
Oh and here's the original bug report for those who want a full read up: <a href="https://bugzilla.xamarin.com/show_bug.cgi?id=24709">F# reserved keyword 'end' being generated in member for resource.designer.fs</a>
