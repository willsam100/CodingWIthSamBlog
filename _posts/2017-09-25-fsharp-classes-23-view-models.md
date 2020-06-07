---
layout: post
title: Everything you need to know about F# classes in 23 view models
date: 2017-09-25 08:21:57.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Class, GUI, Interfaces, View Model]
meta:
  _edit_last: '1'
  mashsb_timestamp: '1591365388'
  _yoast_wpseo_content_score: '30'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '47'
  _yoast_wpseo_focuskw_text_input: fsharp view models
  mashsb_shares: '4'
  mashsb_jsonshares: '{"total":4,"error":"","twitter":2,"facebook_shares":2,"facebook_total":0,"facebook_likes":0,"facebook_comments":0}'
  mashsb_shorturl: http://www.codingwithsam.com/fsharp-classes-23-view-models/
  _yoast_wpseo_focuskw: fsharp view models
  _yoast_wpseo_title: "%%title%% %%sep%% %%category%%"
  _yoast_wpseo_metadesc: View models make for a great way to learn the class features
    of F#. Here are 23 useless view models so you can build many more useful F# view
    models!
  _yoast_wpseo_linkdex: '68'
  dsq_thread_id: '6168821796'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2017/09/25/fsharp-classes-23-view-models/"
---
The View model is the centerpiece to MVVM, with no external dependencies. As a result, the view model is an excellent class that requires no libraries while also exercising many language features related to classes. These requirements make view models a great example for learning how to build classes in a new language.

If learning F# (FSharp) is on your bucket list, getting started with classes is the easiest. Everything you need to know about F# classes is covered here. This post shows 23 (mostly) useless view models to get you started with F#. At the end, you will know how to make any useful view model in F#, so let's get started!

## View Model: #1
As already stated, in the purest sense, a view model is just a class. Creating classes in F# is really easy. Here is our first view model
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel1</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="fn">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="n">42</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
There is no need to create a new file. The only requirement is that it is declared before it is used.
<code>type</code> is synonymous with the <code>class</code> keyword. <code>ViewModel</code> is also public by default, which is generally what the developer intended. Parenthesis follow the class name.

<code>Foo</code> is a public method. They keyword <code>member</code> is used to declare anything that belongs to the class. Additionally don't forget the instance for the method <code>this.</code> required before the name of the method.

A few striking differences that immediately stand out to most developers is the lack of curly braces and types. The braces are omitted in favour of indentation. Should any problems arise with indentation, the compiler will highlight the line. Secondly, the lack of types can seam daunting. The F# compiler utilizes advanced mathematics so this generally not a problem. All items are given a type and checked for consistent usage. This is known as type inference.

## Variables: More than one way
Variables are possible in F#, and the language even has two ways of doing it. An example of each is listed below:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel2</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="n">42</span>

<span class="k">type</span> <span class="rt">ViewModel3</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="fn">ref</span> <span class="n">42</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
A key aspect to highlight in the examples above is that mutation/variables require 'opt in' with keywords. By default, all items are immutable, which is what most developers prefer. [Predictable code]({{ "/2016/12/14/predictable-code/" | relative_url }}) is the result of immutability.
Next up, our view model needs a getter to expose the variables.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel4</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="n">42</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> <span class="o">=</span> <span class="mv">_variable</span>

<span class="k">type</span> <span class="rt">ViewModel5</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="fn">ref</span> <span class="n">42</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> <span class="o">=</span> <span class="o">!</span><span class="mv">_variable</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
As with the methods, the <code>member</code> keyword is used, but the parentheses are left off. For the <code>ref</code> variable, note the <code>!</code> that is used to return the value contained. If this is missed, the wrong type will be returned. If a setter is also present (shown later), then a compiler error will result.
With the getters defined, a setter is is the next step. On a side note, in F# <code><-</code> is used for assignment. For the <code>ref</code> there is a shorthand <code>:=</code> to update the value.

Adding a setter to the property is simple and straight forward. Append the keyword <code>and</code> with <code>set</code> and then name the parameter to function, <code>value</code> is commonly used.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel6</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="n">42</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span>  
        <span class="k">with</span> <span class="prop">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="mv">_variable</span>
        <span class="k">and</span> <span class="prop">set</span><span class="pn">(</span><span class="id">value</span><span class="pn">)</span> <span class="o">=</span> <span class="mv">_variable</span> <span class="k"><-</span> <span class="id">value</span> <span class="c">//Assignment in F#</span>

