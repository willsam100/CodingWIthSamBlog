---
layout: post
title: 'Xamarin Forms Push Notifications: a worked example with FCM on Android'
date: 2019-03-11 19:00:43.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Xamarin]
tags: [FCM, Messaging]
meta:
  _wpcom_is_markdown: '1'
  _oembed_778743a0ed7e4daa2438cfa807bc5f6b: "{{unknown}}"
  _oembed_a2f4bae30c1f92392ac67baa15c877c8: "{{unknown}}"
  _oembed_a4da5f92c80c1bd8d2a291c928007cc7: "{{unknown}}"
  _oembed_3573674dac044e4919e0e06a589dab8a: "{{unknown}}"
  _oembed_6100369467a93eb3aa9c4c96195d32e2: "{{unknown}}"
  _oembed_6abe3c892f07fac02135bf475d46a305: "{{unknown}}"
  _oembed_dbe0a6874c6a366466ab7dc843b51a78: "{{unknown}}"
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591387799'
  _yoast_wpseo_primary_category: '1'
  _yoast_wpseo_content_score: '30'
  mashsb_jsonshares: '{"total":0,"error":"","facebook_shares":0}'
  mashsb_shares: '0'
author:
  login: sam
  email: willsam100@gmail.com
  display_name: Sam Williams
  first_name: Sam
  last_name: Williams
permalink: "/2019/03/11/xamarin-forms-push-notifications-a-worked-example-with-fcm-on-android/"
---
Google's new firebase cloud messaging is a great way to send messages to your users or devices. Best of all, it is a great way to get a little bit of extra background processing time on Android 8+. Xamarin has bindings for Android and iOS to make all of this really simple.

I had to set this up recently, so here are the steps that I had to take to get it going. So you can follow along. I'll build a demo app call <code>pushy</code>, a worked example. FCM supports iOS, but this post focuses on Android (I think we all know that iOS will 'just work').

## Overview
This guide will focus on Android with Xamarin Forms, which is similar to their Java/Kotlin guide.
There are 3 simple steps to setting up FCM (Firebase Cloud Messaging)

- Google Console: creating project and keys to add into the device
- Add libs in the app and subscribing
- Setting up the backend to store device tokens and sending messages to devices
This post follows that order:

- Console setup
- Nugets + Google Play Services
- App code
- App testing
- Better app code
- Backend code
- Handling background/foreground
- Android challenges with Push
- GCM migration

## The app - Pushy
The app we will be building is very very simple but will highlight most of the challenges with Push Notifications. The app will receive a push notification, sent from the server to a specific device. If the app is in the foreground it will show the message on the UI. If the app is not in the foreground, it will show a notification.

## Google Console
For Xamarin, this step is exactly the same as Google describes it.
Adding a project is really straight forward. Simply go to <a href="https://firebase.google.com/">Firebase Console</a>, signup and create a new account.
Then select create a new project, and give it a meaningful name:

<img src="{{ site.baseurl }}/assets/img/FirebaseAddProject.png" alt="Smiley face"  />
See the end of this post for details on migrating from GCM
Add an Android app (Firebase should provide a prompt to add an app after creating the project).
Navigate to settings:

<img src="{{ site.baseurl }}/assets/img/FirebaseProjectSettings.png" alt="Smiley face"  />
From there simply add your app name and download the <code>google-servics.json</code> file. If you missed this step on sign-up, go to settings, and then scroll down. If you did add your app you can download the file, or you can add an app, and then download the file.
<img src="{{ site.baseurl }}/assets/img/FirebaseGoogleServicesJson.png" alt="Smiley face" height="700" width="700" />

## Adding FCM to the app
In Visual Studio, File -> New Project. I'll be using the name <code>pushy</code> (I'll also be using F#), It's a standard Xamarin.Forms app.

In the Android project, make sure you are targeting Android 8 or higher. If not, first update it via the project settings. If you have an existing app, then NO it is not safe to just to update (as some guides state). Changing the Android target is beyond the scope of this post but in short, you must check that you are not using any old APIs that have been removed. If your app does, you will need to change it. The biggest are runtime permissions, background modes and implicit intents (you can't subscribe to power connected).

### Add the files and Nuget packages
Add these two NuGet packages to your Android project:

- Xamarin.GooglePlayServices.Base
- Xamarin.Firebase.Messaging

Add <code>google-servics.json</code> as a file to the Android project (this file can be downloaded from the firebase console again if you deleted it).
**Reload the solution** if you are using VS4Mac, as the IDE does not provide the correct option.

Once that is done, set the correct Build Action:

<img src="{{ site.baseurl }}/assets/img/GoogleServiceFileBuildAction.png" alt="Smiley face" height="700" width="700" />

### Check Google play services if you must
For production, this step must be followed, but a simple test maybe not.
For the Android code snippet, you can see <a href="https://docs.microsoft.com/en-us/xamarin/android/data-cloud/google-messaging/remote-notifications-with-fcm?tabs=macos">Xamarin's post on this</a>

I wanted this to work with Xamarin.Forms and I also prefer to use F#.

In the core project, above <code>MainPage</code> class, I have declared the following. It captures all the possible states for Google Play Services (similar to an enum), but can also hold data too (ie when user input is required in this case).
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="rt">GooglePlayServicesAvailable</span> <span class="o">=</span> 
<span class="pn">|</span> <span onmouseout="hideTip(event, 'fs2', 2)" onmouseover="showTip(event, 'fs2', 2)" class="uc">HasGooglePlayServices</span>
<span class="pn">|</span> <span onmouseout="hideTip(event, 'fs3', 3)" onmouseover="showTip(event, 'fs3', 3)" class="uc">RequiresUser</span> <span class="k">of</span> <span onmouseout="hideTip(event, 'fs4', 4)" onmouseover="showTip(event, 'fs4', 4)" class="rt">string</span> <span class="c">// We can store the message to show the user</span>
<span class="pn">|</span> <span onmouseout="hideTip(event, 'fs5', 5)" onmouseover="showTip(event, 'fs5', 5)" class="uc">NoGooglePlayServices</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The following code is in the <code>MainActivity</code> as a private method. Here we check Google Play Services and return one of the values from <code>GooglePlayServicesAvailable</code> type
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span onmouseout="hideTip(event, 'fs6', 6)" onmouseover="showTip(event, 'fs6', 6)" class="fn">isPlayServicesAvailable</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
    <span class="k">let</span> <span onmouseout="hideTip(event, 'fs7', 7)" onmouseover="showTip(event, 'fs7', 7)" class="id">resultCode</span> <span class="o">=</span>
        <span class="id">GoogleApiAvailability</span><span class="pn">.</span><span class="id">Instance</span><span class="pn">.</span><span class="id">IsGooglePlayServicesAvailable</span><span class="pn">(</span><span class="id">this</span><span class="pn">)</span>

    <span class="k">if</span> <span onmouseout="hideTip(event, 'fs7', 8)" onmouseover="showTip(event, 'fs7', 8)" class="id">resultCode</span> <span class="o">=</span> <span class="id">ConnectionResult</span><span class="pn">.</span><span class="id">Success</span> <span class="k">then</span>
        <span onmouseout="hideTip(event, 'fs2', 9)" onmouseover="showTip(event, 'fs2', 9)" class="uc">HasGooglePlayServices</span>
    <span class="k">else</span> 
        <span class="k">if</span> <span class="id">GoogleApiAvailability</span><span class="pn">.</span><span class="id">Instance</span><span class="pn">.</span><span class="id">IsUserResolvableError</span> <span onmouseout="hideTip(event, 'fs7', 10)" onmouseover="showTip(event, 'fs7', 10)" class="id">resultCode</span> <span class="k">then</span>
            <span class="id">GoogleApiAvailability</span><span class="pn">.</span><span class="id">Instance</span><span class="pn">.</span><span class="id">GetErrorString</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs7', 11)" onmouseover="showTip(event, 'fs7', 11)" class="id">resultCode</span><span class="pn">)</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs3', 12)" onmouseover="showTip(event, 'fs3', 12)" class="uc">RequiresUser</span>
        <span class="k">else</span> <span onmouseout="hideTip(event, 'fs5', 13)" onmouseover="showTip(event, 'fs5', 13)" class="uc">NoGooglePlayServices</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Now I can pass this directly into my forms app to be used from the application.
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// In MainActivity.OnCreate</span>
<span class="id">this</span><span class="pn">.</span><span class="id">LoadApplication</span> <span class="pn">(</span><span class="k">new</span> <span class="id">Pushy</span><span class="pn">.</span><span class="id">App</span> <span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 14)" onmouseover="showTip(event, 'fs6', 14)" class="id">isPlayServicesAvailable</span><span class="pn">)</span><span class="pn">)</span>

