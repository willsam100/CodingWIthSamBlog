---
layout: post
title: Grokking Relative Layout
date: 2017-01-14 06:54:08.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [GUI]
tags: [GUI,XamarinForms,Xaml]
meta:
  _edit_last: '1'
  _mytory_markdown_etag: '"2b7ca975e89a8072b524ae31f30abe497a39c69b"'
  mytory_md_path: https://raw.githubusercontent.com/willsam100/CodingWithSam/master/RelativeLayout.md
  mytory_md_text: ''
  mytory_md_mode: url
  mashsb_timestamp: '1588760300'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/grokking-relative-layout/
  dsq_thread_id: '6141487413'
  mashsb_twitter_handle: willsam100
  _yoast_wpseo_primary_category: '29'

  _yoast_wpseo_content_score: '30'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
description: A example of using Xamarin Forms relative layout. This post
  highlights when to use relative layout instead of the preferred defaults StackLayout/GridLayout
permalink: "/2017/01/14/grokking-relative-layout/"
---
For a Xamarin Forms app, there are several layout containers to choose from. Most UIs should be achievable with a StackLayout or a GridLayout. There are a few exceptions when those two won't solve the problem. This post is an example of Xamarin Forms relative layout.

## The Problem: Why a RelativeLayout
I had to show an triangle in the corner of the screen. See the image below.
<img title="screenshot" src="{{ site.baseurl }}/assets/img/cornerui.png" alt="alt text" />
A StackLayout is very difficult to get items to overlap, a negative margin can be used, but only for a known distance (won't always work for screens with different sizes). Items can be overlapped with a GridLayout if they are placed in the same cell. This renders the items over the entire cell, and my requirement was only part of the cell, the triangle. Using XAML only, this is not possible, and could only be done in code with a translation.

## What is a View
A view, the 'things' that we're trying to layout on the screen and requires a few key pieces of information, called attributes.
- An origin to start drawing the item from. This is an X and Y value, with (0,0) at the top left-hand corner (of the device or container it's being drawn in)
- A width
- A height

## Setting the attributes of a child
A RelativeLayout allows you to specify the X, Y, Width and Height of a child. The values that are set can be pulled from any other child or parent. They do not, need to be the same attribute (X, Y, Width, Height). To set the attributes of the child is as follows:
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
<pre class="fssnip highlighted"><code lang="xml"><span class="k">&lt;</span><span class="i">BoxView</span>  <span class="o">RelativeLayout</span>.<span class="o">YConstraint</span>= ...
          <span class="o">RelativeLayout</span>.<span class="o">XConstraint</span>= ...
          <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span>= ...
          <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span>= ... <span class="k">/&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The simplest value to apply to each of these is the same as the parent. Setting all four values to be the same as the parent would make the view fill the space the parent is given. This is as follows
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
<pre class="fssnip highlighted"><code lang="xml"><span class="k">&lt;</span><span class="i">BoxView</span> <span class="o">RelativeLayout</span>.<span class="o">YConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Y}"</span>
         <span class="o">RelativeLayout</span>.<span class="o">XConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=X"</span>
         <span class="o">RelativeLayout</span>.<span class="o">HeightConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Height}"</span>
         <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Width,Factor=1,Constant=0}"</span> <span class="k">/&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
First, the value must be declared as ConstraintExpression. The type of the relationship is then declared, in this case, RelativeToParent, we'll look at another one is just a moment. The last line also shows two extra parameters that can be used (and are generally needed to do anything useful with RelativeLayout). The constant attribute simply adds the value to the property (like an offset). ie for HeightConstraint if constant=10 then the height would be the property + 10. The value can be negative. A factor is simply a multiplier to the Property attribute. eg.
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="xml"><span class="k">&lt;</span><span class="i">BoxView</span> <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span>= <span class="k">"{ConstraintExpression Type=RelativeToParent, Property=Width,Factor=0.5,Constant=0}"</span>
         ... <span class="k">/&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
If the width of the parent is 480 and the factor is 0.5 then the with of the item will be 240 meaning the view will be half the width of the parent.

## The Relative part of a RelativeLayout
Each of the four attributes can be set from any attribute of another view with the relative layout. They can also be expressed relative to another view. The other view needs to be named using <code>x:Name="itemName"</code>. To get values out of the view use <code>Type=RelativeToView</code> and then specify the name of the view with the Element tag <code>Element=itemName</code> finally to get the value out <code>Property=Height</code> or any other attribute name that is required. An example is below:
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
<pre class="fssnip highlighted"><code lang="xml"><span class="k">&lt;</span><span class="i">RelativeLayout</span><span class="k">&gt;</span>
<span class="k">&lt;</span><span class="i">BoxView</span> <span class="o">x:Name</span><span class="k">="itemName"</span> <span class="o">Text</span><span class="k">="myLabel"</span> 
       <span class="o">RelativeLayout</span>.<span class="o">YConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Y}"</span>
       <span class="o">RelativeLayout</span>.<span class="o">XConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=X"</span>
       <span class="o">RelativeLayout</span>.<span class="o">HeightConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Height}"</span>
       <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Width,Factor=1,Constant=0}"</span> <span class="k">/&gt;</span>