<span class="k">type</span> <span class="rt">ViewModel7</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="mv">_variable</span> <span class="o">=</span> <span class="fn">ref</span> <span class="n">42</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> 
        <span class="k">with</span> <span class="prop">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="o">!</span><span class="mv">_variable</span> <span class="c">// same as _variable.Value</span>
        <span class="k">and</span> <span class="prop">set</span><span class="pn">(</span><span class="id">value</span><span class="pn">)</span> <span class="o">=</span> <span class="mv">_variable</span> <span class="o">:=</span> <span class="id">value</span> <span class="c">//Same as `_variable.Value <- value`</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## When to use <code>mutable</code> or <code>ref</code>
You should prefer <code>mutable</code> over <code>ref</code> as it is easier to with. You can use <code>ref</code> when using a framework that updates the value of properties in a subclass. To illustrate this, here is our next view model:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel8</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">inherit</span> <span class="id">FrameworkViewModel</span><span class="pn">(</span><span class="pn">)</span>

    <span class="k">let</span> <span class="id">_variable</span> <span class="o">=</span> <span class="id">ref</span> <span class="n">42</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> 
        <span class="k">with</span> <span class="prop">get</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="o">!</span><span class="id">_variable</span>
        <span class="k">and</span> <span class="prop">set</span><span class="pn">(</span><span class="id">value</span><span class="pn">)</span> <span class="o">=</span> 
            <span class="k">if</span> <span class="pn">(</span><span class="id">this</span><span class="pn">.</span><span class="id">SetProperty</span><span class="pn">(</span><span class="id">_variable</span><span class="pn">,</span> <span class="id">value</span><span class="pn">)</span><span class="pn">)</span> <span class="k">then</span> 
                <span class="id">this</span><span class="pn">.</span><span class="id">RaisePropertyChanged</span><span class="pn">(</span><span class="s">"Foo"</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
They key part to the code above is the call to <code>SetProperty</code> a method on the base class. For a view model, there is no need to set a value if it has not changed. <code>SetProperty</code> performs this check, and then updates the value if value has changed. A bool, ie true, will be returned if the value is updated. A <code>ref</code> makes this possible.

## Auto Properties: A new keyword
Auto properties have a backing variable. In F# the syntax is a little different to methods. Here is an example with a getter only
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel9</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="k">val</span> <span class="prop">Foo</span> <span class="o">=</span> <span class="n">42</span>

<span class="c">// Long form, prefer the above format if no setter</span>
<span class="k">type</span> <span class="rt">ViewModel10</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="k">val</span> <span class="prop">Foo</span> <span class="o">=</span> <span class="n">42</span> <span class="k">with</span> <span class="id">get</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The key difference here is the use of the keyword <code>val</code>. Additionally the instance, <code>this</code> no longer needs to declared before the name.
Adding a setter is also super easy, just add a comma and the keyword <code>set</code>:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel11</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="k">val</span> <span class="prop">Foo</span> <span class="o">=</span> <span class="mv">42</span> <span class="k">with</span> <span class="id">get</span><span class="pn">,</span> <span class="id">set</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## Scoping rules: There's a twist
Scoping rules are also available in F#. Methods can have their usage restricted to only inside the class with <code>private</code>.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel12</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="k">private</span> <span class="id">this</span><span class="pn">.</span><span class="fn">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="n">42</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
There is no scoping for <code>protected</code> as it creates potential problems with lamda functions accessing the base class. Instead use interfaces/higher-order functions.

## Prefer function over methods
It turns out there is another way to achieve the same as a private method in F#. A function declared in a class in most circumstances will behave the same as a private method. Because of this, it is best to prefer functions over methods, as functions have better type inference and can be chained together easier using F#'s iconic piper operator <code>|&gt;</code>.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel13</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">foo</span> <span class="o">=</span> <span class="n">42</span>
    <span class="k">let</span> <span class="fn">addTen</span> <span class="id">x</span> <span class="o">=</span> <span class="id">x</span> <span class="o">+</span> <span class="n">10</span>
    <span class="k">let</span> <span class="id">data</span> <span class="o">=</span> <span class="id">foo</span> <span class="o">|&gt;</span> <span class="fn">addTen</span> <span class="o">|&gt;</span> <span class="fn">string</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Closely related to access modifiers, is overriding methods. It is also possible in F#, and is just a keyword change to use <code>override</code> instead of <code>member</code>
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
<span class="l">3: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel14</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">inherit</span> <span class="id">MvvmBase</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="fn">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="n">42</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## Adding constructor arguments
Constructor arguments, are passed within the parentheses of the declaring line of the class. See below:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel15</span><span class="pn">(</span><span class="id">foo</span><span class="pn">:</span> <span class="vt">int</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span><span class="pn">:</span> <span class="vt">int</span> <span class="o">=</span> <span class="id">foo</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Types in F# are declared after the name with a colon. The type can also be omitted in many cases and F# will infer the type.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel16</span><span class="pn">(</span><span class="id">foo</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span><span class="pn">:</span> <span class="vt">int</span> <span class="o">=</span> <span class="id">foo</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
For this example, the method <code>Foo</code> has been declared to return an int. F# can figure out that the type of <code>foo</code> must also be an int. Alternatively, the type could be declared the other way around. The types and meaning are identical in both cases:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel17</span><span class="pn">(</span><span class="id">foo</span><span class="pn">:</span> <span class="vt">int</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> <span class="o">=</span> <span class="id">foo</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Another important point to highlight with constructor arguments in F# is that they are immutable:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// Will not compile. Error: `This value is not mutable`</span>
<span class="k">type</span> <span class="rt">ViewModel18</span><span class="pn">(</span><span class="id">foo</span><span class="pn">:</span> <span class="vt">int</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">fooPlusTen</span> <span class="o">=</span> 
        <span class="id">foo</span> <span class="k"><-</span> <span class="n">10</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If immutability is a problem and mutation is required, there is a simple solution. A copy of the constructor argument can be taken and declared with the <code>mutable</code> keyword highlighted above (though many prefer to avoid mutation, as functional competence is gained).
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel19</span><span class="pn">(</span><span class="id">foo</span><span class="pn">:</span> <span class="vt">int</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">_foo</span> <span class="o">=</span> <span class="id">foo</span>
    <span class="k">let</span> <span class="id">fooPlusTen</span> <span class="o">=</span> 
        <span class="mv">_foo</span> <span class="k"><-</span> <span class="n">10</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## A Constructor without a Constructor
The syntax for constructors is a quite different in F#; though much cleaner. To declare a constructor, after the local functions/values and before any member items, <code>do</code> signals the constructor followed by the required statements.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel20</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="k">mutable</span> <span class="mv">foo</span><span class="pn">:</span> <span class="rt">string</span> <span class="o">=</span> <span class="k">null</span>
    <span class="k">do</span> 
        <span class="mv">foo</span> <span class="k"><-</span> <span class="s">"Hello, World"</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="fn">Foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="mv">foo</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
In the example above, the class is initially constructed with <code>foo</code> set to <code>null</code>. The constructor (the statements after <code>do</code>), are evaluated and <code>foo</code> is updated to be "Hello, World".

The example above only aims to highlight the usage of <code>do</code>, however usage of <code>null</code> and <code>mutable</code> are not encouraged. As a developer learns techniques in functional programming, usage of <code>null</code> or <code>mutable</code> is required less. The reason for the reduced usage is that concepts in functional programming, model computation differently resulting in code that is much clear in intent and fewer runtime errors. For a brief highlight of this see my post on [How to turn runtime exceptions into compiler errors]({{ "/2016/12/03/net-none-none-none-or-exceptions/" | relative_url }}).

## Dependencies with interfaces
If you have read this far, we're almost at the point where all the knowledge has been laid out to build any standard view model. To complete this, you need to know how to pass in dependencies, notably interfaces.

An interface could be declared in C# or F# (The C# declaration would need to be in a different project). Here is one written in F#. There are no parenthesis after the name. Methods are declared with the <code>abstract</code> keyword and a type declaration.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="if">IService</span> <span class="o">=</span> 
    <span class="k">abstract</span> <span class="k">member</span> <span class="fn">Foo</span><span class="pn">:</span> <span class="rt">unit</span> <span class="k">-&gt;</span> <span class="vt">int</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
'Foo' is method that, when invoked, returns a single integer. With an interface declared, it can now be used. Here is a view model that is using the interface as a dependency. Simply declare the type and use it as you would any constructor argument.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel21</span><span class="pn">(</span><span class="id">foo</span><span class="pn">:</span> <span class="if">IService</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span><span class="pn">:</span> <span class="vt">int</span> <span class="o">=</span> <span class="fn">foo</span><span class="pn">.</span><span class="id">Foo</span><span class="pn">(</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If you were to compare this to the C#/Java equivalent, you will see that a copy of the dependency does not need to be taken. No strange attributes, no local constructor parameters. No duplicate names.

Another key feature with these interfaces and view models is that they are fully compliant with any IOC framework. Register the interface (F# or C#), make sure the view model is appropriately named and enjoy everything working. These F# interfaces/classes generate very similar IL that the C# equivalent would output, you benefit from clearer code.

## Interfaces are optional
The last example showed building a view model with a dependency through an interface. In large apps, both interfaces themselves and the number of interfaces a view model requires can get large. Many developers have found this can make things difficult to work with, and hard to refactor as the abstractions are no longer clear.
There is an alternative that many developers have already turned to. Replace the interface with just a function. For a full read up on this Scott Wlaschin provides the details: <a href="https://fsharpforfunandprofit.com/posts/dependency-injection-1/">Functional approaches to dependency injection</a>. To follow the approach that Scott talks about, all we need to do is pass in functions.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">ViewModel22</span><span class="pn">(</span><span class="fn">foo</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">data</span> <span class="o">=</span> <span class="fn">foo</span><span class="pn">(</span><span class="pn">)</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span><span class="pn">:</span> <span class="vt">int</span> <span class="o">=</span> <span class="id">data</span>

<span class="k">let</span> <span class="id">vm</span> <span class="o">=</span> <span class="id">ViewModel</span><span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-&gt;</span> <span class="n">42</span><span class="pn">)</span>  
</code></pre>
</td>
</tr>
</tbody>
</table>
As stated, no interface needed, just a function. Type inference checks the type for us so everything must be wired up correctly. Additionally, it must be noted that construction of the class must be done explicitly (Leave a comment if you know of an IOC container that can resolve functions). The result is that the code is now easier to read, especially for juniors since an IOC container is an advanced topic.

Using this functional approach will result in many functions passed into the view model. Incase it gets hard to keep track of those names and types, an alias for the type can be created to keep dependencies readable. Here's an example with a couple of dependencies:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="rt">IFoo</span> <span class="o">=</span>  <span class="rt">unit</span> <span class="k">-&gt;</span> <span class="vt">int</span>
<span class="k">type</span> <span class="rt">IBar</span> <span class="o">=</span>  <span class="rt">unit</span> <span class="k">-&gt;</span> <span class="rt">string</span>
<span class="k">type</span> <span class="rt">ViewModel23</span><span class="pn">(</span><span class="fn">foo</span><span class="pn">:</span> <span class="rt">IFoo</span><span class="pn">,</span> <span class="fn">bar</span><span class="pn">:</span> <span class="rt">IBar</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">data</span> <span class="o">=</span> <span class="fn">foo</span><span class="pn">(</span><span class="pn">)</span> <span class="o">|&gt;</span> <span class="fn">string</span>
    <span class="k">let</span> <span class="id">message</span> <span class="o">=</span> <span class="fn">bar</span><span class="pn">(</span><span class="pn">)</span> <span class="o">+</span> <span class="s">" , "</span> <span class="o">+</span> <span class="id">data</span>
    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="prop">Foo</span> <span class="o">=</span> <span class="id">message</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The type of <code>foo</code> and <code>bar</code> are declared explicitly, rather than being inferred by the compiler. As arguments to the class they now hve readable types as opposed to function types. The alias only affects readability, and will not help the compiler find errors.

So there you go, 23 absolutely useless view models. F# makes it easier and clear to create view models. The code is shorter and with a powerful compiler, both typing and errors can be reduced!

Your challenge: How many ways can you combine the 22 view models to make useful view models?