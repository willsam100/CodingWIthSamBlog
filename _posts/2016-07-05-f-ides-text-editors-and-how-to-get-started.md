---
layout: post
title: 'F#: IDEs, text editors and how to get started'
date: 2016-07-05 18:22:07.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
# categories: []
# tags:
# - F#
# - REPL
# - Scripting
# - Xamarin Studio
meta:
  _edit_last: '1'
  mashsb_timestamp: '1590183609'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/f-ides-text-editors-and-how-to-get-started/
  dsq_thread_id: '6133548469'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2016/07/05/f-ides-text-editors-and-how-to-get-started/"
---
Finding the right tool, IDE or text editor can be a tough task when starting with a new language. What’s more, every developer has an opinion, sometimes appearing more like a religion than a conscious choice. I’ll skip the <a href="https://en.wikipedia.org/wiki/Editor_war">editor wars</a> and cut to the chase of the my experience with some current F# editors at the time of writing. With my preferred IDE, Xamarin Studio, I’ll show how to start writing code beyond a print line statement, and also how to start learning the language.

## Installation of F#
F# can be installed several ways, but I’m on a Mac so i’ll be focusing that. My best recommendation for installing F# is with <a href="https://www.xamarin.com">Xamarin Studio</a>. There is now a free version so you don’t have to drop a lot of coin just to get your feet wet with F# development and/or Mobile. Additionally Xamarin is doing a lot of work with F# and their <a href="https://developer.xamarin.com/releases/studio/xamarin.studio_6.0/xamarin.studio_6.0/#Roslyn_integration">latest release</a> shows a number of improvements, so it will continue to get better over time. However, If you do want to pursue other ways (as with the text editors), check out fsharp.org specifically <a href="http://fsharp.org/use/mac/">http://fsharp.org/use/mac/</a> (they have missed out Atom, but the steps are simillar to Visusal Studio Code). 

## Text Editors
As already stated, Xamarin Studio (and IDE) is my preferred choice, but i’ll cover two other text editors, and how to set them with <a href="http://ionide.io">Ionide</a> (a plugin) that gives the editors IDE like attributes. First up is Microsoft’s <a href="https://code.visualstudio.com">Visual Studio Code</a>. The link above already has some instructions for getting it up and running. This was my first choice and I did attempt to use if for a little bit. The second editor is of course <a href="https://atom.io">Atom</a>, a great editor by Github. The steps to install Atom with ionide plugin support are similar to Visual Studio Code. Atom, with Inonide, is able to handle projects thanks to additionally support from Inoide which Visual Studio Code currently does support. This a big advantage since using a project is a much better approach even when attempting small/beginner tasks. Unfortunately for me, I wasn’t able to use these editors for very long before something broke with (what i assume) to be inonide. At the time of the writing these is an issue open with Inonide for the crash that i was experiencing. I moved to Xamarin Studio because of this, but also found that it provided a lot of further benefits that the text editors did not have. I suspect these are issues are local to me, so if you’re a diehard text editor fan, go ahead and get it set it, and don’t forget to leave me a comment about it.
Update: I have Visual Studio Code now working with Ionide. Ionide has pushed an update that gives significantly better error messages then Xamarin Studio, and also gives nice warnings and tips. With Visual Studio now using [Projekt](http://fsprojects.github.io/Projekt/) this is also a productive way for writing F# (I have replaced my default text editor to Visual Studio Code, sorry Sublime)

## Writing code 
Moving on from text editors and IDEs, once you have something installed you're going to want to run some code. Having learnt Python and Haskell using the REPL (Read-Evaluate-Print-Loop), I thought using the REPL directly was going to be the best approach. Unfortunately this is not the best case for a couple of reasons:  

- Auto-complete is your best friend, his power is limited in REPL
- F# script files allow you to send lines of code so you can test it in the REPL
- Python has limited types so is great for scripting
- Haskell has no official IDE so you’re not going to a great auto-complete experience anywhere
- An F# script file means you won’t loose you work after hitting CTRL-C a few too many times. 

After trying out the REPL, I found that it doesn’t scale very well when you write to write something more than a print line statement. F# Script files avoid this problem while still giving you the flexibility by sending lines to it for evaluation. Xamarin Studio (and the editors) are able to send lines of code from the script file to the REPL (Read-Evaluate-Print-Loop). Now that we’ve covered how to work, it’s almost time to create that hello world script and send it to the REPL, but before you do there is one last thing cover. 
F# script files are really great, and to make them even better (and you more productive) is to put them in a project so you can add libraries. The reason for this is referencing the nuget packages in a script files requires a reference to the DLL. Creating a project makes this a lot easier and it’s trivial to find the DLL once installed. Additionally if you’re using Xamarin Studio then installing packages from nuget is also a lot simpler (Did I mention that I prefer Xamarin Studio over a text editor?). You must be itching to write some code now so let’s get to it!
With Xamarin Studio installed, let’s create a script file as part of a project. <b>Open Xamarin Studi</b>o, from the File menu: <b>New -&gt; Solution</b>. In the pop on the left<b> select .NET</b> and on the right<b> select F# Tutorial. </b>Give it a name and click <b>create</b>. This has created a project with a script file appropriately named script.fsx. Additionally (since we chose the Tutorial template) the file has a lot of language examples. We need to get our hello world output done, so select all the text delete it, you can undo the changes we’re making to get it back or create a new project. With a blank file the code is:
Printfn “Hello World”
Select the line and then CRTL-Enter or right click and select ‘Send selection to F# Interactive’
&nbsp;
So you got the necessary hello world out of the way? The last item that I have not covered is adding packages via nuget. In Xamarin Studio this is very easy, and not too difficult in a script file. Let's add <a href="http://fsharp.github.io/FSharp.Data/">Fsharp.Data</a> a great library for accessing data. (C# devs, add this as you normally would)  <b>Right click on the project -&gt; Add -&gt; Nuget package</b>. A dialog will popup, in the search box type Fsharp.Data, then click add. Once you have this installed, you need to reference the dll from the script file. Because we’re using Xamarin, it is easier to know where these packages were installed, add the following lines to the top of the script file:

```fsharp
#r "../packages/FSharp.Data.2.3.0/lib/net40/FSharp.Data.dll"
open FSharp.Data
```

The version number in the library might change so you may need to check folder in Finder. Keeping track and finding these packages and dlls is a little harder with text editor. 

That’s it for this post, start having fun with Xamarin Studio, script files and the language tutorial. Stay tuned for next post, as we dive deeper into F#.
