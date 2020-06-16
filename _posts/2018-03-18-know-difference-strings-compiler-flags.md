---
layout: post
title: Do you know the difference between strings and compiler symbols?
date: 2018-03-18 11:18:20.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Mobile]
tags: [Android ,release ,testing]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1586895424'
  _yoast_wpseo_focuskw_text_input: Strings or Compiler symbols
  _yoast_wpseo_focuskw: Strings or Compiler symbols
  _yoast_wpseo_linkdex: '50'
  _yoast_wpseo_content_score: '90'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '33'
  mashsb_shares: '2'
  mashsb_jsonshares: '{"total":2,"error":"","facebook_shares":0,"twitter":2,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/know-difference-strings-compiler-flags/
  dsq_thread_id: '6558057292'
  _yoast_wpseo_metadesc: Learn how to use the strings.xml file in Android correctly
    and how to handle strings in code that should not be used in production. Code
    with confidence using compiler symbols.
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2018/03/18/know-difference-strings-compiler-flags/"
---
## Reverse Engineering APKs
Lately I have been working on reverse engineering an app. Along the way I came across something interesting that I had to share.
I decompiled the APK from the Play Store (very easy to do), and went looking around the files. In the decompiled APK, I found many problems (which was why I was decompiling it in the first place). In the <code>strings.xml</code> file, was something very surprising to me; to the point that it inspired this blog post.
The urls for the app were all stored in the <code>strings.xml</code> file! This included the dev and staging environments, of which there were a few. This is not the best practise.

## What is the <code>strings.xml</code> file really for?
The purpose of the <code>strings.xml</code> file for Android (Xamarin Forms uses the <code>.Resx</code> file extension; the concept is the same) is only for text that will appear on the UI of the app. By doing this it allows for internationalization (allowing the app to support multiple languages).

Adding additional information here has a few downsides. Data for a DEV build should not be in a production; if only to prove that the user can never see it. Consider the url issue. If the DEV url is not present in the release version of the app, then it is impossible for a developer to make a coding error where the app uses the DEV url. That also means the developer has less testing to do, since it is is not possible to write a unit test for an impossible scenario.

Security is also improved by only putting UI text in the <code>strings.xml</code> file. An attacker would have less information to find vulnerabilities in the system.
Putting code (strings are code) in the correct place also makes the code clearer for other developers. It makes the intent obvious that text in the <code>strings.xml</code> is for the UI (and careful attention to the language should be given as the text is for an end user), while strings should be in the code. Strings are for developers and the developer should write strings (like any piece of code) with other developers in mind.

## Using compiler symbols
The solution is in fact rather easy - use compiler symbols. As already mentioned, it will provide many benefits in helping that the release APK works as intended.
Compiler symbols, allow a developer to instruct the compiler which sections of the code to include or omit. When a developer complies the app, the symbols are passed to the compiler that controls the behaviour (the symbols are generally controlled by the configuration of the IDE eg DEBUG/RELEASE).
An example is easy to understand:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pp">#if</span> <span class="id">DEBUG</span>
<span class="inactive">    </span><span class="inactive">string</span><span class="inactive"> </span><span class="inactive">const</span><span class="inactive"> </span><span class="inactive">url</span><span class="inactive"> </span><span class="inactive">=</span><span class="inactive"> </span><span class="inactive">&quot;https://dev.mycompany.com&quot;</span>
<span class="pp">#else</span>    
    <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="fn">string</span> <span class="k">const</span> <span class="id">url</span> <span class="o">=</span> <span class="s">&quot;https://prod.mycompany.com&quot;</span>
<span class="pp">#endif</span>
</code></pre>
</td>
</tr>
</table>
The statements are rather intuitive. For the line <code>#if DEBUG</code>, if the symbol <code>DEBUG</code> is defined, the compiler will include the nested code. Otherwise the compiler will include the <code>#else</code> section of the code. Compiler statements do not need to contain an <code>#else</code> block.

When using any IDE (Visual Studio or Android Studio) there are options to control the configuration ie what complier symbols are present. Be default the configuration <code>DEBUG</code> is selected and as a result the <code>DEBUG</code> compiler symbol will be added when compiling. For the section of the code above, the app that is compiled will have the url <code>"https://dev.mycompany.com"</code>. If <code>RELEASE</code> was selected as the configuration, then the <code>DEBUG</code> symbol would not be present. And as you guessed, the resulting app would have a url of <code>"https://prod.mycompany.com"</code>

If you have more than one dev,test,staging environment, add more code in the <code>#if DEBUG</code> section, and ensure that when the <code>RELEASE</code> configuration is selected, there is no dev/test code in the APK. Following this pattern will help reduce any silly mistakes with a production version of the app, make the code clear for other developers to understand, and reduce the APK size.
Until next time



Happy coding