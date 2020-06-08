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
 
```fsharp
type GooglePlayServicesAvailable = 
    | HasGooglePlayServices
    | RequiresUser of string // We can store the message to show the user
    | NoGooglePlayServices
```

The following code is in the <code>MainActivity</code> as a private method. Here we check Google Play Services and return one of the values from <code>GooglePlayServicesAvailable</code> type

```fsharp
let isPlayServicesAvailable() =
    let resultCode =
        GoogleApiAvailability.Instance.IsGooglePlayServicesAvailable(this)

    if resultCode = ConnectionResult.Success then
        HasGooglePlayServices
    else 
        if GoogleApiAvailability.Instance.IsUserResolvableError resultCode then
            GoogleApiAvailability.Instance.GetErrorString(resultCode) |> RequiresUser
        else NoGooglePlayServices
```

Now I can pass this directly into my forms app to be used from the application.

```fsharp
// In MainActivity.OnCreate
this.LoadApplication (new Pushy.App (isPlayServicesAvailable))

// Xamarin Forms App pass function through to MainPage
App(isPlayServicesAvailablen) =
    inherit Application(MainPage = MainPage(isPlayServicesAvailable))

// MainPage showing usage
type MainPage(isPlayServicesAvailable) =
inherit ContentPage()
let _ = base.LoadFromXaml(typeof<MainPage>)
let label = base.FindByName<Label>("Label")

do 
    let message = 
        match isPlayServicesAvailable () with 
        | HasGooglePlayServices -> "Pushy is ready for push"
        | NoGooglePlayServices -> "Pushy is not supported on this device"
        | RequiresUser x -> sprintf "Pushy needs your help: %s" x

    label.Text <- message 
```

Alternatively, I could wire this function up to Xamarin Forms' Dependency Service.

### Register the app with Google.
This was much harder when using GCM, now it is pretty easy. In the application, manifest add the following. No need to change anything:

```xml
<receiver
    android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver"
    android:exported="false" />
<receiver
    android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver"
    android:exported="true"
    android:permission="com.google.android.c2dm.permission.SEND">
    <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="${applicationId}" />
    </intent-filter>
</receiver>
```

<blockquote>Note: Because we are using Xamairn, updating the Application Manifest is required. Google's docs for Java app state that this can be removed.</blockquote>
Finally, we add the service to call the Google Nuget library. I have put this above MainActivity

```fsharp
[<Service; IntentFilter([| "com.google.firebase.INSTANCE_ID_EVENT" |])>]
type MyFirebaseIIDService() =
    inherit FirebaseInstanceIdService()

    let sendRegistrationToServer token = 
        // Add custom implementation, as needed.
        ()
        
    override this.OnTokenRefresh() = 
    
        let refreshedToken = FirebaseInstanceId.Instance.Token;
        "Refreshed token: " + refreshedToken |> Debug.WriteLine
        sendRegistrationToServer refreshedToken
```

You will need to implement the step for sending the token to your backend server. Without the token, you won't be able to send messages to a single device.
In order to test this out, it can be a good idea to log the device token, so that we can pass that to the server.
Add the following to <code>OnCreate</code> in <code>MainActivity</code>:
 
```fsharp
"InstanceID token: " + FirebaseInstanceId.Instance.Token |> Debug.WriteLine
```

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

```fsharp
[<Service; IntentFilter([| "com.google.firebase.MESSAGING_EVENT" |])>]
type  MyFcmListenerService() as this =
    inherit FirebaseMessagingService()

    // Helper to convert the F# map type
    let dictToMap (dic : System.Collections.Generic.IDictionary<_,_>) = dic |> Seq.map (|KeyValue|) |> Map.ofSeq

    override this.OnMessageReceived(message: RemoteMessage) = 
    
        // Log the items
        data |> Map.iter (fun key value -> sprintf "%s:%s" key value |> Debug.WriteLine )
```

Whenever a message (more on sending the right message later) to the device, <code>OnMessageReceived</code> will be called. Our implementation is very simple, but good enough to tell us that things are working when connected to a debugger (reading the logs).

### Improving the testing method!
To test this setup now, we can't use the web console. Instead, we will send messages directly to the device. The simplest way of doing this is via curl on the command line.

Here is a sample curl request. You will need to update it with your API key (find this in the firebase console), and your device token (leave off the device token if you want to send to everyone):

```terminal
curl -X POST -H "Authorization: key=Your_API_Key" 
-H "Content-Type: application/json" 
    -d '{ 
            "registration_ids": [ 
                "device_token_from_app_logs"
            ], 
        "data": { 
            "title": "Hello",
            "body": "Hello, world"
        },
        "priority": "high"
    }' https://fcm.googleapis.com/fcm/send
```


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