<span class="c">// Xamarin Forms App pass function through to MainPage</span>
<span class="id">App</span><span class="pn">(</span><span class="id">isPlayServicesAvailablen</span><span class="pn">)</span> <span class="o">=</span>
    <span class="k">inherit</span> <span class="id">Application</span><span class="pn">(</span><span class="id">MainPage</span> <span class="o">=</span> <span class="id">MainPage</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 15)" onmouseover="showTip(event, 'fs6', 15)" class="id">isPlayServicesAvailable</span><span class="pn">)</span><span class="pn">)</span>

<span class="c">// MainPage showing usage</span>
<span class="k">type</span> <span class="id">MainPage</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 16)" onmouseover="showTip(event, 'fs6', 16)" class="id">isPlayServicesAvailable</span><span class="pn">)</span> <span class="o">=</span>
<span class="k">inherit</span> <span class="id">ContentPage</span><span class="pn">(</span><span class="pn">)</span>
<span class="k">let</span> <span class="id">_</span> <span class="o">=</span> <span class="k">base</span><span class="pn">.</span><span class="id">LoadFromXaml</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs8', 17)" onmouseover="showTip(event, 'fs8', 17)" class="id">typeof</span><span class="pn">&lt;</span><span class="id">MainPage</span><span class="pn">></span><span class="pn">)</span>
<span class="k">let</span> <span class="id">label</span> <span class="o">=</span> <span class="k">base</span><span class="pn">.</span><span class="id">FindByName</span><span class="pn">&lt;</span><span class="id">Label</span><span class="pn">></span><span class="pn">(</span><span class="s">"Label"</span><span class="pn">)</span>

<span class="k">do</span> 
    <span class="k">let</span> <span class="id">message</span> <span class="o">=</span> 
        <span class="k">match</span> <span onmouseout="hideTip(event, 'fs6', 18)" onmouseover="showTip(event, 'fs6', 18)" class="id">isPlayServicesAvailable</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">with</span> 
        <span class="pn">|</span> <span onmouseout="hideTip(event, 'fs2', 19)" onmouseover="showTip(event, 'fs2', 19)" class="id">HasGooglePlayServices</span> <span class="k">-></span> <span class="s">"Pushy is ready for push"</span>
        <span class="pn">|</span> <span onmouseout="hideTip(event, 'fs5', 20)" onmouseover="showTip(event, 'fs5', 20)" class="id">NoGooglePlayServices</span> <span class="k">-></span> <span class="s">"Pushy is not supported on this device"</span>
        <span class="pn">|</span> <span onmouseout="hideTip(event, 'fs3', 21)" onmouseover="showTip(event, 'fs3', 21)" class="id">RequiresUser</span> <span class="id">x</span> <span class="k">-></span> <span onmouseout="hideTip(event, 'fs9', 22)" onmouseover="showTip(event, 'fs9', 22)" class="id">sprintf</span> <span class="s">"Pushy needs your help: %s"</span> <span class="id">x</span>

    <span class="id">label</span><span class="pn">.</span><span class="id">Text</span> <span class="k">&lt;-</span> <span class="id">message</span> 
</code></pre>
</td>
</tr>
</tbody>
</table>
Alternatively, I could wire this function up to Xamarin Forms' Dependency Service.

### Register the app with Google.
This was much harder when using GCM, now it is pretty easy. In the application, manifest add the following. No need to change anything:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pn">&lt;</span><span class="id">receiver</span>
    <span class="id">android</span><span class="pn">:</span><span class="id">name</span><span class="o">=</span><span class="s">"com.google.firebase.iid.FirebaseInstanceIdInternalReceiver"</span>
    <span class="id">android</span><span class="pn">:</span><span class="id">exported</span><span class="o">=</span><span class="s">"false"</span> <span class="o">/></span>
<span class="pn">&lt;</span><span class="id">receiver</span>
    <span class="id">android</span><span class="pn">:</span><span class="id">name</span><span class="o">=</span><span class="s">"com.google.firebase.iid.FirebaseInstanceIdReceiver"</span>
    <span class="id">android</span><span class="pn">:</span><span class="id">exported</span><span class="o">=</span><span class="s">"true"</span>
    <span class="id">android</span><span class="pn">:</span><span class="id">permission</span><span class="o">=</span><span class="s">"com.google.android.c2dm.permission.SEND"</span><span class="pn">></span>
    <span class="pn">&lt;</span><span class="id">intent</span><span class="o">-</span><span class="id">filter</span><span class="pn">></span>
        <span class="pn">&lt;</span><span class="id">action</span> <span class="id">android</span><span class="pn">:</span><span class="id">name</span><span class="o">=</span><span class="s">"com.google.android.c2dm.intent.RECEIVE"</span> <span class="o">/></span>
        <span class="pn">&lt;</span><span class="id">action</span> <span class="id">android</span><span class="pn">:</span><span class="id">name</span><span class="o">=</span><span class="s">"com.google.android.c2dm.intent.REGISTRATION"</span> <span class="o">/></span>
        <span class="pn">&lt;</span><span class="id">category</span> <span class="id">android</span><span class="pn">:</span><span class="id">name</span><span class="o">=</span><span class="s">"${applicationId}"</span> <span class="o">/></span>
    <span class="o">&lt;/</span><span class="id">intent</span><span class="o">-</span><span class="id">filter</span><span class="pn">></span>
<span class="o">&lt;/</span><span class="id">receiver</span><span class="pn">></span>
</code></pre>
</td>
</tr>
</tbody>
</table>
<blockquote>Note: Because we are using Xamairn, updating the Application Manifest is required. Google's docs for Java app state that this can be removed.</blockquote>
Finally, we add the service to call the Google Nuget library. I have put this above MainActivity
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pn">[&lt;</span><span class="id">Service</span><span class="pn">;</span> <span class="id">IntentFilter</span><span class="pn">(</span><span class="pn">[|</span> <span class="s">"com.google.firebase.INSTANCE_ID_EVENT"</span> <span class="pn">|]</span><span class="pn">)</span><span class="pn">>]</span>
<span class="k">type</span> <span class="id">MyFirebaseIIDService</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span>
    <span class="k">inherit</span> <span class="id">FirebaseInstanceIdService</span><span class="pn">(</span><span class="pn">)</span>

    <span class="k">let</span> <span class="id">sendRegistrationToServer</span> <span class="id">token</span> <span class="o">=</span> 
        <span class="c">// Add custom implementation, as needed.</span>
        <span class="pn">(</span><span class="pn">)</span>
        
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnTokenRefresh</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    
        <span class="k">let</span> <span class="id">refreshedToken</span> <span class="o">=</span> <span class="id">FirebaseInstanceId</span><span class="pn">.</span><span class="id">Instance</span><span class="pn">.</span><span class="id">Token</span><span class="pn">;</span>
        <span class="s">"Refreshed token: "</span> <span class="o">+</span> <span class="id">refreshedToken</span> <span class="o">|></span> <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span>
        <span class="id">sendRegistrationToServer</span> <span class="id">refreshedToken</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
You will need to implement the step for sending the token to your backend server. Without the token, you won't be able to send messages to a single device.
In order to test this out, it can be a good idea to log the device token, so that we can pass that to the server.
Add the following to <code>OnCreate</code> in <code>MainActivity</code>:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="s">"InstanceID token: "</span> <span class="o">+</span> <span class="id">FirebaseInstanceId</span><span class="pn">.</span><span class="id">Instance</span><span class="pn">.</span><span class="id">Token</span> <span class="o">|></span> <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Build and run the app. Check the logs for the InstanceID token, we'll use it later on.

#### If you just want to see a notification
Run the app on your device, and you can use firebase console to a message to your device. It will show up as a notification. This is fine for testing, not useful for production.

Login into the firebase console, select your project. On the left menu select 'Cloud Messaging'. From there you can navigate
<img src="{{ site.baseurl }}/assets/img/fcm-cloud-messaging.png" alt="Smiley face" />

In production, this method is not useful, as it always a notification, and it is not reliable if the phone is in the background. There is some evidence to suggest this method only works if the app is in the foreground. This is because firebase sends these messages as 'notification' messages.
Let's fix our app to handle all types of messages.

