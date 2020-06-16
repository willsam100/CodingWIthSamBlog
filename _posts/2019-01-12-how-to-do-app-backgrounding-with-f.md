---
layout: post
title: How to do app backgrounding with F#
date: 2019-01-12 23:01:35.000000000 +13:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: [Mobile]
tags: [Mobile, iOS, Android]
meta:
  _edit_last: '1'
  mashsb_twitter_handle: willsam100
  mashsb_timestamp: '1591194621'
  _yoast_wpseo_primary_category: '33'
  _yoast_wpseo_content_score: '30'
  mashsb_jsonshares: '{"facebook_total":1,"facebook_likes":1,"facebook_comments":0}'
  mashsb_shares: '1'
# author:
#   login: sam
#   email: willsam100@gmail.com
#   display_name: Sam Williams
#   first_name: Sam
#   last_name: Williams
permalink: "/2019/01/12/how-to-do-app-backgrounding-with-f/"
---
<!-- wp:html -->

## How to do app backgrounding with F#
F# + Xamarin, sounds simple. You built and app in record time (maybe with <a href="" title="https://fsprojects.github.io/Fabulous/, &quot;Fabulous on Github">Fabulous</a>).

It's app show-time, but instead of celebration you were met with UI lag or exceptions and crashes when the app is not showing.

What happened? Why did F# not solve all of my problems?
Mobile platforms (iOS and Android) have strict requirements about when and how some things are down. CPU and power (battery) are finite resources in a phone.

Background modes are going to be your new best friends.
Backgrounding can have several meanings in the context of mobile apps. Broadly these can fit into three levels.

## Level 1: UI thread is for UI work only
To keep the UI responsive, perform long-running tasks on a background thread. A long-running task is typically considered more than 100ms.
Over ~100ms, users think the app is lagging. Showing a spinner or progress bar while the task is running will make a huge difference.
<iframe width="50" height="50" src="https://lottiefiles.com/iframe/50-material-loader" frameborder="1" align="left"></iframe> <-- (like this spinner!)
This means:
- Loading a lot of data from many tables could easily take over 100ms - it's time to measure and monitor.
- All network requests must be done on a background thread.

## Level 2: Small updates while the app is showing
Level 2 is all about small infrequent tasks being done. Downloading and uploading content are examples of this background mode. A key aspect of level 2 is there is some flexibility around when the task is completed.
Another example is showing a local-notification when something has happened.
Both platforms have converged to allow simple updates in the background (many apps require this). For Android, this is the intention behind the Job Scheduler. iOS has background app refresh and a few other modes.

## Level 3: Continuously running when the app is not shown (the screen may be off too).
Both platforms restrict this heavily (prior to Android 8, this was not the case). These restrictions are to reduce unnecessary battery drain.  In most cases, it is possible to optimise the computation so that it only requires level 2 of backgrounding or fits in the allowed continuous background execution.
There are few cases when continuous backgrounding makes sense for user experience - e.g. Playing music or going for a run and seeing your location update etc. (so no Bitcoin mining <a href="" title="https://www.xda-developers.com/cryptocurrency-monero-ethereum-litecoin-bitcoin-mining-android-smartphones/">apps</a>)

**Instead of running a computation in the background, consider if you can solve the problem in another way.** eg calculate when the timer will expire, and schedule a local-notification to be shown.

If you really need to use level 3 backgrounding:
- for Android - use a foreground service. This will let the user know that your app is running in the background.
- For iOS, you’re pretty much out of luck. You can try, but you will need to use one of the background modes. If you abuse the mode, Apple may remove your app from the store.


## Navigating different levels of backgrounding with F#
### Level 1
F# has many benefits that make this simple to do. A simpler way of doing this is by using mono threads (.NET Task / F# async). Your options for doing this are:

- F# async
- <a href="" title="https://github.com/rspeele/TaskBuilder.fs">F# task computation library</a>
- a MailboxProcessor

If you want to keep your code clean after that, you could wrap the task in a thin layer of logic to get recurring updates. This will involve more code for abstraction, but will result in cleaner code.
### Level 2
Get acquainted with the native APIs. Unfortunately this mean you will have to embrace the OOP side of F#.

- If the task is mostly stateless, then you're in for a productivity gain. Put the shared code in an F# module in the core project. To enable background, setup up the platform specific code. Then, just call the F# module. No need for excessive objects and interfaces. 
- When there is app-state (or many dependencies), then objects and interfaces are going to be needed. Xamarin Forms Messaging Center (or some MVVM framework equivalent) should be used to glue them all together.

For platform APIs these documentation may help:
- For iOS take a peek at <a href="" title="https://developer.apple.com/documentation/uikit/core_app/managing_your_app_s_life_cycle/preparing_your_app_to_run_in_the_background/updating_your_app_with_background_app_refresh">background app refresh</a>
- For Android <a href="" title="https://docs.microsoft.com/en-us/xamarin/android/platform/android-job-scheduler">Job Scheduler</a>. Note it only works for Android 5.0+.

### Level 3
Again, using platform-specific APIs will be required. Shared code is very unlikely (though Messaging Center can allow for some). For iOS you will need to update your info.plist for the specific item that you need eg. playing music.

Android can do this through the use of a <a href="" title="https://developer.android.com/about/versions/oreo/background">foreground service</a>. Implement the required code and should be good to go – the user will know your app is running.

<a href="" title="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html">iOS documentation is here</a> and will require that you meet their T&amp;Cs. Background location is probably the most common. For <a href="" title="https://developer.apple.com/documentation/corelocation/getting_the_user_s_location/handling_location_events_in_the_background">iOS is here</a> and for <a href="" title="https://developer.android.com/about/versions/oreo/background-location-limits">Android is here</a>

## Go forth in the background
When you are met with a lagging or crashing app after using F#, don't be discouraged! Background modes will sort things out. Once you are familiar with the requirements of each platform for backgrounding, you could see how leaving out the mutation would actually help with writing less code - yay F#!