```terminal
curl -X POST -H "Authorization: key=API_ACCESS_KEY" 
-H "Content-Type: application/json" 
    -d '{ 
            "registration_ids": [ 
                "YOUR_DEVICE_ID_TOKEN"
            ], 
        "data": { // make sure this is 'data' and NOT notification
            "message": "Hello, World" // <- Add your custom data here. 
        },
        "priority": "high"
    }' https://fcm.googleapis.com/fcm/send // <- make sure that this has fcm NOT GCM.
```

I have only shown the example that works by interpreting the message in the app - the <code>data</code> field in the payload. Google supports another field <code>notification</code> which is what the firebase console sends. Stick with <code>data</code> and handle the message locally on the phone. Everything will be fine (or better than using the notification version).
That should be all you need to get the backend going under the 'legacy' HTTP APIs. As already stated, Google is not turning these off, has no plans too yet.

### HTTP v1 API (The new API)
Because this API requires short-lived OAuth 2.0 tokens to send a message, it is much harder to test via curl. As a result, a library is generally used. See above to find the .NET library.
For the sake of completeness here is what a curl request would look like under the new 'FCM HTTP v1' API.

```terminal
curl -X POST --header "Authorization: key=Bearer Short_lived_oauth_token"
    --Header "Content-Type: application/json" 
    -d '{
            "to":"$YOUR_DEVICE_ID_TOKEN",
            "notification":{"message":"Hello, World"},
            "priority":10
        }' https://fcm.googleapis.com/fcm/send
```


The big difference about this though, is that using the curl is really hard. To send this message, we can't use the API key. Instead, we need to generate an OAuth short-lived token - that's hard. For OAuth, use libraries.

## HTTP v1 API with .NET
Instead of using, curl, let's change to .NET, and use that. We'll need this to deploy to the backend anyway. To run this code locally, I'm going to use a simple console application running on dotnet core 2.*

Install the required NuGet package - FirebaseAdmin (at the time or writing the latest version was 1.2.0).
First we need to authenticate:

```fsharp
let secretFilename = "path/to/secretkey.json"
let firebaseApp = 
        FirebaseApp.Create(
            AppOptions(
                Credential = GoogleCredential.FromFile(secretFilename)))
```


To get the JSON secret key, in firebase generate a private key pair. The file will then be offered as a download. This is a password, so treat it with the same care.
// Insert images to get private key
<img src="{{ site.baseurl }}/assets/img/Service-Accounts.png" alt="Smiley face"/>
<img src="{{ site.baseurl }}/assets/img/Generate-Key-Pair.png" alt="Smiley face"/>
Now that the app is authenticated, it's time to send a message:

```fsharp
let registrationToken = "device_token" // Same as curl example. This comes from the device. In prod, this is sent to the server and pulled from storage (DB)

// Create the message, with appropriate data. 
let message =
    Message(
        Data = (dict [
            "title", "Pushy"
            "body", "Pushy is alive and well"
        ] |> ReadOnlyDictionary),
        Token = registrationToken)

try
    let response = FirebaseMessaging.DefaultInstance.SendAsync(message) |> Async.AwaitTask |> Async.RunSynchronously
    Console.WriteLine("Successfully sent message: " + response)
with e -> printfn "Failed to send message:\n%s\n%s" e.Message e.StackTrace 
```

That is all the code needed to generate a request <a href="https://gist.github.com/willsam100/8f249673187ffbfeffc26425bea19583">see here for full snippet</a>.
Use <code>dotnet run</code> and see messages sent to your device. Again, if you have trouble receiving messages see the below for the troubleshooting section.
Once everything is working, it is possible to make the required changes and deploy a variation of this code to production.

## Improving the mobile app experience
So far we only have messages showing as logs on the device, which is clearly not good enough for production. Let's fix that.
Again, Android never makes things easy. For Android 8.* and higher we need a notification channel. All up here is the code added to <code>MyFcmListenerService</code>

```fsharp
let createNotificationChannel () = 
    let CHANNEL_ID = "my_channel_01"; // The id of the channel. 
    let name = "PushyChannel"
    let importance = Android.App.NotificationImportance.High
    let mChannel = 
        new NotificationChannel(CHANNEL_ID, name, importance,
            LockscreenVisibility = NotificationVisibility.Public)
    mChannel.EnableVibration true
    mChannel.EnableLights true

    (CHANNEL_ID, mChannel)

let showNotification (title:string) (body:string) = 

    Debug.WriteLine "Showing notification"
    let intent = new Intent(context, typeof<MainActivity>)
    let pendingIntent = PendingIntent.GetActivity(context, 0, intent, PendingIntentFlags.UpdateCurrent)

    let notification = 
        if (Build.VERSION.SdkInt >= Android.OS.BuildVersionCodes.O) then
            let (CHANNEL_ID, mChannel) = createNotificationChannel()
            NotificationManager.FromContext(context).CreateNotificationChannel mChannel
            (new Android.App.Notification.Builder(context, CHANNEL_ID))
                .SetSmallIcon(Resources.Drawable.icon)
                .SetContentTitle(title)
                .SetContentText(body)
                .SetChannelId(CHANNEL_ID)
                .SetContentIntent(pendingIntent)
                .SetAutoCancel(true)
                .Build();

        else
            (new App.NotificationCompat.Builder(context))
                .SetPriority(App.NotificationCompat.PriorityDefault)
                .SetSmallIcon(Resources.Drawable.icon)
                .SetContentTitle(title)
                .SetContentText(body)
                .SetContentIntent(pendingIntent)
                .SetAutoCancel(true)
                .Build();

    NotificationManager.FromContext(context).Notify(1, notification);
```