#### Handling incoming messages
It turns out that for Android, things are never straight forward. The code so far does not work in all scenarios, or all devices.
We need to add a message handler, to intercept messages. I have added the following code above <code>MainActivity</code> in the Android project:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="pn">[&lt;</span><span class="id">Service</span><span class="pn">;</span> <span class="id">IntentFilter</span><span class="pn">(</span><span class="pn">[|</span> <span class="s">"com.google.firebase.MESSAGING_EVENT"</span> <span class="pn">|]</span><span class="pn">)</span><span class="pn">>]</span>
<span class="k">type</span>  <span class="id">MyFcmListenerService</span><span class="pn">(</span><span class="pn">)</span> <span class="k">as</span> <span class="id">this</span> <span class="o">=</span>
    <span class="k">inherit</span> <span class="id">FirebaseMessagingService</span><span class="pn">(</span><span class="pn">)</span>

    <span class="c">// Helper to convert the F# map type</span>
    <span class="k">let</span> <span class="id">dictToMap</span> <span class="pn">(</span><span class="id">dic</span> <span class="pn">:</span> <span onmouseout="hideTip(event, 'fs10', 23)" onmouseover="showTip(event, 'fs10', 23)" class="id">System</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs11', 24)" onmouseover="showTip(event, 'fs11', 24)" class="id">Collections</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs12', 25)" onmouseover="showTip(event, 'fs12', 25)" class="id">Generic</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs13', 26)" onmouseover="showTip(event, 'fs13', 26)" class="id">IDictionary</span><span class="pn">&lt;</span><span class="id">_</span><span class="pn">,</span><span class="id">_</span><span class="pn">></span><span class="pn">)</span> <span class="o">=</span> <span class="id">dic</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs14', 27)" onmouseover="showTip(event, 'fs14', 27)" class="id">Seq</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs15', 28)" onmouseover="showTip(event, 'fs15', 28)" class="id">map</span> <span class="pn">(</span><span class="pn">|</span><span onmouseout="hideTip(event, 'fs16', 29)" onmouseover="showTip(event, 'fs16', 29)" class="id">KeyValue</span><span class="pn">|</span><span class="pn">)</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs17', 30)" onmouseover="showTip(event, 'fs17', 30)" class="id">Map</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs18', 31)" onmouseover="showTip(event, 'fs18', 31)" class="id">ofSeq</span>

    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnMessageReceived</span><span class="pn">(</span><span class="id">message</span><span class="pn">:</span> <span class="id">RemoteMessage</span><span class="pn">)</span> <span class="o">=</span> 
    
        <span class="c">// Log the items</span>
        <span class="id">data</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs17', 32)" onmouseover="showTip(event, 'fs17', 32)" class="id">Map</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs19', 33)" onmouseover="showTip(event, 'fs19', 33)" class="id">iter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">key</span> <span class="id">value</span> <span class="k">-></span> <span onmouseout="hideTip(event, 'fs9', 34)" onmouseover="showTip(event, 'fs9', 34)" class="id">sprintf</span> <span class="s">"%s:%s"</span> <span class="id">key</span> <span class="id">value</span> <span class="o">|></span> <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span> <span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Whenever a message (more on sending the right message later) to the device, <code>OnMessageReceived</code> will be called. Our implementation is very simple, but good enough to tell us that things are working when connected to a debugger (reading the logs).

### Improving the testing method!
To test this setup now, we can't use the web console. Instead, we will send messages directly to the device. The simplest way of doing this is via curl on the command line.

Here is a sample curl request. You will need to update it with your API key (find this in the firebase console), and your device token (leave off the device token if you want to send to everyone):
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">curl</span> <span class="o">-</span><span class="id">X</span> <span class="id">POST</span> <span class="o">-</span><span class="id">H</span> <span class="s">"Authorization: key=Your_API_Key"</span> 
<span class="o">-</span><span class="id">H</span> <span class="s">"Content-Type: application/json"</span> 
    <span class="o">-</span><span class="id">d</span> <span class="id">'</span><span class="pn">{</span> 
            <span class="s">"registration_ids"</span><span class="pn">:</span> <span class="pn">[</span> 
                <span class="s">"device_token_from_app_logs"</span>
            <span class="pn">]</span><span class="pn">,</span> 
        <span class="s">"data"</span><span class="pn">:</span> <span class="pn">{</span> 
            <span class="s">"title"</span><span class="pn">:</span> <span class="s">"Hello"</span><span class="pn">,</span>
            <span class="s">"body"</span><span class="pn">:</span> <span class="s">"Hello, world"</span>
        <span class="pn">}</span><span class="pn">,</span>
        <span class="s">"priority"</span><span class="pn">:</span> <span class="s">"high"</span>
    <span class="pn">}</span><span class="id">'</span> <span class="id">https</span><span class="pn">:</span><span class="c">//fcm.googleapis.com/fcm/send</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Make sure your app is either running in the background, or in the foreground. It's best to be connected to the debugger. Send the curl request, and in the logs, you should see the <code>title</code> and <code>body</code> printed out.
If you have trouble receiving messages see the below for the troubleshooting section.
More on this whole background/foreground thing later!
We're done, at least for now on the app. On to refining the backend (and seeing in detail what we did with that curl request).

## The backend
As already stated, in order to send messages to a single device, the instanceID token is needed. For this the backend is really made up of two parts:
- receiving and storing the InstanceID Token
- sending a message to the device

### Receiving and storing the InstanceID token
I won't go into too much detail on the first point; it's really just building a REST API that accepts the value. A couple of key requirements are to make sure to <em>link the InstanceID token with a device identifier, and preferably the user</em>. This provides the link, so that at the application level on the server/app we can send a message to a user or device, lookup the InstanceID token, and then fire off the message via FCM.

## Sending messages to the device
Onto the second part, sending a message to the device. Google's documentation on this is at first a little misleading. There are two APIs and they are both valid, though one is label <code>legacy</code> (I thought we had semver for this).

If you have an existing GCM system, the only thing you need to do is change the base URL, from <code>gcm-http.googleapis.com/gcm/</code> to <code>fcm.googleapis.com/fcm/</code>. The rest of the API will still work. As Google states in their documentation, they are not removing them yet.

<blockquote>"The FCM equivalent of the GCM HTTP protocol is labelled "legacy" only to distinguish it clearly from the HTTP v1 API. The API is fully supported and Google has no near-term plan to deprecate it." Google - <a href="https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update-server-endpoints">https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update-server-endpoints</a></blockquote>
When we send the curl request, we used the legacy API, with the newly updated endpoint. According to Google's docs, everything is fine with this approach, nothing is going to be turned off.

### What's with legacy, should I upgrade?
The new API is titled 'FCM HTTP v1 API' (again it's called v1, but it's really the second version the legacy one was the first), and has two main benefits:

- Security is a little better (short-lived OAuth 2.0 tokens)
- Custom messages per platform (ie iOS vs Android)

The security tokens are nice, but not essential. For the platform messages, well, we're using Xamarin Forms so from a high level the business logic should be same, ie you probably don't need specific messages.

### If targeting the new API
Their main guide is here: <a href="https://firebase.google.com/docs/cloud-messaging/migrate-v1">https://firebase.google.com/docs/cloud-messaging/migrate-v1</a> it's simple if you're using Java.

If you're using .NET, the library and setup details are here: <a href="https://firebase.google.com/docs/admin/setup">https://firebase.google.com/docs/admin/setup</a>

### Sending messages with the 'Legacy' API
Full docs are <a href="https://firebase.google.com/docs/cloud-messaging/http-server-ref">here</a>,but that's rather verbose and abstract.
Here is the curl example again:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">curl</span> <span class="o">-</span><span class="id">X</span> <span class="id">POST</span> <span class="o">-</span><span class="id">H</span> <span class="s">"Authorization: key=API_ACCESS_KEY"</span> 
<span class="o">-</span><span class="id">H</span> <span class="s">"Content-Type: application/json"</span> 
    <span class="o">-</span><span class="id">d</span> <span class="id">'</span><span class="pn">{</span> 
            <span class="s">"registration_ids"</span><span class="pn">:</span> <span class="pn">[</span> 
                <span class="s">"YOUR_DEVICE_ID_TOKEN"</span>
            <span class="pn">]</span><span class="pn">,</span> 
        <span class="s">"data"</span><span class="pn">:</span> <span class="pn">{</span> <span class="c">// make sure this is 'data' and NOT notification</span>
            <span class="s">"message"</span><span class="pn">:</span> <span class="s">"Hello, World"</span> <span class="c">// &lt;- Add your custom data here. </span>
        <span class="pn">}</span><span class="pn">,</span>
        <span class="s">"priority"</span><span class="pn">:</span> <span class="s">"high"</span>
    <span class="pn">}</span><span class="id">'</span> <span class="id">https</span><span class="pn">:</span><span class="c">//fcm.googleapis.com/fcm/send // &lt;- make sure that this has fcm NOT GCM.</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
I have only shown the example that works by interpreting the message in the app - the <code>data</code> field in the payload. Google supports another field <code>notification</code> which is what the firebase console sends. Stick with <code>data</code> and handle the message locally on the phone. Everything will be fine (or better than using the notification version).
That should be all you need to get the backend going under the 'legacy' HTTP APIs. As already stated, Google is not turning these off, has no plans too yet.

