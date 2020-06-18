---
layout: post
title: How to mvvmCross with F# that will increase your productivity
date: 2017-05-22 19:53:37.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Xamarin, XamarinForms, Xaml]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1589841226'
  mashsb_shares: '3'
  mashsb_jsonshares: '{"total":3,"error":"","facebook_shares":3,"twitter":0,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/xamarin-forms-mvvmcross-fsharp/
  dsq_thread_id: '6132484987'
  _yoast_wpseo_focuskw_text_input: mvvmcross fsharp
  _yoast_wpseo_focuskw: mvvmcross fsharp
  _yoast_wpseo_title: "%%title%% %%sep%% %%category%%"

  _yoast_wpseo_linkdex: '74'
  _yoast_wpseo_content_score: '60'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '29'
  _wp_old_slug: xamarin-forms-mvvmcross-with-f
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
description: MvvmCross is a great framework for MVVM. Learn how to use
MvvmCross with F#/FSharp and avoid a couple of nasty bugs! Tips and tricks included
permalink: "/2017/05/22/xamarin-forms-mvvmcross-fsharp/"
---

## prerequisites

- Understanding of C#, Xamarin, Mvvm/MvvmCross
F# supports Object-Orientated programming. MvvmCross is a framework for building apps with an MVVM design pattern. This blog post walks through combing them using a PCL ie MvvmCross with F#. A new project will be created. A new F# core PCL will then be added. The C# core will then be translated to F#. A small problem awaits....

## Getting setup
Follow the IDE template and get a MvvmCross project setup with a C# PCl. We will change the core once this has been added. If you haven't added a template you can add it to your IDE from <a title="https://github.com/jimbobbennett/MVVMCross.XSAddIn">Xamarin Studio</a> or <a title="https://marketplace.visualstudio.com/items?itemName=JimBobBennett.MvvmCrossforVisualStudio">Visual Studio</a>. Once the project is created and all the packages have been installed, run your app to double check everything installed correctly. I've named my app <code>MvvmCrossViewModelDemo</code>

## Adding F# core
Next, we will add an F# core to replace the C# Core. On the solution right click and add a Forms PCL, make sure to choose F# for the language drop down. For the name you can choose anything, this post will use 'central'. Once the F# PCL has been added, do the following:

- remove windows 8.1 support from the PCL, ie select '.NET Portable Subset (.NET Framework 4.5, Windows 8)'
-- This is showing as profile7 on my machine
- add the required packages via Nuget (or Paket if you know how)
-- MvvmCross
-- MvvmCross.Binding
-- MvvmCross.Core
-- MvvmCross.Platform
-- Xamarin.Forms

## Porting MvxApplication
Rename the file <code>MyPage.fs</code> to <code>App.fs</code>
The <code>App.cs</code> is the first block of code to port. It appears as follows:
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
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">public</span> <span class="k">class</span> App <span class="o">:</span> MvvmCross.Core.ViewModels.MvxApplication
{
  <span class="k">public</span> <span class="k">override</span> <span class="k">void</span> Initialize()
  {
    CreatableTypes()
      .EndingWith(<span class="s">"Service"</span>)
      .AsInterfaces()
      .RegisterAsLazySingleton();

    RegisterAppStart&lt;ViewModels.FirstViewModel&gt;();
  }
}
</code></pre>
</td>
</tr>
</tbody>
</table>
This is a direct port so the F# looks as follows:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// F# port of MvvmCross MvxApplication</span>
<span class="k">type</span> <span class="id">App</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">inherit</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Core</span><span class="pn">.</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">MvxApplication</span><span class="pn">(</span><span class="pn">)</span> 
    
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">Initialize</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
        <span class="id">this</span><span class="pn">.</span><span class="id">CreatableTypes</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">EndingWith</span><span class="pn">(</span><span class="s">"Service"</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">AsInterfaces</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">RegisterAsLazySingleton</span><span class="pn">(</span><span class="pn">)</span>
        <span class="id">this</span><span class="pn">.</span><span class="id">RegisterAppStart</span><span class="pn">&lt;</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">FirstViewModel</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Update the <code>App.fs</code> file to be as follows:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">namespace</span> <span class="id">MvvmCrossViewModelDemo</span><span class="pn">.</span><span class="id">Central</span>
<span class="k">open</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Platform</span><span class="pn">.</span><span class="id">IoC</span>

<span class="k">type</span> <span class="id">App</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">inherit</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Core</span><span class="pn">.</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">MvxApplication</span><span class="pn">(</span><span class="pn">)</span> 

    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">Initialize</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    
        <span class="id">this</span><span class="pn">.</span><span class="id">CreatableTypes</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">EndingWith</span><span class="pn">(</span><span class="s">"Service"</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">AsInterfaces</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">RegisterAsLazySingleton</span><span class="pn">(</span><span class="pn">)</span>

        <span class="id">this</span><span class="pn">.</span><span class="id">RegisterAppStart</span><span class="pn">&lt;</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">FirstViewModel</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