This code handles all versions of Android. It will show the notification with the title, body and icon. When the user taps the notification, it will open the <code>MainActivity</code> and also clear the notification.
<blockquote>TIP: F# does not like circular dependencies but Android requires them here. See the full repo at the bottom of the post for how to structure to work around that. Obviously, you could just use C#.</blockquote>
To call the function, we can update <code>OnMessageReceived</code> method with the following:

```fsharp
override this.OnMessageReceived(message: RemoteMessage) = 

    let data = dictToMap message.Data
    if data.ContainsKey "title" && data.ContainsKey "body" then 
        let title, body = (data.["title"], data.["body"])
        showNotification title body
```

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

```fsharp
// Using option types, because null was a billion dollar mistake
let mutable formsApp: Pushy.App option = None
member this.MainPage(): Xamarin.Forms.Page Option = 
    formsApp |> Option.map (fun x -> x.MainPage)

override this.OnCreate (bundle: Bundle) =
    // Rest of code omitted for brevity 

    let app = new Pushy.App (isPlayServicesAvailable)
    formsApp <- Some app
    this.LoadApplication app
```

And now the checking to return if the app is in the background, or the showing <code>MainPage</code> from Xamairn Forms.

```fsharp
let (|HasForegroundUI|InBackground|) (activity: Activity) = 
    match activity with 
    | :? MainActivity as mainActivity -> // Type check if have the MainActivity
        match mainActivity.MainPage () with 
        | Some mainPage ->              // Check if we have a xamarin.Forms app
            match mainPage with 
            | :? Pushy.MainPage as mainPage -> HasForegroundUI mainPage  // Check the the MainPage is showing
            | _ -> InBackground
        | None -> InBackground
    | _ -> InBackground
```

This code does a series of type checks and must return either <code>InForeground</code> with the <code>MainPage</code> or <code>InBackground</code> with no data. The following code is how to call it:

```fsharp
let handleNotification title body = 
    match CrossCurrentActivity.Current.Activity with // Check the current activity
    | HasForegroundUI mainPage -> mainPage.HandleMessage title body // We need to define this method
    | InBackground -> 
        // TODO: show notification
        showNotification title body // We will need to replace this later

override this.OnMessageReceived(message: RemoteMessage) = 

    let data = dictToMap message.Data
    if data.ContainsKey "title" && data.ContainsKey "body" then 
        let title, body = (data.["title"], data.["body"])
        handleNotification title body // Updated to call our new function

    data |> Map.iter (fun key value -> Debug.WriteLine <| sprintf "%s:%s" key value)
```

Here pass in the current activity, and will just pattern match on the result of the function above.
<blockquote>Tip: If we miss a case here, the compiler will let us know. F# benefits.</blockquote>
When the app is in the background, we need to show the notification as we previously did. We could call <code>showNotification</code> as before, but that will cause us problems in a moment.

### Wiring up the MainPage
In <code>MainPage</code> (Forms) we can now define the <code>HandleMessage</code> method:

```fsharp 
member this.HandleMessage title body = 
    // MainThread.BeginInvokeOnMainThread is from Xamarin.Essentials
    MainThread.BeginInvokeOnMainThread (fun () -> 
        label.Text <- sprintf "Received Message: %s" body )
```

### It works, but not for one case
With that code in place, run it and see that when the app is in the foreground it works, and when you press the BACK button and then send a message it works (shows a notification).

It FAILS, when you open the app, press the HOME button, and then send a notification. The UI is updated, instead of showing a notification, but the UI is not actually visible. Weird Android. We can fix this.

### Pull out the notification logic
We will need to pull out the notification logic, attach it to an interface, and allow it to be called from <code>MainPage</code> (as well as <code>OnMessageReceived</code>).
Above <code>MainPage</code> the following interface can be declared:

```fsharp 
type ShowNotification = 
    abstract member ShowNotification: title:string -> body:string -> unit
```

A class can then be added in Android to implement that, without notification code

