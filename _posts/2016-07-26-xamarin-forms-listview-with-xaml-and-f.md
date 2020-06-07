---
layout: post
title: Xamarin Forms ListView with xaml and F#
date: 2016-07-26 18:41:14.000000000 +12:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
# categories:
# - GUI
# - Mobile
# tags:
# - Android
# - F#
# - Forms
# - iOS
# - listView
# - XamarinForms
meta:
  _edit_last: '1'
  _oembed_ab970a0b094524d8501be95a59b7918d: "{{unknown}}"
  _oembed_716aad282c4fd01956da7bab2367a707: "{{unknown}}"
  mytory_md_path: ''
  mytory_md_text: ''
  mytory_md_mode: url
  mashsb_timestamp: '1589841261'
  mashsb_shares: '0'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shorturl: http://www.codingwithsam.com/xamarin-forms-listview-with-xaml-and-f/
  dsq_thread_id: '6132678432'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2016/07/26/xamarin-forms-listview-with-xaml-and-f/"
---
In this short post I will show how to us Xamarin forms ListView with F#. I'll use the example from the Xamarin ListVIew guides <a href="https://developer.xamarin.com/guides/xamarin-forms/user-interface/listview/data-and-databinding/">here</a> and the code can also be found on <a href="https://github.com/willsam100/SimpleList">my repo</a>. A screenshot below is what we will create, very similar to the Xamarin guide.

<img class="size-medium wp-image-31 aligncenter" src="{{ site.baseurl }}/assets/img/Screen-Shot-2016-07-23-at-4.27.51-PM-194x300.png" alt="Screen Shot 2016-07-23 at 4.27.51 PM" width="194" height="300" />
&nbsp;

## Creating the project
To begin with, create a Forms solution and project with F#, and give it a name. I'll use SimpleList. My build of Xamarin Studio has a small bug so in the droid (and iOS if you have it) project I had to replace the line
```fsharp
this.LoadApplication (new ${AppName}.App ())
```
with
```fsharp
this.LoadApplication (new SimpleList.App ())
```
Check that everything builds.
&nbsp;

## Adding the xaml
It's time to add the xaml. Because we're using F#, native xaml is not yet supported so we must add it manually. Jonathan Wood  has a <a href="http://www.wintellect.com/devcenter/jwood/using-xaml-f-xamarin-forms-screencast">video</a> on how to do this, the steps are:

- Add an empty text file to the SimpleList project (any name is fine, i will use SimpleList.xaml), make sure to add the extension .xaml`
- right click on the xaml file, select 'Build Action' and the select 'EmbededResource'`

Now we have an empty xaml file we need some xaml. Following the guide from xamarin, add the following:<code>

```xml
<pre lang="xml"><?xml version="1.0" encoding="utf-8"?>
<contentpage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" xmlns:constants="clr-namespace:XamarinFormsSample;assembly=XamarinFormsXamlSample" x:class="XamarinFormsXamlSample.Views.EmployeeListPage" title="Employee List">
  <listview x:name="EmployeeView">
    <listview.itemtemplate>
      <datatemplate>
        <textcell text="{Binding DisplayName}" />
      </datatemplate>
    </listview.itemtemplate>
  </listview>
</contentpage>
```
There are couple of items to take note of here. Starting from the top,  the name of this ContentPage is ```EmployeeListPage``` in the namespace ```SimpleList``` so we will need to use this to load the xaml from F#. The name of the list is ```EmployeeView``` which will also be used to find the item from F#. Finally the name of the property that will be displayed is ```DisplayName```.
&nbsp;

## Creating the F# types
With xaml all done, open the main fsharp file (extension fs), SimpleList.fs and add this to top of the file:
```fsharp
open Xamarin.Forms.Xaml
open System
```
then update the following:
```fsharp
type App() =
  inherit Application()
  ```
with
```fsharp
type App() =
  inherit Application(MainPage = EmployeeListPage())
  ```
The type for EmployeeListPage doesn't yet exist but we'll create it next. The code inserted tells Xamarin what to load as the main page. Now to create the EmployeeListPage, above type declaration for App add the following (F# can only reference things above it in declaration, including across multiple files):
```fsharp
type EmployeeListPage() =
  inherit ContentPage()
  base.LoadFromXaml(typeof<EmployeeListPage>) |> ignore
```
Here we have created a type of ```EmployeeListPage``` which is akin to a class in C#, and loaded the xaml. There is no content, but you should be able to build and run and now.
&nbsp;

## Adding Data To The List
Did you check that it builds and runs? awesome, let's add some employee data. For this we need a place to store the data and it must have a property with the name DisplayName, an F# record will suffice. Add the following above <strong>EmployeeListPage:</strong>
```fsharp
type Employee = {
  DisplayName: string
}
```
this declares an F# record which is the same as an immutable C# class (private setters) and override equals methods for equality on the string i.e. not reference equality. In the EmployeeListPage some employees can be created:
```fsharp
let employees = [
    { DisplayName="Rob Finnerty"}
    { DisplayName="Bill Wrestler"}
    { DisplayName="Dr. Geri-Beth Hooper"}
    { DisplayName="Dr. Keith Joyce-Purdy"}
    { DisplayName="Sheri Spruce"}
    { DisplayName="Burt Indybrick"}  ]
```
This creates a list of Employee's (the list is an F# list that is not available in C#) and creates an instance of each of. The syntax of F# really shines here as creating objects is really easy and putting the items on new lines means I didn't have to add a semi-colon ( ; ) between each item. Finally we need to give the list to the list view so add the last two lines:

```fsharp
let employeeListView = base.FindByName<listview>("EmployeeView")
do employeeListView.ItemsSource <- ((employees |> List.toSeq) :> Collections.IEnumerable)
```
The first line is pulling the ListView out of the xaml by name (not the best from a functional perspective since this isn't type safe but will do for now). The second line is setting the ItemSource to our data (yes ItemsSource is a variable :( but no getting around this for now). The last part of that line is doing some type changing. Earlier I said that the list was an F# type; because of that we need to change it to a Seq which is an IEnumerable on .NET, then a cast will keep the compiler happy. We're all done now fire up and the app and see the list!
&nbsp;

## One Final Touch
Excellent did you get that going fine? There was one little thing that bugged me a bit about the list. When you touched on an item it remained highlighted. To fix that we need to add an event handler and clear out the selected item. Since F# runs on an OO framework that means using the null keyword :(.
Add the following below the line of setting the ItemSource:
```fsharp
do employeeListView.ItemTapped.AddHandler(
  new EventHandler<itemtappedeventargs>(fun x y -> (x :?> ListView).SelectedItem <- null ))
```
A little bit of new F# syntax here, the keyword <strong>fun </strong>creates an anonymous function and <strong>:?>; </strong>does a downcast since the first parameter of the method must be of type object. From there the selectedItem can be set to null. Build and run and see that the highlighting has been removed.
&nbsp;
That's a wrap for this part folks. I haven't made a mention of indention for the F# code (like Python and Haskell, F# is also whitespace sensitive. For a full explanation check out <a href="https://fsharpforfunandprofit.com/posts/fsharp-syntax/">this</a> ) so see my example if you have any trouble building. Part 2 will expand a little further with a custom data cell i.e. the next page in Xamarin ListView Guide.