<span class="k">&lt;</span><span class="i">BoxView</span> <span class="o">RelativeLayout</span>.<span class="o">YConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Width, Factor=0.5}"</span>
         <span class="o">RelativeLayout</span>.<span class="o">XConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=X"</span>
         <span class="o">RelativeLayout</span>.<span class="o">HeightConstraint</span><span class="k">="{ConstraintExpression  Type=RelativeToParent, Property=Height, Factor=0.5}"</span>
         <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToView, Element=itemName,Property=Width,Factor=1,Constant=0}"</span><span class="k">/&gt;</span>
<span class="k">&lt;/</span><span class="i">RelativeLayout</span><span class="k">&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The first BoxView will fill the entire screen. The second BoxView will fill the bottom half of the screen. The with is equal to the width of itemName, which is set to the width of the screen.

## How I did the Triangle
As already shown in the image above, I needed to show a triangle in the top left corner of a cell (the cell is part of a ListView), and I also wanted it to scale for screen sizes (the test was to rotate to landscape and still have everything layout correctly). This was achieved by nesting everything in a relative layout. The content was placed in a StackLayout. A BoxView was then used for the triangle. It was set relative to the origin, a fixed width and a negative constant so that it is always placed correctly. Add some rotation and it's now a triangle. Because the width is always fixed size then the constant value can be known to ensure it will always be placed correctly. Here's the XAML to make it clear.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="xml"><span class="k">&lt;</span><span class="i">ViewCell</span><span class="k">&gt;</span>
        <span class="k">&lt;</span><span class="i">RelativeLayout</span> <span class="o">HeightRequest</span><span class="k">="100"</span> <span class="o">MinimumHeightRequest</span><span class="k">="100"</span><span class="k">&gt;</span>
            <span class="k">&lt;</span><span class="i">StackLayout</span> <span class="o">HorizontalOptions</span><span class="k">="FillAndExpand"</span> <span class="o">x:Name</span><span class="k">="anchorView"</span><span class="k">&gt;</span>
                    <span class="k">&lt;</span><span class="i">Label</span> <span class="o">Text</span><span class="k">="Your code here"</span><span class="k">/&gt;</span>
            <span class="k">&lt;/</span><span class="i">StackLayout</span><span class="k">&gt;</span>
            <span class="k">&lt;</span><span class="i">BoxView</span> <span class="o">Color</span><span class="k">="#FFD180"</span> <span class="o">Rotation</span><span class="k">="-45"</span> <span class="o">Opacity</span><span class="k">=".90"</span>
                        <span class="o">RelativeLayout</span>.<span class="o">YConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent, Property=Y,Factor=0.5}"</span>
                        <span class="o">RelativeLayout</span>.<span class="o">XConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToView, ElementName=anchorView, 
                                                    Property=Width, Factor=0, Constant=-210}"</span>
                        <span class="o">RelativeLayout</span>.<span class="o">WidthConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent,Property=Width,Factor=0,Constant=400}"</span>
                        <span class="o">RelativeLayout</span>.<span class="o">HeightConstraint</span><span class="k">="{ConstraintExpression Type=RelativeToParent,Property=Height,Factor=0.5}"</span> <span class="k">/&gt;</span>
        <span class="k">&lt;/</span><span class="i">RelativeLayout</span><span class="k">&gt;</span>
<span class="k">&lt;/</span><span class="i">ViewCell</span><span class="k">&gt;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

## What it can't do
For those who have used <a href="https://developer.android.com/guide/topics/ui/layout/relative.html">RelativeLayout in android</a>, Xamarin Forms' RelativeLayout is not as expressive in XAML. For a page to be effective, it's best to avoid fixed sizing, or it won't scale well on all those Android devices (and the growing number of devices for apple too). Halfway through this post I tried to create a great example using a RelativeLayout and xaml only to find that it can't be done, constants would have been required, so a GridLayout would have been better.

RelativeLayout can't be a StackLayout. In order to place one view below the other, you would need to set the Y value of the bottom view equal to the Y value of the top view + the height. This calculation can't be done in XAML. As a result, it's difficult to lay out a view that will fit all screen sizes. Using the Constant attribute doesn't work well across screen sizes.

It can't be a GridLayout as well for the same reason as the StackLayout, it's not possible in XAML to add values together.

## Summary
RelativeLayout is great for adding a few styles, but for most of your views stick to GridLayout and StackLayouts.