##  Adding views
Simply add the views to the F# PCL project as you would with a C# project. Be sure to select <code>Forms ContentPage Xaml</code>

When added, copy the XAML from below (or the C# project) and paste into the XAML for the F# project. Don't forget to update the two-class references (<code>x:Class</code>) for each of the files at the top of the XAML. I've included just the FirstPage.xaml:
FirstPage.xaml
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
<pre class="fssnip highlighted"><code lang="xml"><span class="prep">&lt;?</span>xml version=<span class="s">"1.0"</span> encoding=<span class="s">"utf-8"</span><span class="prep">?&gt;</span>
<span class="k">&lt;</span><span class="i">ContentPage</span> <span class="o">xmlns</span><span class="k">="http://xamarin.com/schemas/2014/forms"</span> <span class="o">xmlns:x</span><span class="k">="http://schemas.microsoft.com/winfx/2009/xaml"</span> <span class="o">xmlns:forms</span><span class="k">="using:Xamarin.Forms"</span> <span class="o">x:Class</span><span class="k">="MvvmCrossViewModelDemo.Central.FirstPage"</span> <span class="o">Title</span><span class="k">="First Page"</span><span class="k">&gt;</span>
    <span class="c">&lt;!-- Jul 22 2015 Xamarin have not yet provided Device.OnPlatform property for W81. The below syntax works by putting "5, 0, 5, 95" into the default--&gt;</span>
    <span class="k">&lt;</span><span class="i">ContentPage.Padding</span> Thickness="5, 0, 5, 95"<span class="k">&gt;</span>
        <span class="k">&lt;</span><span class="i">OnPlatform</span> <span class="o">x:TypeArguments</span><span class="k">="Thickness"</span> <span class="o">iOS</span><span class="k">="5, 20, 5, 0"</span> <span class="o">Android</span><span class="k">="5, 0, 5, 0"</span> <span class="o">WinPhone</span><span class="k">="5, 0, 5, 0"</span> <span class="k">/&gt;</span>
    <span class="k">&lt;/</span><span class="i">ContentPage.Padding</span><span class="k">&gt;</span>
    <span class="k">&lt;</span><span class="i">ContentPage.ToolbarItems</span><span class="k">&gt;</span>
        <span class="k">&lt;</span><span class="i">ToolbarItem</span> <span class="o">Name</span><span class="k">="Menu1"</span> <span class="o">Text</span><span class="k">="About"</span> <span class="o">ClassId</span><span class="k">="About"</span> <span class="o">Order</span><span class="k">="Primary"</span> <span class="o">Command</span><span class="k">="{Binding ShowAboutPageCommand}"</span><span class="k">&gt;</span>
            <span class="k">&lt;</span><span class="i">ToolbarItem.Icon</span><span class="k">&gt;</span>
                <span class="k">&lt;</span><span class="i">OnPlatform</span> <span class="o">x:TypeArguments</span><span class="k">="FileImageSource"</span> <span class="o">WinPhone</span><span class="k">="Toolkit.Content/ApplicationBar.Add.png"</span> <span class="k">/&gt;</span>
            <span class="k">&lt;/</span><span class="i">ToolbarItem.Icon</span><span class="k">&gt;</span>
        <span class="k">&lt;/</span><span class="i">ToolbarItem</span><span class="k">&gt;</span>
    <span class="k">&lt;/</span><span class="i">ContentPage.ToolbarItems</span><span class="k">&gt;</span>
    <span class="k">&lt;</span><span class="i">StackLayout</span> <span class="o">Spacing</span><span class="k">="10"</span> <span class="o">Orientation</span><span class="k">="Vertical"</span><span class="k">&gt;</span>
        <span class="k">&lt;</span><span class="i">Label</span> <span class="o">FontSize</span><span class="k">="24"</span> <span class="o">Text</span><span class="k">="Enter your nickname in the box below"</span> <span class="k">/&gt;</span>
        <span class="k">&lt;</span><span class="i">Entry</span> <span class="o">Placeholder</span><span class="k">="Who are you?"</span> <span class="o">TextColor</span><span class="k">="Red"</span> <span class="o">Text</span><span class="k">="{Binding YourNickname}"</span> <span class="k">/&gt;</span>
        <span class="k">&lt;</span><span class="i">Label</span> <span class="o">FontSize</span><span class="k">="24"</span> <span class="o">Text</span><span class="k">="{Binding Hello}"</span> <span class="k">/&gt;</span>
    <span class="k">&lt;/</span><span class="i">StackLayout</span><span class="k">&gt;</span>
<span class="k">&lt;/</span><span class="i">ContentPage</span><span class="k">&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## Porting the view models
A full version of <code>App.fs</code> with the view models is at the bottom of this blog if you want to skip to the end.

Let's start with the <code>AboutViewModel</code>, paste the following code below in <code>App.fs</code> below <code>open MvvmCross.Platform.IoC</code>:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">open</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Core</span><span class="pn">.</span><span class="id">ViewModels</span>

<span class="k">module</span> <span class="id">ViewModels</span> <span class="o">=</span> 

    <span class="k">type</span> <span class="id">AboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="k">inherit</span> <span class="id">MvxViewModel</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
We need the import <code>MvvmCross.Core.ViewModels</code> in order to subclass,<code>MvxViewModel</code> just as we do in C#. (Note the project won't compile until everything is complete)

## Porting the FirstViewModel
The <code>FirstViewModel</code> requires a bit more work to translate, so is covered bit by bit. <code>FirstViewModel</code> should be placed below <code>AboutViewModel</code> in the <code>ViewModels</code> modules. F# is indentation sensitive so this means it should be indented two tabs.
Create the class with subclass:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">FirstViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>  
    <span class="k">inherit</span> <span class="id">MvxViewModel</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Add the variable to hold the string with its associated property for <code>YourNickName</code> :
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">yourNickname</span> <span class="o">=</span> <span class="id">ref</span> <span class="s">""</span>
<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">YourNickname</span> 
    <span class="k">with</span> <span class="id">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="o">!</span><span class="id">yourNickname</span> 
    <span class="k">and</span> <span class="id">set</span><span class="pn">(</span><span class="id">value</span><span class="pn">)</span> <span class="o">=</span>
        <span class="k">if</span> <span class="pn">(</span><span class="id">this</span><span class="pn">.</span><span class="id">SetProperty</span><span class="pn">(</span><span class="id">yourNickname</span><span class="pn">,</span> <span class="id">value</span><span class="pn">)</span><span class="pn">)</span> <span class="k">then</span> 
            <span class="id">this</span><span class="pn">.</span><span class="id">RaisePropertyChanged</span><span class="pn">(</span><span class="s">"Hello"</span><span class="pn">)</span>
        <span class="k">else</span> 
            <span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
My first attempt at doing this, (and I'm sure most people learning F# would be the same), was to use <code>let mutable yourNickname = ""</code> to hold the mutable string. However it doesn't work, (the code compiles but value/property is never updated). This is because of the implementation of <code>SetProperty</code> in MvvmCross.
The implementation of <code>SetProperty</code> is:
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
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">protected</span> <span class="k">bool</span> SetProperty&lt;T&gt; (<span class="k">ref</span> T storage, T <span class="k">value</span>, [CallerMemberName] <span class="k">string</span> propertyName <span class="o">=</span> <span class="k">null</span>)
{
    <span class="k">if</span> (<span class="k">object</span>.Equals (storage, <span class="k">value</span>)) {
        <span class="k">return</span> <span class="k">false</span>;
    }
    storage <span class="o">=</span> <span class="k">value</span>;
    <span class="k">this</span>.RaisePropertyChanged (propertyName);
    <span class="k">return</span> <span class="k">true</span>;
}
</code></pre>
</td>
</tr>
</tbody>
</table>
Notice that <code>ref</code> that is passed in. In F# if you take a <code>ref</code> of a mutable (ie <code>ref yourNickname</code>) you don't get the same result as taking a ref in C#. In F# it creates a wrapper that holds a mutable value. This is not what we want as we won't be able to compare it with the raw string; it will always return false. For more details see this post <a title="https://davefancher.com/2014/03/24/passing-arguments-by-reference-in-f/">Pass by reference</a>. By using a ref value in the view model, this problem is avoided and all we need to remember is to dereference <code>yourNickname</code> when we want the string.
As a result of using the ref we now have this <code>!</code> in the getter:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">with</span> <span class="id">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="o">!</span><span class="id">yourNickname</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As stated earlier, we need to return a string and we're using a reference type which is a wrapper type. The exclamation mark is a shorthand to return the enclosed mutable field.
Next, we can add the getter the for the string. Again we use <code>!</code> to get the string value out:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Hello</span> <span class="o">=</span> <span class="id">sprintf</span> <span class="s">"Hello %s"</span> <span class="o">!</span><span class="id">yourNickname</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Finally, we can add in the command to navigate to the <code>AboutViewModel</code>:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">member</span> <span class="k">private</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="id">this</span><span class="pn">.</span><span class="id">ShowViewModel</span><span class="pn">&lt;</span><span class="id">AboutViewModel</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>

<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutPageCommand</span> 
    <span class="k">with</span> <span class="id">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="k">new</span> <span class="id">MvxCommand</span><span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
You'll notice that we had to create a private method to make the call to base method <code>ShowViewModel</code>. The reason for this is that <code>MvxCommand</code> command takes in a function, and calling <code>ShowViewModel</code> must be called from a subclass ie it's not public method. The compiler gives an error if a function is passed in that directly calls <code>ShowViewModel</code> as it cannot prove that the call is made from a subclass ie it's enforcing the method is not public. The private method to <code>FirstViewModel</code> proves that the call is made from a subclass, so everything now compiles.
That's it for porting the C# to F#. Note that with F# we don't need lots of files. The view models are declared above the app class, and F# compiler enforces that, making the code easy to navigate. Here is how the entire <code>App.fs</code> should look:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">namespace</span> <span class="id">MvvmCrossViewModelDemo</span><span class="pn">.</span><span class="id">Central</span>

<span class="k">open</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Platform</span><span class="pn">.</span><span class="id">IoC</span>
<span class="k">open</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Core</span><span class="pn">.</span><span class="id">ViewModels</span>

<span class="k">module</span> <span class="id">ViewModels</span> <span class="o">=</span> 

    <span class="k">type</span> <span class="id">AboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
        <span class="k">inherit</span> <span class="id">MvxViewModel</span><span class="pn">(</span><span class="pn">)</span>

    <span class="k">type</span> <span class="id">FirstViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>  
        <span class="k">inherit</span> <span class="id">MvxViewModel</span><span class="pn">(</span><span class="pn">)</span>
    
        <span class="k">let</span> <span class="id">yourNickname</span> <span class="o">=</span> <span class="id">ref</span> <span class="s">""</span>

        <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">YourNickname</span> 
            <span class="k">with</span> <span class="id">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="o">!</span><span class="id">yourNickname</span> 
            <span class="k">and</span> <span class="id">set</span><span class="pn">(</span><span class="id">value</span><span class="pn">)</span> <span class="o">=</span>
                <span class="k">if</span> <span class="pn">(</span><span class="id">this</span><span class="pn">.</span><span class="id">SetProperty</span><span class="pn">(</span><span class="id">yourNickname</span><span class="pn">,</span> <span class="id">value</span><span class="pn">)</span><span class="pn">)</span> <span class="k">then</span> 
                    <span class="id">this</span><span class="pn">.</span><span class="id">RaisePropertyChanged</span><span class="pn">(</span><span class="s">"Hello"</span><span class="pn">)</span>
                <span class="k">else</span> 
                    <span class="pn">(</span><span class="pn">)</span>

        <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">Hello</span> <span class="o">=</span> <span class="id">sprintf</span> <span class="s">"Hello %s"</span> <span class="o">!</span><span class="id">yourNickname</span>

        <span class="k">member</span> <span class="k">private</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
            <span class="id">this</span><span class="pn">.</span><span class="id">ShowViewModel</span><span class="pn">&lt;</span><span class="id">AboutViewModel</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>

        <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutPageCommand</span> 
            <span class="k">with</span> <span class="id">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="k">new</span> <span class="id">MvxCommand</span><span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowAboutViewModel</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="id">ignore</span><span class="pn">)</span>
        
<span class="k">type</span> <span class="id">App</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">inherit</span> <span class="id">MvvmCross</span><span class="pn">.</span><span class="id">Core</span><span class="pn">.</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">MvxApplication</span><span class="pn">(</span><span class="pn">)</span> 

    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">Initialize</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    
        <span class="id">this</span><span class="pn">.</span><span class="id">CreatableTypes</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">EndingWith</span><span class="pn">(</span><span class="s">"Service"</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">AsInterfaces</span><span class="pn">(</span><span class="pn">)</span>
            <span class="pn">.</span><span class="id">RegisterAsLazySingleton</span><span class="pn">(</span><span class="pn">)</span>

        <span class="id">this</span><span class="pn">.</span><span class="id">RegisterAppStart</span><span class="pn">&lt;</span><span class="id">ViewModels</span><span class="pn">.</span><span class="id">FirstViewModel</span><span class="pn">&gt;</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## The last wiring effort
For both the iOS and droid project, remove the project reference to the C# core and change it to the F# core (named Central in the post). Alternatively you could delete the C# core if you want to be sure you are using F#. Compile these projects, and you will get a compile error in the <code>Setup.cs</code> files. Change <code>Core</code> to <code>Central</code> in <code>Setup.cs</code> and everything should compile, ie:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="csharp"><span class="k">return</span> <span class="k">new</span> Central.App();
</code></pre>
</td>
</tr>
</tbody>
</table>

## Wrapping up
This post has shown how to create an F# core with mvvmCross. Remember the following:

- use ref for your variables if you need to call <code>SetProperty</code>, or any other method that takes in a ref value

- use private methods if need to pass a function that invokes a protected method on a base class
As always, leave a comment if you found this helpful or know a better way!
