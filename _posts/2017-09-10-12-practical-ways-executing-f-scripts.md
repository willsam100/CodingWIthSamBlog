---
layout: post
title: 12 practical ways for executing F# scripts
date: 2017-09-10 18:35:57.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Tools]
tags: [F#, Script-Files]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1591261501'
  _yoast_wpseo_focuskw_text_input: executing F# scripts
  _yoast_wpseo_focuskw: executing F# scripts
  _yoast_wpseo_metadesc: With all platforms considered, this post shows the 12 ways
    to execute an F# (fsx) script including from a text editor, IDEs or even a Terminal
    window
  _yoast_wpseo_linkdex: '75'
  _yoast_wpseo_content_score: '60'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '46'
  ampforwp_custom_content_editor: ''
  ampforwp_custom_content_editor_checkbox: ''
  ampforwp-amp-on-off: default
  mashsb_shares: '3'
  mashsb_jsonshares: '{"total":3,"error":"","facebook_shares":2,"twitter":1,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/12-practical-ways-executing-f-scripts/
  dsq_thread_id: '6133572967'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2017/09/10/12-practical-ways-executing-f-scripts/"
---
There are many different ways to run an F# script (.fsx) file. All of them are covered here! For F#ers on Windows here is the official documentation <a href="https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/fsharp-interactive/," title="FSharp Interactive">MSDN: FSharp Interactive</a>.
Leave a comment with your favorite!

## Text Editors
The following steps apply to the excellent plugin Ionide for VS Code or Atom. If Ionide is not installed on your machine, here's some help [F#: Getting started with IDEs and Text Editors]({{ "/2016/07/05/f-ides-text-editors-and-how-to-get-started/" | relative_url }})
1: Sending the entire file: CMD-SHIFT-P -> <code>FSI: Send file</code>

- The entire file will be loaded and executed.
- Reliability is guaranteed as the whole script is reloaded.
- May not be the fastest; no shortcut

2: By selection (highlight a block of code first): CMD-SHIT-P: <code>FSI: send selection</code> Note the shortcut

- Will compile and execute the selection
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.

3: By line (Put cursor on desired line first): CMD-SHIT-P: <code>FSI: send line</code>

- Will compile and execute the line
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.


## IDEs: Visual Studio For Mac
4: Sending the entire file: Right click -> <code>Send current file to F# Interactive</code>

- The entire file will be loaded and executed.
- Reliability is guaranteed as the whole script is reloaded.
- May not be the fastest; no shortcut

UPDATE:

Thanks to <a href="https://twitter.com/JasonImison" title="Twitter: @JasonImison">Jason Imison</a> for pointing out that it is possible to set a shortcut.

- Open <code>Preferences</code>
- On the left select <code>Key Bindings</code>
- Filter using the search box <code>send current file</code>
- Select the item, and set a shortcut in the text box
- <code>Apply</code> ->  <code>Ok</code>

5: By selection (highlight a block of code first): Right Click -> 'Send selection to F# Interactive'

- Will compile and execute the selection
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.

6: By line (Put cursor on desired line first): Right Click -> <code>Send line to F# Interactive</code>

- Will compile and execute the line
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.


## IDEs: Visual Studio For Windows
7: By Selection: Right Click -> Execute in Interactive (shortcut is <code>Alt + Enter</code>)

- Will compile and execute the selection
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.

8: By line (Put cursor on desired line first): Shortcut: <code>Alt + '</code>.

- Will compile and execute the line
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.

If the shortcuts are not correct, <a href="http://brandewinder.com/2016/02/06/10-fsharp-scripting-tips/" title="10 fsharp scripting tips">Mathias has a discovered a quick solution</a>

## Terminal
open a terminal window. The command to start an F# REPL is <code>fsharpi</code>
9: Copy and Paste: <code>fsharpi</code> ->  paster in code -> type <code>;;</code> -> press enter/return

- Will compile and execute the selection
- One of the fastest methods with the shortcut
- Can be error prone, since the F#er must remember to send earlier code segments
- Type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage cannot be resolved.
- Parent functions must be reloaded if a called function is updated.

10: Load entire file. <code>fsharpi --load:Script.fsx</code> Script.fsx must be in the current directory

- The entire file will be loaded into a namespace of the filename eg Script
- Reliability is guaranteed as the whole script is reloaded.
- Requires compiling the entire file each time
- Useful if execution of the script needs to be controlled/delayed.

11: Load and run entire file. <code>fsharpi --use:Script.fsx</code> Script.fsx must be in the current directory

- The entire file will be loaded, namespace opened and executed
- Reliability is guaranteed as the whole script is reloaded.
- Requires compiling the entire file each time

UPDATE:
Thanks to <a href="https://twitter.com/JasonImison" title="Twitter: @JasonImison">Jason Imison</a> for point out the following:
13: Shebang:

- Add the following to the top of the script file: <code>#!/usr/bin/env fsharpi --exec
- Make the file executable:</code>chmod +x Script.fsx
- Execute: ./Script.fsxa
- Only works on Mac/Linux
- The entire file will be loaded, namespace opened and executed
- Reliability is guaranteed as the whole script is reloaded.
- Requires compiling the entire file each time


## Rider: For the sake of completeness
12: By Selection: Shortcut <code>option + enter</code>

- Will compile and execute the selection
- One of the fastest methods with the shortcut
- can be error prone, since the F#er must remember to send earlier code segments
- type inference can be negatively affected. Eg a function with statically resolved type parameter sent to REPL without usage.
- Parent functions must be reloaded if a called function is updated.

If a method is missing leave a comment!