### HTTP v1 API (The new API)
Because this API requires short-lived OAuth 2.0 tokens to send a message, it is much harder to test via curl. As a result, a library is generally used. See above to find the .NET library.
For the sake of completeness here is what a curl request would look like under the new 'FCM HTTP v1' API.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="id">curl</span> <span class="o">-</span><span class="id">X</span> <span class="id">POST</span> <span class="o">--</span><span class="id">header</span> <span class="s">"Authorization: key=Bearer Short_lived_oauth_token"</span>
    <span class="o">--</span><span class="id">Header</span> <span class="s">"Content-Type: application/json"</span> 
    <span class="o">-</span><span class="id">d</span> <span class="id">'</span><span class="pn">{</span>
            <span class="s">"to"</span><span class="pn">:</span><span class="s">"$YOUR_DEVICE_ID_TOKEN"</span><span class="pn">,</span>
            <span class="s">"notification"</span><span class="pn">:</span><span class="pn">{</span><span class="s">"message"</span><span class="pn">:</span><span class="s">"Hello, World"</span><span class="pn">}</span><span class="pn">,</span>
            <span class="s">"priority"</span><span class="pn">:</span><span class="n">10</span>
        <span class="pn">}</span><span class="id">'</span> <span class="id">https</span><span class="pn">:</span><span class="c">//fcm.googleapis.com/fcm/send</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
The big difference about this though, is that using the curl is really hard. To send this message, we can't use the API key. Instead, we need to generate an OAuth short-lived token - that's hard. For OAuth, use libraries.

## HTTP v1 API with .NET
Instead of using, curl, let's change to .NET, and use that. We'll need this to deploy to the backend anyway. To run this code locally, I'm going to use a simple console application running on dotnet core 2.*

Install the required NuGet package - FirebaseAdmin (at the time or writing the latest version was 1.2.0).
First we need to authenticate:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">secretFilename</span> <span class="o">=</span> <span class="s">"path/to/secretkey.json"</span>
<span class="k">let</span> <span class="id">firebaseApp</span> <span class="o">=</span> 
        <span class="id">FirebaseApp</span><span class="pn">.</span><span class="id">Create</span><span class="pn">(</span>
            <span class="id">AppOptions</span><span class="pn">(</span>
                <span class="id">Credential</span> <span class="o">=</span> <span class="id">GoogleCredential</span><span class="pn">.</span><span class="id">FromFile</span><span class="pn">(</span><span class="id">secretFilename</span><span class="pn">)</span><span class="pn">)</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
To get the JSON secret key, in firebase generate a private key pair. The file will then be offered as a download. This is a password, so treat it with the same care.
// Insert images to get private key
<img src="{{ site.baseurl }}/assets/img/Service-Accounts.png" alt="Smiley face"/>
<img src="{{ site.baseurl }}/assets/img/Generate-Key-Pair.png" alt="Smiley face"/>
Now that the app is authenticated, it's time to send a message:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">registrationToken</span> <span class="o">=</span> <span class="s">"device_token"</span> <span class="c">// Same as curl example. This comes from the device. In prod, this is sent to the server and pulled from storage (DB)</span>

<span class="c">// Create the message, with appropriate data. </span>
<span class="k">let</span> <span class="id">message</span> <span class="o">=</span>
    <span class="id">Message</span><span class="pn">(</span>
        <span onmouseout="hideTip(event, 'fs20', 35)" onmouseover="showTip(event, 'fs20', 35)" class="id">Data</span> <span class="o">=</span> <span class="pn">(</span><span onmouseout="hideTip(event, 'fs21', 36)" onmouseover="showTip(event, 'fs21', 36)" class="id">dict</span> <span class="pn">[</span>
            <span class="s">"title"</span><span class="pn">,</span> <span class="s">"Pushy"</span>
            <span class="s">"body"</span><span class="pn">,</span> <span class="s">"Pushy is alive and well"</span>
        <span class="pn">]</span> <span class="o">|></span> <span class="id">ReadOnlyDictionary</span><span class="pn">)</span><span class="pn">,</span>
        <span class="id">Token</span> <span class="o">=</span> <span class="id">registrationToken</span><span class="pn">)</span>

<span class="k">try</span>
    <span class="k">let</span> <span class="id">response</span> <span class="o">=</span> <span class="id">FirebaseMessaging</span><span class="pn">.</span><span class="id">DefaultInstance</span><span class="pn">.</span><span class="id">SendAsync</span><span class="pn">(</span><span class="id">message</span><span class="pn">)</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs22', 37)" onmouseover="showTip(event, 'fs22', 37)" class="id">Async</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs23', 38)" onmouseover="showTip(event, 'fs23', 38)" class="id">AwaitTask</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs22', 39)" onmouseover="showTip(event, 'fs22', 39)" class="id">Async</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs24', 40)" onmouseover="showTip(event, 'fs24', 40)" class="id">RunSynchronously</span>
    <span class="id">Console</span><span class="pn">.</span><span class="id">WriteLine</span><span class="pn">(</span><span class="s">"Successfully sent message: "</span> <span class="o">+</span> <span class="id">response</span><span class="pn">)</span>
<span class="k">with</span> <span class="id">e</span> <span class="k">-></span> <span onmouseout="hideTip(event, 'fs25', 41)" onmouseover="showTip(event, 'fs25', 41)" class="id">printfn</span> <span class="s">"Failed to send message:\n%s\n%s"</span> <span class="id">e</span><span class="pn">.</span><span class="id">Message</span> <span class="id">e</span><span class="pn">.</span><span class="id">StackTrace</span> 
</code></pre>
</td>
</tr>
</tbody>
</table>
That is all the code needed to generate a request <a href="https://gist.github.com/willsam100/8f249673187ffbfeffc26425bea19583">see here for full snippet</a>.
Use <code>dotnet run</code> and see messages sent to your device. Again, if you have trouble receiving messages see the below for the troubleshooting section.
Once everything is working, it is possible to make the required changes and deploy a variation of this code to production.

## Improving the mobile app experience
So far we only have messages showing as logs on the device, which is clearly not good enough for production. Let's fix that.
Again, Android never makes things easy. For Android 8.* and higher we need a notification channel. All up here is the code added to <code>MyFcmListenerService</code>
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">createNotificationChannel</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">let</span> <span class="id">CHANNEL_ID</span> <span class="o">=</span> <span class="s">"my_channel_01"</span><span class="pn">;</span> <span class="c">// The id of the channel. </span>
    <span class="k">let</span> <span class="id">name</span> <span class="o">=</span> <span class="s">"PushyChannel"</span>
    <span class="k">let</span> <span class="id">importance</span> <span class="o">=</span> <span class="id">Android</span><span class="pn">.</span><span class="id">App</span><span class="pn">.</span><span class="id">NotificationImportance</span><span class="pn">.</span><span class="id">High</span>
    <span class="k">let</span> <span class="id">mChannel</span> <span class="o">=</span> 
        <span class="k">new</span> <span class="id">NotificationChannel</span><span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">name</span><span class="pn">,</span> <span class="id">importance</span><span class="pn">,</span>
            <span class="id">LockscreenVisibility</span> <span class="o">=</span> <span class="id">NotificationVisibility</span><span class="pn">.</span><span class="id">Public</span><span class="pn">)</span>
    <span class="id">mChannel</span><span class="pn">.</span><span class="id">EnableVibration</span> <span class="k">true</span>
    <span class="id">mChannel</span><span class="pn">.</span><span class="id">EnableLights</span> <span class="k">true</span>

    <span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">mChannel</span><span class="pn">)</span>