```fsharp
type NotificationHandler(context:Context) = 

    let createNotificationChannel () = 
        let CHANNEL_ID = "my_channel_01";// The id of the channel. 
        let name = "FcmChannel"
        let importance = Android.App.NotificationImportance.High
        let mChannel = 
            new NotificationChannel(CHANNEL_ID, name, importance,
                LockscreenVisibility = NotificationVisibility.Public)
        mChannel.EnableVibration true
        mChannel.EnableLights true

        (CHANNEL_ID, mChannel)

    member this.ShowNotification (title:string) (body:string) = 

        Debug.WriteLine "Showing notification"
        let intent = new Intent(context, typeof<MainActivity>)
        let pendingIntent = PendingIntent.GetActivity(context, 0, intent, PendingIntentFlags.UpdateCurrent)

        let notification = 
            if (Build.VERSION.SdkInt >= Android.OS.BuildVersionCodes.O) then
                let (CHANNEL_ID, mChannel) = createNotificationChannel()
                NotificationManager.FromContext(context).CreateNotificationChannel mChannel
                (new Android.App.Notification.Builder(context, CHANNEL_ID))
                    .SetSmallIcon(Resources.Drawable.icon)
                    .SetContentTitle(title)
                    .SetContentText(body)
                    .SetChannelId(CHANNEL_ID)
                    .SetContentIntent(pendingIntent)
                    .SetAutoCancel(true)
                    .Build();

            else
                (new App.NotificationCompat.Builder(context))
                    .SetPriority(App.NotificationCompat.PriorityDefault)
                    .SetSmallIcon(Resources.Drawable.icon)
                    .SetContentTitle(title)
                    .SetContentText(body)
                    .SetContentIntent(pendingIntent)
                    .SetAutoCancel(true)
                    .Build();

        NotificationManager.FromContext(context).Notify(1, notification);
    
    interface ShowNotification with 
        member this.ShowNotification title body =
            this.ShowNotification title body
```

<blockquote>TIP: See the repo for full details, as this class also has circular dependencies which F# is helpfully discouraging us from creating.</blockquote>

### Checking if the UI is showing
To know if the app is actually showing on the UI we need to keep track of it (sadly this means state). This can be done with the <code>App</code> class from Xamairn.Forms

```fsharp
type App(isPlayServicesAvailable) =
    inherit Application()

    // Initlize MainPage with required depencies
    do base.MainPage <- MainPage(isPlayServicesAvailable, App.CheckInForeground)

    static member val IsInForeground: bool = false with get,set

    static member CheckInForeground () = 
        App.IsInForeground

    // State tracking
    override this.OnStart() = App.IsInForeground <- true
    override this.OnResume() = App.IsInForeground <- true
    override this.OnSleep() = App.IsInForeground <- false
```

Here we are storing the apps state in a static variable <code>IsInForeground</code>, and then passing a method through to <code>MainPage</code> to read it <code>CheckInForeground</code>. Remember, that circular dependencies are bad, so passing method through is required (If using C# you could just read it off <code>App</code>).

### Wire up all the details
Over on <code>MainPage</code>, we can now wire up all the details. Here is the new implementation of the method

```fsharp
member this.HandleMessage title body = 
    // The Android Activity can still exist, but not be shown on the UI.
    // only show when the activity (this page) is on the UI
    if isInForegound () then 
        MainThread.BeginInvokeOnMainThread (fun () -> 
            label.Text <- sprintf "Received Message: %s" body )
    else 
        Debug.WriteLine "Activity alive but in the background, showing notification"
        showNotification.ShowNotification title body
```

<code>isInForegound</code> is expected and this is the function we passed into the class. We also need a way to show the notifications, which is where our interface comes in. For that reason, we need another dependency to our class <code>showNotification</code>. Our <code>MainPage</code> constructor now looks like this:

```fsharp
type MainPage(isPlayServicesAvailable, isInForegound, showNotification:ShowNotification) =

// Required update given MainPage depdenecy
type App(isPlayServicesAvailable, notificationHandler:ShowNotification) =
inherit Application()

    do base.MainPage <- MainPage(isPlayServicesAvailable, App.CheckInForeground, notificationHandler)
```

Finally, we just need to update <code>MainActivity</code> so that it compiles:

```fsharp
let app = new Pushy.App (isPlayServicesAvailable, NotificationHandler this)
```

### Wire up the details:
We can now update the <code>handleNotification</code> to use the <code>NotificationHandler</code> instead of the function directly.

```fsharp
let handleNotification title body = 
    match CrossCurrentActivity.Current.Activity with // Check the current activity
    | HasForegroundUI mainPage -> mainPage.HandleMessage title body // We need to define this method
    | InBackground -> 
        let notifHandler = NotificationHandler(this)
        notifHandler.ShowNotification title body
```

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