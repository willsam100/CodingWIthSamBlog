---
layout: post
title: 'Xamarin Forms ListView with F# part 2: Custom cell'
date: 2016-08-09 19:01:15.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [UI]
tags: [Android, F#, iOS, ListView, XamarinForms, Mobile]
meta:
  _edit_last: '1'
  mytory_md_path: ''
  mytory_md_text: ''
  mytory_md_mode: url
  mashsb_timestamp: '1589229568'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/xamarin-forms-listview-with-f-part-2-custom-cell/
  dsq_thread_id: '6134873776'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2016/08/09/xamarin-forms-listview-with-f-part-2-custom-cell/"
---

As promised, this post is part 2 of the ListView series. If you missed [part 1]({{ "/2016/07/26/xamarin-forms-listview-with-xaml-and-f" | relative_url }}), I covered setting up a Xamarin Forms ListView with a data source as per the Xamarin guide. In this post, I'll cover the next page in the <a href="https://developer.xamarin.com/guides/xamarin-forms/user-interface/listview/customizing-cell-appearance/">Xamarin guide </a>with a custom cell.


## Getting started: add the xaml

Let's get to it, create another forms project with F#, for this post I'll use the name **CustomSimpleList**. Add the xaml file (see my [part 1]({{ "/2016/07/26/xamarin-forms-listview-with-xaml-and-f" | relative_url }}) if you've forgotten this), I'll name this CustomListView.xaml and add the following:

```xml
<contentpage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:class="CustomSimpleList.CustomCellPage">
    <contentpage.content>
        <listview x:name="listView">
            <listview.itemtemplate>
                <datatemplate>
                    <viewcell>
                        <stacklayout backgroundcolor="#eee" orientation="Vertical">
                            <stacklayout orientation="Horizontal">
                                <label text="{Binding Title}" textcolor="#f35e20">
                                </label><label text="{Binding Rating}" horizontaloptions="EndAndExpand" textcolor="#503026">
                            </label></stacklayout>
                        </stacklayout>
                    </viewcell>
                </datatemplate>
            </listview.itemtemplate>
        </listview>
    </contentpage.content>
</contentpage>
```

Be sure to update the following items:

- Class to have the correct namespace and name
- Remove the line: 

```xml
<image source="{Binding image}" />
```

(details why are at the bottom).

## Create some Types

With the xaml created, it's time to create some types. As in the last post open, the fs file that was automatically created. For the main type I'll use the name CustomCellPage, so update app to load this as main page (don't forget to open the required libraries):
```inherit Application(MainPage = CustomCellPage())```
and then above that line add the type:

```fsharp
type CustomCellPage() =
    inherit ContentPage() 

    let _ = base.LoadFromXaml(typeof<customcellpage>)
    let movies = [
        {Title = "Zootopia (2016)"; Rating = "98%"} 
        {Title = "Love & Friendship (2016)"; Rating = "99%"}
        {Title = "The Jungle Book (2016)"; Rating = "94%"}
        {Title = "Hunt for the Wilderpeople (2016)"; Rating = "98%"}
        {Title = "Finding Dory (2016)"; Rating = "94%"} ]
    let movieList = base.FindByName<listview>("listView")
    do movieList.ItemsSource <- ((movies |> List.toSeq) :> Collections.IEnumerable)
```

This should look familiar as the only thing that has changed is the record type.

## Make it compile

To finish this small example, add the last required type above the CustomCellPage:
```fsharp
type Movie = {
    Title: string
    Rating: string
}
```
With that last type added everything should compile and run now. Voila!

## But where are my images    
So the original guide from Xamarin had images, what's up? Well, there's a bug in F# with forms and Android (I'm not sure if this affects iOS. If some has tried it out I would love to know). I have filled a bug with xamarin for this and will add an update when I get one. 

## What's actually broken
Following <a href="https://developer.xamarin.com/guides/xamarin-forms/working-with/images/">this guide</a> using either local images or embedded images an exception is thrown when trying to load the image from the image source. There is an example <a href="https://github.com/willsam100/ImageSingle">available on github</a> if you want the details, it uses local images. For embedded images, I also added the debugging code and verified that images are in the package. Additionally, I haven't found a work around for this (maybe writing some custom UI for forms). Again leave me a comment if you have found a workaround or also encounter this bug.
That's a wrap folks!