<span class="k">let</span> <span class="id">showNotification</span> <span class="pn">(</span><span class="id">title</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 42)" onmouseover="showTip(event, 'fs4', 42)" class="id">string</span><span class="pn">)</span> <span class="pn">(</span><span class="id">body</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 43)" onmouseover="showTip(event, 'fs4', 43)" class="id">string</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span> <span class="s">"Showing notification"</span>
    <span class="k">let</span> <span class="id">intent</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Intent</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span onmouseout="hideTip(event, 'fs8', 44)" onmouseover="showTip(event, 'fs8', 44)" class="id">typeof</span><span class="pn">&lt;</span><span class="id">MainActivity</span><span class="pn">></span><span class="pn">)</span>
    <span class="k">let</span> <span class="id">pendingIntent</span> <span class="o">=</span> <span class="id">PendingIntent</span><span class="pn">.</span><span class="id">GetActivity</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="id">intent</span><span class="pn">,</span> <span class="id">PendingIntentFlags</span><span class="pn">.</span><span class="id">UpdateCurrent</span><span class="pn">)</span>

    <span class="k">let</span> <span class="id">notification</span> <span class="o">=</span> 
        <span class="k">if</span> <span class="pn">(</span><span class="id">Build</span><span class="pn">.</span><span class="id">VERSION</span><span class="pn">.</span><span class="id">SdkInt</span> <span class="pn">></span><span class="o">=</span> <span class="id">Android</span><span class="pn">.</span><span class="id">OS</span><span class="pn">.</span><span class="id">BuildVersionCodes</span><span class="pn">.</span><span class="id">O</span><span class="pn">)</span> <span class="k">then</span>
            <span class="k">let</span> <span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">mChannel</span><span class="pn">)</span> <span class="o">=</span> <span class="id">createNotificationChannel</span><span class="pn">(</span><span class="pn">)</span>
            <span class="id">NotificationManager</span><span class="pn">.</span><span class="id">FromContext</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">.</span><span class="id">CreateNotificationChannel</span> <span class="id">mChannel</span>
            <span class="pn">(</span><span class="k">new</span> <span class="id">Android</span><span class="pn">.</span><span class="id">App</span><span class="pn">.</span><span class="id">Notification</span><span class="pn">.</span><span class="id">Builder</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span class="id">CHANNEL_ID</span><span class="pn">)</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetSmallIcon</span><span class="pn">(</span><span class="id">Resources</span><span class="pn">.</span><span class="id">Drawable</span><span class="pn">.</span><span class="id">icon</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentTitle</span><span class="pn">(</span><span class="id">title</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentText</span><span class="pn">(</span><span class="id">body</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetChannelId</span><span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentIntent</span><span class="pn">(</span><span class="id">pendingIntent</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetAutoCancel</span><span class="pn">(</span><span class="k">true</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">Build</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>

        <span class="k">else</span>
            <span class="pn">(</span><span class="k">new</span> <span class="id">App</span><span class="pn">.</span><span class="id">NotificationCompat</span><span class="pn">.</span><span class="id">Builder</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetPriority</span><span class="pn">(</span><span class="id">App</span><span class="pn">.</span><span class="id">NotificationCompat</span><span class="pn">.</span><span class="id">PriorityDefault</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetSmallIcon</span><span class="pn">(</span><span class="id">Resources</span><span class="pn">.</span><span class="id">Drawable</span><span class="pn">.</span><span class="id">icon</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentTitle</span><span class="pn">(</span><span class="id">title</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentText</span><span class="pn">(</span><span class="id">body</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetContentIntent</span><span class="pn">(</span><span class="id">pendingIntent</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">SetAutoCancel</span><span class="pn">(</span><span class="k">true</span><span class="pn">)</span>
                <span class="pn">.</span><span class="id">Build</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>

    <span class="id">NotificationManager</span><span class="pn">.</span><span class="id">FromContext</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">.</span><span class="id">Notify</span><span class="pn">(</span><span class="n">1</span><span class="pn">,</span> <span class="id">notification</span><span class="pn">)</span><span class="pn">;</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This code handles all versions of Android. It will show the notification with the title, body and icon. When the user taps the notification, it will open the <code>MainActivity</code> and also clear the notification.
<blockquote>TIP: F# does not like circular dependencies but Android requires them here. See the full repo at the bottom of the post for how to structure to work around that. Obviously, you could just use C#.</blockquote>
To call the function, we can update <code>OnMessageReceived</code> method with the following:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnMessageReceived</span><span class="pn">(</span><span class="id">message</span><span class="pn">:</span> <span class="id">RemoteMessage</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">data</span> <span class="o">=</span> <span class="id">dictToMap</span> <span class="id">message</span><span class="pn">.</span><span class="id">Data</span>
    <span class="k">if</span> <span class="id">data</span><span class="pn">.</span><span class="id">ContainsKey</span> <span class="s">"title"</span> <span class="o">&amp;&amp;</span> <span class="id">data</span><span class="pn">.</span><span class="id">ContainsKey</span> <span class="s">"body"</span> <span class="k">then</span> 
        <span class="k">let</span> <span class="id">title</span><span class="pn">,</span> <span class="id">body</span> <span class="o">=</span> <span class="pn">(</span><span class="id">data</span><span class="pn">.</span><span class="pn">[</span><span class="s">"title"</span><span class="pn">]</span><span class="pn">,</span> <span class="id">data</span><span class="pn">.</span><span class="pn">[</span><span class="s">"body"</span><span class="pn">]</span><span class="pn">)</span>
        <span class="id">showNotification</span> <span class="id">title</span> <span class="id">body</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This is all that is required to always show a notification to the user, regardless of what state the app is in (Background or foreground).


### Are we done yet - Troubleshooting and common problems
What we have now, the firebase console, the app code and the backend server code is enough for a feature roll-out to send push notifications to a device. (I assume the app pushes the server token to the backend)

There's one small catch though - it's about Android.

Android does not send push notifications to an app that is not running (Foreground and background are fine). For most users, this is not a problem, but with some manufacturers they have modified Android to kill apps in the background.

To solve this, you will need to notify customers on certain phones, to change their settings to keep the app going. Here is a link to a table that shows how to navigate to the various settings for each phone.

<a href="https://github.com/invertase/react-native-firebase/issues/1238#issuecomment-410422872">https://github.com/invertase/react-native-firebase/issues/1238#issuecomment-410422872</a>
For more information on this, here a couple of posts:

<a href="https://onesignal.com/blog/manufacturers-interfere-with-reliable-notifications/">Manufacturers interfere with reliable notifications</a>

<a href="https://medium.freecodecamp.org/why-your-push-notifications-never-see-the-light-of-day-3fa297520793">Why your push notifications never see the light of day</a>

### What does it mean to kill the Android app?
To kill an app:
- Debug the app from Visual Studio, and then stop the app from Visual Studio
- Open Settings -> App -> Your App (Pushy) -> Force Stop

In these cases the app is dead. Sending push notifications won't work.
For most versions of Android, the following will NOT kill the app:
- Open the app, then press the HOME button
- Open the app, then press the BACK button, and see the launcher
- Clear the app from the RECENTS menu

In each of these scenarios the process for the app should still be running (The first scenario will still have an Activity too). It should stay running too if the app is running on a good version of Android. After a few days, the app should still be there, ready to receive a push notification.

A bad (in the context of a developer wanted to show a user push notification from an app) version of Android will not leave the app running in the background, and will instead kill the app. This is done in the name of battery saving (the settings to disable this are under the battery. How much it will save on battery is another question). See the section above for details on how to address this (as developers we must educate our users, as there are no APIs to disable the battery saving techniques).

## Don't show a notification when in the foreground
When the user is using the app, it seems a bit silly to send them a notification. Why not just update the page they are viewing, assuming it has the capacity to do that.

The code that follows is a very coupled way of doing this with Xamarin Forms.

### Detecting if the app is in the background.
The first step is to detect if the app is in the foreground or background. A good test of this is to see if we have an activity.
Using the NuGet package from James, <code>Plugin.CurrentActivity</code> we can find that out. It is also possible to find out what [Xamairn Forms] page is showing. 

First, we need to update <code>MainActivity</code> to store the Xamarin Forms app.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="c">// Using option types, because null was a billion dollar mistake</span>
<span class="k">let</span> <span class="k">mutable</span> <span class="id">formsApp</span><span class="pn">:</span> <span class="id">Pushy</span><span class="pn">.</span><span class="id">App</span> <span onmouseout="hideTip(event, 'fs26', 45)" onmouseover="showTip(event, 'fs26', 45)" class="id">option</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs27', 46)" onmouseover="showTip(event, 'fs27', 46)" class="id">None</span>
<span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">MainPage</span><span class="pn">(</span><span class="pn">)</span><span class="pn">:</span> <span class="id">Xamarin</span><span class="pn">.</span><span class="id">Forms</span><span class="pn">.</span><span class="id">Page</span> <span onmouseout="hideTip(event, 'fs28', 47)" onmouseover="showTip(event, 'fs28', 47)" class="id">Option</span> <span class="o">=</span> 
    <span class="id">formsApp</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs28', 48)" onmouseover="showTip(event, 'fs28', 48)" class="id">Option</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs29', 49)" onmouseover="showTip(event, 'fs29', 49)" class="id">map</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">x</span> <span class="k">-></span> <span class="id">x</span><span class="pn">.</span><span class="id">MainPage</span><span class="pn">)</span>

<span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnCreate</span> <span class="pn">(</span><span class="id">bundle</span><span class="pn">:</span> <span class="id">Bundle</span><span class="pn">)</span> <span class="o">=</span>
    <span class="c">// Rest of code omitted for brevity </span>

    <span class="k">let</span> <span class="id">app</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Pushy</span><span class="pn">.</span><span class="id">App</span> <span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 50)" onmouseover="showTip(event, 'fs6', 50)" class="id">isPlayServicesAvailable</span><span class="pn">)</span>
    <span class="id">formsApp</span> <span class="k">&lt;-</span> <span onmouseout="hideTip(event, 'fs30', 51)" onmouseover="showTip(event, 'fs30', 51)" class="id">Some</span> <span class="id">app</span>
    <span class="id">this</span><span class="pn">.</span><span class="id">LoadApplication</span> <span class="id">app</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
And now the checking to return if the app is in the background, or the showing <code>MainPage</code> from Xamairn Forms.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="pn">(</span><span class="pn">|</span><span class="id">HasForegroundUI</span><span class="pn">|</span><span class="id">InBackground</span><span class="pn">|</span><span class="pn">)</span> <span class="pn">(</span><span class="id">activity</span><span class="pn">:</span> <span class="id">Activity</span><span class="pn">)</span> <span class="o">=</span> 
    <span class="k">match</span> <span class="id">activity</span> <span class="k">with</span> 
    <span class="pn">|</span> <span class="o">:?</span> <span class="id">MainActivity</span> <span class="k">as</span> <span class="id">mainActivity</span> <span class="k">-></span> <span class="c">// Type check if have the MainActivity</span>
        <span class="k">match</span> <span class="id">mainActivity</span><span class="pn">.</span><span class="id">MainPage</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">with</span> 
        <span class="pn">|</span> <span onmouseout="hideTip(event, 'fs30', 52)" onmouseover="showTip(event, 'fs30', 52)" class="id">Some</span> <span class="id">mainPage</span> <span class="k">-></span>              <span class="c">// Check if we have a xamarin.Forms app</span>
            <span class="k">match</span> <span class="id">mainPage</span> <span class="k">with</span> 
            <span class="pn">|</span> <span class="o">:?</span> <span class="id">Pushy</span><span class="pn">.</span><span class="id">MainPage</span> <span class="k">as</span> <span class="id">mainPage</span> <span class="k">-></span> <span class="id">HasForegroundUI</span> <span class="id">mainPage</span>  <span class="c">// Check the the MainPage is showing</span>
            <span class="pn">|</span> <span class="id">_</span> <span class="k">-></span> <span class="id">InBackground</span>
        <span class="pn">|</span> <span onmouseout="hideTip(event, 'fs27', 53)" onmouseover="showTip(event, 'fs27', 53)" class="id">None</span> <span class="k">-></span> <span class="id">InBackground</span>
    <span class="pn">|</span> <span class="id">_</span> <span class="k">-></span> <span class="id">InBackground</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
This code does a series of type checks and must return either <code>InForeground</code> with the <code>MainPage</code> or <code>InBackground</code> with no data. The following code is how to call it:
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">handleNotification</span> <span class="id">title</span> <span class="id">body</span> <span class="o">=</span> 
    <span class="k">match</span> <span class="id">CrossCurrentActivity</span><span class="pn">.</span><span class="id">Current</span><span class="pn">.</span><span class="id">Activity</span> <span class="k">with</span> <span class="c">// Check the current activity</span>
    <span class="pn">|</span> <span class="id">HasForegroundUI</span> <span class="id">mainPage</span> <span class="k">-></span> <span class="id">mainPage</span><span class="pn">.</span><span class="id">HandleMessage</span> <span class="id">title</span> <span class="id">body</span> <span class="c">// We need to define this method</span>
    <span class="pn">|</span> <span class="id">InBackground</span> <span class="k">-></span> 
        <span class="c">// TODO: show notification</span>
        <span class="id">showNotification</span> <span class="id">title</span> <span class="id">body</span> <span class="c">// We will need to replace this later</span>

<span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnMessageReceived</span><span class="pn">(</span><span class="id">message</span><span class="pn">:</span> <span class="id">RemoteMessage</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">data</span> <span class="o">=</span> <span class="id">dictToMap</span> <span class="id">message</span><span class="pn">.</span><span class="id">Data</span>
    <span class="k">if</span> <span class="id">data</span><span class="pn">.</span><span class="id">ContainsKey</span> <span class="s">"title"</span> <span class="o">&amp;&amp;</span> <span class="id">data</span><span class="pn">.</span><span class="id">ContainsKey</span> <span class="s">"body"</span> <span class="k">then</span> 
        <span class="k">let</span> <span class="id">title</span><span class="pn">,</span> <span class="id">body</span> <span class="o">=</span> <span class="pn">(</span><span class="id">data</span><span class="pn">.</span><span class="pn">[</span><span class="s">"title"</span><span class="pn">]</span><span class="pn">,</span> <span class="id">data</span><span class="pn">.</span><span class="pn">[</span><span class="s">"body"</span><span class="pn">]</span><span class="pn">)</span>
        <span class="id">handleNotification</span> <span class="id">title</span> <span class="id">body</span> <span class="c">// Updated to call our new function</span>

    <span class="id">data</span> <span class="o">|></span> <span onmouseout="hideTip(event, 'fs17', 54)" onmouseover="showTip(event, 'fs17', 54)" class="id">Map</span><span class="pn">.</span><span onmouseout="hideTip(event, 'fs19', 55)" onmouseover="showTip(event, 'fs19', 55)" class="id">iter</span> <span class="pn">(</span><span class="k">fun</span> <span class="id">key</span> <span class="id">value</span> <span class="k">-></span> <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span> <span class="o">&lt;|</span> <span onmouseout="hideTip(event, 'fs9', 56)" onmouseover="showTip(event, 'fs9', 56)" class="id">sprintf</span> <span class="s">"%s:%s"</span> <span class="id">key</span> <span class="id">value</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Here pass in the current activity, and will just pattern match on the result of the function above.
<blockquote>Tip: If we miss a case here, the compiler will let us know. F# benefits.</blockquote>
When the app is in the background, we need to show the notification as we previously did. We could call <code>showNotification</code> as before, but that will cause us problems in a moment.

### Wiring up the MainPage
In <code>MainPage</code> (Forms) we can now define the <code>HandleMessage</code> method:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">HandleMessage</span> <span class="id">title</span> <span class="id">body</span> <span class="o">=</span> 
    <span class="c">// MainThread.BeginInvokeOnMainThread is from Xamarin.Essentials</span>
    <span class="id">MainThread</span><span class="pn">.</span><span class="id">BeginInvokeOnMainThread</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-></span> 
        <span class="id">label</span><span class="pn">.</span><span class="id">Text</span> <span class="k">&lt;-</span> <span onmouseout="hideTip(event, 'fs9', 57)" onmouseover="showTip(event, 'fs9', 57)" class="id">sprintf</span> <span class="s">"Received Message: %s"</span> <span class="id">body</span> <span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

### It works, but not for one case
With that code in place, run it and see that when the app is in the foreground it works, and when you press the BACK button and then send a message it works (shows a notification).

It FAILS, when you open the app, press the HOME button, and then send a notification. The UI is updated, instead of showing a notification, but the UI is not actually visible. Weird Android. We can fix this.

### Pull out the notification logic
We will need to pull out the notification logic, attach it to an interface, and allow it to be called from <code>MainPage</code> (as well as <code>OnMessageReceived</code>).
Above <code>MainPage</code> the following interface can be declared:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
<span class="l">2: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">ShowNotification</span> <span class="o">=</span> 
    <span class="k">abstract</span> <span class="k">member</span> <span class="id">ShowNotification</span><span class="pn">:</span> <span class="id">title</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 58)" onmouseover="showTip(event, 'fs4', 58)" class="id">string</span> <span class="k">-></span> <span class="id">body</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 59)" onmouseover="showTip(event, 'fs4', 59)" class="id">string</span> <span class="k">-></span> <span onmouseout="hideTip(event, 'fs31', 60)" onmouseover="showTip(event, 'fs31', 60)" class="id">unit</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
A class can then be added in Android to implement that, without notification code
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
<span class="l">43: </span>
<span class="l">44: </span>
<span class="l">45: </span>
<span class="l">46: </span>
<span class="l">47: </span>
<span class="l">48: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">NotificationHandler</span><span class="pn">(</span><span class="id">context</span><span class="pn">:</span><span class="id">Context</span><span class="pn">)</span> <span class="o">=</span> 

    <span class="k">let</span> <span class="id">createNotificationChannel</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
        <span class="k">let</span> <span class="id">CHANNEL_ID</span> <span class="o">=</span> <span class="s">"my_channel_01"</span><span class="pn">;</span><span class="c">// The id of the channel. </span>
        <span class="k">let</span> <span class="id">name</span> <span class="o">=</span> <span class="s">"FcmChannel"</span>
        <span class="k">let</span> <span class="id">importance</span> <span class="o">=</span> <span class="id">Android</span><span class="pn">.</span><span class="id">App</span><span class="pn">.</span><span class="id">NotificationImportance</span><span class="pn">.</span><span class="id">High</span>
        <span class="k">let</span> <span class="id">mChannel</span> <span class="o">=</span> 
            <span class="k">new</span> <span class="id">NotificationChannel</span><span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">name</span><span class="pn">,</span> <span class="id">importance</span><span class="pn">,</span>
                <span class="id">LockscreenVisibility</span> <span class="o">=</span> <span class="id">NotificationVisibility</span><span class="pn">.</span><span class="id">Public</span><span class="pn">)</span>
        <span class="id">mChannel</span><span class="pn">.</span><span class="id">EnableVibration</span> <span class="k">true</span>
        <span class="id">mChannel</span><span class="pn">.</span><span class="id">EnableLights</span> <span class="k">true</span>

        <span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">mChannel</span><span class="pn">)</span>

    <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowNotification</span> <span class="pn">(</span><span class="id">title</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 61)" onmouseover="showTip(event, 'fs4', 61)" class="id">string</span><span class="pn">)</span> <span class="pn">(</span><span class="id">body</span><span class="pn">:</span><span onmouseout="hideTip(event, 'fs4', 62)" onmouseover="showTip(event, 'fs4', 62)" class="id">string</span><span class="pn">)</span> <span class="o">=</span> 

        <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span> <span class="s">"Showing notification"</span>
        <span class="k">let</span> <span class="id">intent</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Intent</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span onmouseout="hideTip(event, 'fs8', 63)" onmouseover="showTip(event, 'fs8', 63)" class="id">typeof</span><span class="pn">&lt;</span><span class="id">MainActivity</span><span class="pn">></span><span class="pn">)</span>
        <span class="k">let</span> <span class="id">pendingIntent</span> <span class="o">=</span> <span class="id">PendingIntent</span><span class="pn">.</span><span class="id">GetActivity</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span class="n">0</span><span class="pn">,</span> <span class="id">intent</span><span class="pn">,</span> <span class="id">PendingIntentFlags</span><span class="pn">.</span><span class="id">UpdateCurrent</span><span class="pn">)</span>

        <span class="k">let</span> <span class="id">notification</span> <span class="o">=</span> 
            <span class="k">if</span> <span class="pn">(</span><span class="id">Build</span><span class="pn">.</span><span class="id">VERSION</span><span class="pn">.</span><span class="id">SdkInt</span> <span class="pn">></span><span class="o">=</span> <span class="id">Android</span><span class="pn">.</span><span class="id">OS</span><span class="pn">.</span><span class="id">BuildVersionCodes</span><span class="pn">.</span><span class="id">O</span><span class="pn">)</span> <span class="k">then</span>
                <span class="k">let</span> <span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">,</span> <span class="id">mChannel</span><span class="pn">)</span> <span class="o">=</span> <span class="id">createNotificationChannel</span><span class="pn">(</span><span class="pn">)</span>
                <span class="id">NotificationManager</span><span class="pn">.</span><span class="id">FromContext</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">.</span><span class="id">CreateNotificationChannel</span> <span class="id">mChannel</span>
                <span class="pn">(</span><span class="k">new</span> <span class="id">Android</span><span class="pn">.</span><span class="id">App</span><span class="pn">.</span><span class="id">Notification</span><span class="pn">.</span><span class="id">Builder</span><span class="pn">(</span><span class="id">context</span><span class="pn">,</span> <span class="id">CHANNEL_ID</span><span class="pn">)</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetSmallIcon</span><span class="pn">(</span><span class="id">Resources</span><span class="pn">.</span><span class="id">Drawable</span><span class="pn">.</span><span class="id">icon</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentTitle</span><span class="pn">(</span><span class="id">title</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentText</span><span class="pn">(</span><span class="id">body</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetChannelId</span><span class="pn">(</span><span class="id">CHANNEL_ID</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentIntent</span><span class="pn">(</span><span class="id">pendingIntent</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetAutoCancel</span><span class="pn">(</span><span class="k">true</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">Build</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>

            <span class="k">else</span>
                <span class="pn">(</span><span class="k">new</span> <span class="id">App</span><span class="pn">.</span><span class="id">NotificationCompat</span><span class="pn">.</span><span class="id">Builder</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetPriority</span><span class="pn">(</span><span class="id">App</span><span class="pn">.</span><span class="id">NotificationCompat</span><span class="pn">.</span><span class="id">PriorityDefault</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetSmallIcon</span><span class="pn">(</span><span class="id">Resources</span><span class="pn">.</span><span class="id">Drawable</span><span class="pn">.</span><span class="id">icon</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentTitle</span><span class="pn">(</span><span class="id">title</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentText</span><span class="pn">(</span><span class="id">body</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetContentIntent</span><span class="pn">(</span><span class="id">pendingIntent</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">SetAutoCancel</span><span class="pn">(</span><span class="k">true</span><span class="pn">)</span>
                    <span class="pn">.</span><span class="id">Build</span><span class="pn">(</span><span class="pn">)</span><span class="pn">;</span>

        <span class="id">NotificationManager</span><span class="pn">.</span><span class="id">FromContext</span><span class="pn">(</span><span class="id">context</span><span class="pn">)</span><span class="pn">.</span><span class="id">Notify</span><span class="pn">(</span><span class="n">1</span><span class="pn">,</span> <span class="id">notification</span><span class="pn">)</span><span class="pn">;</span>
    
    <span class="k">interface</span> <span class="id">ShowNotification</span> <span class="k">with</span> 
        <span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">ShowNotification</span> <span class="id">title</span> <span class="id">body</span> <span class="o">=</span>
            <span class="id">this</span><span class="pn">.</span><span class="id">ShowNotification</span> <span class="id">title</span> <span class="id">body</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
<blockquote>TIP: See the repo for full details, as this class also has circular dependencies which F# is helpfully discouraging us from creating.</blockquote>

### Checking if the UI is showing
To know if the app is actually showing on the UI we need to keep track of it (sadly this means state). This can be done with the <code>App</code> class from Xamairn.Forms
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
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">App</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 64)" onmouseover="showTip(event, 'fs6', 64)" class="id">isPlayServicesAvailable</span><span class="pn">)</span> <span class="o">=</span>
    <span class="k">inherit</span> <span class="id">Application</span><span class="pn">(</span><span class="pn">)</span>

    <span class="c">// Initlize MainPage with required depencies</span>
    <span class="k">do</span> <span class="k">base</span><span class="pn">.</span><span class="id">MainPage</span> <span class="k">&lt;-</span> <span class="id">MainPage</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 65)" onmouseover="showTip(event, 'fs6', 65)" class="id">isPlayServicesAvailable</span><span class="pn">,</span> <span class="id">App</span><span class="pn">.</span><span class="id">CheckInForeground</span><span class="pn">)</span>

    <span class="k">static</span> <span class="k">member</span> <span class="k">val</span> <span class="id">IsInForeground</span><span class="pn">:</span> <span onmouseout="hideTip(event, 'fs32', 66)" onmouseover="showTip(event, 'fs32', 66)" class="id">bool</span> <span class="o">=</span> <span class="k">false</span> <span class="k">with</span> <span class="id">get</span><span class="pn">,</span><span onmouseout="hideTip(event, 'fs33', 67)" onmouseover="showTip(event, 'fs33', 67)" class="id">set</span>

    <span class="k">static</span> <span class="k">member</span> <span class="id">CheckInForeground</span> <span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> 
        <span class="id">App</span><span class="pn">.</span><span class="id">IsInForeground</span>

    <span class="c">// State tracking</span>
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnStart</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">App</span><span class="pn">.</span><span class="id">IsInForeground</span> <span class="k">&lt;-</span> <span class="k">true</span>
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnResume</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">App</span><span class="pn">.</span><span class="id">IsInForeground</span> <span class="k">&lt;-</span> <span class="k">true</span>
    <span class="k">override</span> <span class="id">this</span><span class="pn">.</span><span class="id">OnSleep</span><span class="pn">(</span><span class="pn">)</span> <span class="o">=</span> <span class="id">App</span><span class="pn">.</span><span class="id">IsInForeground</span> <span class="k">&lt;-</span> <span class="k">false</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Here we are storing the apps state in a static variable <code>IsInForeground</code>, and then passing a method through to <code>MainPage</code> to read it <code>CheckInForeground</code>. Remember, that circular dependencies are bad, so passing method through is required (If using C# you could just read it off <code>App</code>).

### Wire up all the details
Over on <code>MainPage</code>, we can now wire up all the details. Here is the new implementation of the method
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">member</span> <span class="id">this</span><span class="pn">.</span><span class="id">HandleMessage</span> <span class="id">title</span> <span class="id">body</span> <span class="o">=</span> 
    <span class="c">// The Android Activity can still exist, but not be shown on the UI.</span>
    <span class="c">// only show when the activity (this page) is on the UI</span>
    <span class="k">if</span> <span class="id">isInForegound</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">then</span> 
        <span class="id">MainThread</span><span class="pn">.</span><span class="id">BeginInvokeOnMainThread</span> <span class="pn">(</span><span class="k">fun</span> <span class="pn">(</span><span class="pn">)</span> <span class="k">-></span> 
            <span class="id">label</span><span class="pn">.</span><span class="id">Text</span> <span class="k">&lt;-</span> <span onmouseout="hideTip(event, 'fs9', 68)" onmouseover="showTip(event, 'fs9', 68)" class="id">sprintf</span> <span class="s">"Received Message: %s"</span> <span class="id">body</span> <span class="pn">)</span>
    <span class="k">else</span> 
        <span class="id">Debug</span><span class="pn">.</span><span class="id">WriteLine</span> <span class="s">"Activity alive but in the background, showing notification"</span>
        <span class="id">showNotification</span><span class="pn">.</span><span class="id">ShowNotification</span> <span class="id">title</span> <span class="id">body</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
<code>isInForegound</code> is expected and this is the function we passed into the class. We also need a way to show the notifications, which is where our interface comes in. For that reason, we need another dependency to our class <code>showNotification</code>. Our <code>MainPage</code> constructor now looks like this:
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">type</span> <span class="id">MainPage</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 69)" onmouseover="showTip(event, 'fs6', 69)" class="id">isPlayServicesAvailable</span><span class="pn">,</span> <span class="id">isInForegound</span><span class="pn">,</span> <span class="id">showNotification</span><span class="pn">:</span><span class="id">ShowNotification</span><span class="pn">)</span> <span class="o">=</span>

<span class="c">// Required update given MainPage depdenecy</span>
<span class="k">type</span> <span class="id">App</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 70)" onmouseover="showTip(event, 'fs6', 70)" class="id">isPlayServicesAvailable</span><span class="pn">,</span> <span class="id">notificationHandler</span><span class="pn">:</span><span class="id">ShowNotification</span><span class="pn">)</span> <span class="o">=</span>
<span class="k">inherit</span> <span class="id">Application</span><span class="pn">(</span><span class="pn">)</span>

    <span class="k">do</span> <span class="k">base</span><span class="pn">.</span><span class="id">MainPage</span> <span class="k">&lt;-</span> <span class="id">MainPage</span><span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 71)" onmouseover="showTip(event, 'fs6', 71)" class="id">isPlayServicesAvailable</span><span class="pn">,</span> <span class="id">App</span><span class="pn">.</span><span class="id">CheckInForeground</span><span class="pn">,</span> <span class="id">notificationHandler</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Finally, we just need to update <code>MainActivity</code> so that it compiles:
<table class="pre">
<tbody>
<tr>
<td class="lines">
<pre class="fssnip"><span class="l">1: </span>
</pre>
</td>
<td class="snippet">
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">app</span> <span class="o">=</span> <span class="k">new</span> <span class="id">Pushy</span><span class="pn">.</span><span class="id">App</span> <span class="pn">(</span><span onmouseout="hideTip(event, 'fs6', 72)" onmouseover="showTip(event, 'fs6', 72)" class="id">isPlayServicesAvailable</span><span class="pn">,</span> <span class="id">NotificationHandler</span> <span class="id">this</span><span class="pn">)</span>
</code></pre>
</td>
</tr>
</tbody>
</table>

### Wire up the details:
We can now update the <code>handleNotification</code> to use the <code>NotificationHandler</code> instead of the function directly.
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
<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="id">handleNotification</span> <span class="id">title</span> <span class="id">body</span> <span class="o">=</span> 
    <span class="k">match</span> <span class="id">CrossCurrentActivity</span><span class="pn">.</span><span class="id">Current</span><span class="pn">.</span><span class="id">Activity</span> <span class="k">with</span> <span class="c">// Check the current activity</span>
    <span class="pn">|</span> <span class="id">HasForegroundUI</span> <span class="id">mainPage</span> <span class="k">-></span> <span class="id">mainPage</span><span class="pn">.</span><span class="id">HandleMessage</span> <span class="id">title</span> <span class="id">body</span> <span class="c">// We need to define this method</span>
    <span class="pn">|</span> <span class="id">InBackground</span> <span class="k">-></span> 
        <span class="k">let</span> <span class="id">notifHandler</span> <span class="o">=</span> <span class="id">NotificationHandler</span><span class="pn">(</span><span class="id">this</span><span class="pn">)</span>
        <span class="id">notifHandler</span><span class="pn">.</span><span class="id">ShowNotification</span> <span class="id">title</span> <span class="id">body</span>
</code></pre>
</td>
</tr>
</tbody>
</table>
Sadly we need all these items, as the activity may not exist (which holds the reference to Forms). The state tracking in Xamarin.Forms is the simplest way to know if the activity has is still alive in the background. If desired the state could be handled via <code>MainActivity</code> and Android's lifecycle methods. They have a tricky flow through, with <code>OnPause</code> vs <code>OnStart</code> etc I prefer to leverage Forms.

## Migration from GCM
If you haven't migrated from the GCM yet (it's being turned off very soon), there are TWO things you need to do differently to this post.

- When creating the firebase project, the name must come from the existing GCM project. This will mean the projectIDs remain the same.
- Change the backend URL to the one with FCM in it.

The firebase project should be created first. Everything else is backwards compatible. Though it's generally best to upgrade the app before server.

## Final Details
If you made it this far and found post the helpful, leave a comment, or share this post so it can help others.
Links:

<a href="https://gist.github.com/willsam100/d4775b42a09394fed25069ca18cfcd49">Gist core circular dependant classes</a>

<a href="https://github.com/willsam100/PushyFcm">Full Repo</a>
Related Links:

App

<a href="https://docs.microsoft.com/en-us/xamarin/android/data-cloud/google-messaging/&#10;https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update-server-endpoints&#10;https://firebase.google.com/docs/cloud-messaging/android/receive">https://docs.microsoft.com/en-us/xamarin/android/data-cloud/google-messaging/

[https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update-server-endpoints](https://developers.google.com/cloud-messaging/android/android-migrate-fcm#update-server-endpoints)

[https://firebase.google.com/docs/cloud-messaging/android/receive](https://firebase.google.com/docs/cloud-messaging/android/receive)
<a href="https://github.com/MicrosoftDocs/azure-docs/issues/14606&#10;https://medium.freecodecamp.org/why-your-push-notifications-never-see-the-light-of-day-3fa297520793&#10;https://github.com/firebase/quickstart-android/issues/41&#10;https://github.com/invertase/react-native-firebase/issues/1238#issuecomment-410422872&#10;https://onesignal.com/blog/manufacturers-interfere-with-reliable-notifications/">https://github.com/MicrosoftDocs/azure-docs/issues/14606

[https://medium.freecodecamp.org/why-your-push-notifications-never-see-the-light-of-day-3fa297520793](https://medium.freecodecamp.org/why-your-push-notifications-never-see-the-light-of-day-3fa297520793)

[https://github.com/firebase/quickstart-android/issues/41](https://github.com/firebase/quickstart-android/issues/41)

[https://github.com/invertase/react-native-firebase/issues/1238#issuecomment-410422872](https://github.com/invertase/react-native-firebase/issues/1238#issuecomment-410422872)

[https://onesignal.com/blog/manufacturers-interfere-with-reliable-notifications/](https://onesignal.com/blog/manufacturers-interfere-with-reliable-notifications/)


**Backend**

<a href="https://firebase.google.com/docs/cloud-messaging/server&#10;https://firebase.google.com/docs/cloud-messaging/migrate-v1&#10;https://firebase.google.com/docs/cloud-messaging/auth-server&#10;https://firebase.google.com/docs/cloud-messaging/http-server-ref&#10;https://firebase.google.com/docs/admin/setup">https://firebase.google.com/docs/cloud-messaging/server

[https://firebase.google.com/docs/cloud-messaging/migrate-v1](https://firebase.google.com/docs/cloud-messaging/migrate-v1)

[https://firebase.google.com/docs/cloud-messaging/auth-server](https://firebase.google.com/docs/cloud-messaging/auth-server)

[https://firebase.google.com/docs/cloud-messaging/http-server-ref](https://firebase.google.com/docs/cloud-messaging/http-server-ref)

[https://firebase.google.com/docs/admin/setup](https://firebase.google.com/docs/admin/setup)