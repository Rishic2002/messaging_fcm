Firebase Cloud Messaging
========================

Firebase Cloud Messaging (FCM) is a cross-platform messaging solution that lets you reliably send messages at no cost.

Using FCM, you can notify a client app that new email or other data is available to sync. You can send notification messages to drive user re-engagement and retention. For use cases such as instant messaging, a message can transfer a payload of up to 4 KB to a client app.

Installation
------------

### 1\. Make sure to initialize Firebase

### [Follow this guide](https://firebase.flutter.dev/docs/overview) to install `firebase_core` and initialize Firebase if you haven't already.

### 2\. Add dependency

On the root of your Flutter project, run the following command to install the plugin:

flutter pub add firebase_messaging

### 3\. Android Integration

If you are using Flutter Android Embedding V2 (Flutter Version >= 1.12) then no additional integration steps are required for Android.

#### **Flutter Android Embedding V1**

For the Flutter Android Embedding V1, the background service must be provided a callback to register plugins with the background isolate. This is done by giving the `FlutterFirebaseMessagingBackgroundService` a callback to call your application's `onCreate` method.

In particular, its `Application` class:
'''

*// ...*

*import* io.flutter.plugins.firebase.messaging.FlutterFirebaseMessagingBackgroundService;

*public* *class* Application *extends* FlutterApplication *implements* PluginRegistrantCallback {

  *// ...*

  @Override

  *public* *void* onCreate() {

    *super*.onCreate();

    FlutterFirebaseMessagingBackgroundService.setPluginRegistrant(*this*);

  }

  @Override

  *public* *void* registerWith(PluginRegistry registry) {

    GeneratedPluginRegistrant.registerWith(registry);

  }

  *// ...*

}
'''
Which is usually reflected in the application's `AndroidManifest.xml`. E.g.:

    <application

        android:name=".Application"

        ...

Note: Not calling `FlutterFirebaseMessagingBackgroundService.setPluginRegistrant` will result in an exception being thrown when a message eventually comes through.

### 4\. Apple Integration

##### **CAUTION**

Firebase Messaging plugin will not work if you disable method swizzling. Please remove or set to `true` the `FirebaseAppDelegateProxyEnabled` in your Info.plist file if it exists.

iOS & macOS require additional configuration before you can start receiving messages through Firebase. Read the integration documentation on how to [setup iOS or macOS with Firebase Cloud Messaging](https://firebase.flutter.dev/docs/messaging/apple-integration).

### 5\. Rebuild your app

Once complete, rebuild your Flutter application:

flutter run

Cloud Messaging
===============

To start using the Cloud Messaging package within your project, import it at the top of your project files:

*import* 'package:firebase_messaging/firebase_messaging.dart';

Before using Firebase Cloud Messaging, you must first have ensured you have [initialized FlutterFire](https://firebase.flutter.dev/docs/overview#initializing-flutterfire).

To create a new Messaging instance, call the [`instance`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/instance.html) getter on [`FirebaseMessaging`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging-class.html):

FirebaseMessaging messaging = FirebaseMessaging.instance;

Messaging currently only supports usage with the default Firebase App instance.

Receiving messages
------------------

##### **IOS SIMULATORS**

FCM via APNs does not work on iOS Simulators. To receive messages & notifications a real device is required.

The Cloud Messaging package connects applications to the [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging) service. You can send message payloads directly to devices at no cost. Each message payload can be up to 4 KB in size, containing pre-defined or custom data to suit your applications requirements.

Common use-cases for using messages could be:

-   Displaying a notification (see [Notifications](https://firebase.flutter.dev/docs/messaging/notifications)).
-   Syncing message data silently on the device (e.g. via [shared_preferences](https://flutter.dev/docs/cookbook/persistence/key-value)).
-   Updating the application's UI.

To learn about how to send messages to devices from your own server setup, view the [Server Integration](https://firebase.flutter.dev/docs/messaging/server-integration)documentation.

Depending on a device's state, incoming messages are handled differently. To understand these scenarios & how to integrate FCM into your own application, it is first important to establish the various states a device can be in:

|

**State**

 |

**Description**

 |
| --- | --- |
|

Foreground

 |

When the application is open, in view & in use.

 |
|

Background

 |

When the application is open, however in the background (minimised). This typically occurs when the user has pressed the "home" button on the device, has switched to another app via the app switcher or has the application open on a different tab (web).

 |
|

Terminated

 |

When the device is locked or the application is not running. The user can terminate an app by "swiping it away" via the app switcher UI on the device or closing a tab (web).

 |

There are a few preconditions which must be met before the application can receive message payloads via FCM:

-   The application must have opened at least once (to allow for registration with FCM).
-   On iOS, if the user swipes away the application from app Switcher, it must be manually reopened again for background messages to start working again.
-   On Android, if the user force quits the app from device settings, it must be manually reopened again for messages to start working.
-   On iOS & macOS, you must have [correctly setup your project](https://firebase.flutter.dev/docs/messaging/apple-integration) to integrate with FCM and APNs.
-   On web, you must have requested a token (via `getToken`) with the key of a "Web Push certificate".

### Requesting permission (Apple & Web) 

On iOS, macOS & web, before FCM payloads can be received on your device, you must first ask the user's permission. Android applications are not required to request permission.

The `firebase_messaging` package provides a simple API for requesting permission via the [`requestPermission`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/requestPermission.html)method. This API accepts a number of named arguments which define the type of permissions you'd like to request, such as whether messaging containing notification payloads can trigger a sound or read out messages via Siri. By default, the method requests sensible default permissions. The reference API provides full documentation on what each permission is for.

To get started, call the method from your application (on iOS a native modal will be displayed, on web the browser's native API flow will be triggered):

FirebaseMessaging messaging = FirebaseMessaging.instance;

NotificationSettings settings = *await* messaging.requestPermission(

  alert: true,

  announcement: false,

  badge: true,

  carPlay: false,

  criticalAlert: false,

  provisional: false,

  sound: true,

);

print('User granted permission: ${settings.authorizationStatus}');

The `NotificationSettings` class returned from the request details information regarding the user's decision.

The `authorizationStatus` property can return a value which can be used to determine the user's overall decision:

-   `authorized`: The user granted permission.
-   `denied`: The user denied permission.
-   `notDetermined`: The user has not yet chosen whether to grant permission.
-   `provisional`: The user granted provisional permission (see [Provisional Permission](https://firebase.flutter.dev/docs/messaging/permissions#provisional-permission)).

On Android `authorizationStatus` will return `authorized` if the user has not disabled notifications for the app via the operating systems settings.

The other properties on `NotificationSettings` return whether a specific permission is enabled, disabled or not supported on the current device.

For further information, view the [Permissions](https://firebase.flutter.dev/docs/messaging/permissions) documentation.

Handling messages
-----------------

Once permission has been granted & the different types of device state have been understood, your application can now start to handle the incoming FCM payloads.

### Web Tokens

On the web, before a message can be sent to the browser you must do two things.

1.  Create an initial handshake with Firebase by passing in the public `vapidKey` to `messaging.getToken(vapidKey: 'KEY')` method. Head over to the [Firebase Console](https://console.firebase.google.com/project/_/settings/cloudmessaging) and create a new "Web Push Certificate". A key will be provided, which you can provide to the method:

FirebaseMessaging messaging = FirebaseMessaging.instance;

*// use the returned token to send messages to users from your custom server*

String token = *await* messaging.getToken(

  vapidKey: "BGpdLRs......",

);

1.  Create a `firebase-messaging-sw.js` file inside the `web/` directory in the root of your project. In your `web/index.html` file, please ensure this file is referenced and registered as a `serviceWorker` as demonstrated below:

  *<!-- ...other html setup. -->*

  <script>

    *if* ("serviceWorker" *in* navigator) {

      window.addEventListener("load", *function* () {

        navigator.serviceWorker.register("/firebase-messaging-sw.js");

      });

    }

  </script>

  *<!-- ...put the script at the bottom of the enclosing body tags. -->*

</body>

### Message types

A message payload can be viewed as one of three types:

1.  Notification only message: The payload contains a `notification` property, which will be used to [present a visible notification to the user](https://firebase.flutter.dev/docs/messaging/notifications).
2.  Data only message: Also known as a "silent message", this payload contains custom key/value pairs within the `data` property which can be used how you see fit. These messages are considered "low priority" (more on this later).
3.  Notification & Data messages: Payloads with both `notification` and `data` properties.

Based on your application's current state, incoming payloads require different implementations to handle them:

|  |

**Foreground**

 |

**Background**

 |

**Terminated**

 |
| --- | --- | --- | --- |
|

Notification

 |

`onMessage`

 |

`onBackgroundMessage`

 |

`onBackgroundMessage`

 |
|

Data

 |

`onMessage`

 |

`onBackgroundMessage` 

 |

`onBackgroundMessage` 

 |
|

Notification & Data

 |

`onMessage`

 |

`onBackgroundMessage`

 |

`onBackgroundMessage`

 |

Data only messages are considered low priority by devices when your application is in the background or terminated, and will be ignored. You can however explicitly increase the priority by sending additional properties on the FCM payload:

-   On Android, set the `priority` field to `high`.
-   On Apple (iOS & macOS), set the `content-available` field to `true`.

Since the sending of FCM payloads is custom to your own setup, it is best to read the official [FCM API reference](https://firebase.google.com/docs/cloud-messaging/concept-options) for your chosen Firebase Admin SDK.

### Foreground messages

To listen to messages whilst your application is in the foreground, listen to the [`onMessage`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/onMessage.html) stream.

FirebaseMessaging.onMessage.listen((RemoteMessage message) {

  print('Got a message whilst in the foreground!');

  print('Message data: ${message.data}');

  *if* (message.notification != *null*) {

    print('Message also contained a notification: ${message.notification}');

  }

});

The stream contains a `RemoteMessage`, detailing various information about the payload, such as where it was from, the unique ID, sent time, whether it contained a notification & more. Since the message was retrieved whilst your application is in the foreground, you can directly access your Flutter application's state & context.

#### **Foreground & Notification messages**

Notification messages which arrive whilst the application is in the foreground will not display a visible notification by default, on both Android & iOS. It is, however, possible to override this behavior:

-   On Android, you must create a "High Priority" notification channel.
-   On iOS, you can update the presentation options for the application.

More details on this are discussed in the [Notification: Foreground notifications](https://firebase.flutter.dev/docs/messaging/notifications#foreground-notifications) documentation.

### Background messages

The process of handling background messages is currently different on Android/Apple & web based platforms. We're working to see if it's possible to align these flows.

-       Android/Apple

-       Web

Handling messages whilst your application is in the background is a little different. Messages can be handled via the [`onBackgroundMessage`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/onBackgroundMessage.html) handler. When received, an isolate is spawned (Android only, iOS/macOS does not require a separate isolate) allowing you to handle messages even when your application is not running.

There are a few things to keep in mind about your background message handler:

1.  It must not be an anonymous function.
2.  It must be a top-level function (e.g. not a class method which requires initialization).

Future<*void*> _firebaseMessagingBackgroundHandler(RemoteMessage message) *async* {

  *// If you're going to use other Firebase services in the background, such as Firestore,*

  *// make sure you call `initializeApp` before using other Firebase services.*

  *await* Firebase.initializeApp();

  print("Handling a background message: ${message.messageId}");

}

*void* main() {

  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

  runApp(MyApp());

}

Since the handler runs in its own isolate outside your applications context, it is not possible to update application state or execute any UI impacting logic. You can, however, perform logic such as HTTP requests, perform IO operations (e.g. updating local storage), communicate with other plugins etc.

It is also recommended to complete your logic as soon as possible. Running long, intensive tasks impacts device performance and may cause the OS to terminate the process. If tasks run for longer than 30 seconds, the device may automatically kill the process.

#### **Notifications**

If your message is a notification one (includes a `notification` property), the Firebase SDKs will intercept this and display a visible notification to your users (assuming you have requested permission & the user has notifications enabled). Once displayed, the background handler will be executed (if provided).

To learn about how to handle user interaction with a notification, view the [Notifications](https://firebase.flutter.dev/docs/messaging/notifications) documentation.

#### **Debugging & Hot Reload**

FlutterFirebase Messaging does now support debugging and hot reloading for background isolates, but only if your main isolate is also being debugged, (e.g. run your application in debug and then background it by switching apps so it's no longer in the foreground).

For viewing additional logs on iOS, the "console.app" application on your Mac also displays system logs for your iOS device, including those from Flutter.

#### **Low priority messages**

As mentioned above, data only messages are classed as "low priority". Devices can throttle and ignore these messages if your application is in the background, terminated, or a variety of other conditions such as low battery or currently high CPU usage.

You should not rely on data only messages to be delivered. They should only be used to support your application's non-critical functionality, e.g. pre-fetching data so the next time the user opens your app the data is ready to be displayed and if the message never gets delivered then your app still functions and fetches data on open.

To help improve delivery, you can bump the priority of messages. Note: this still does not guarantee delivery.

Topics
------

Topics are a mechanism which allow a device to subscribe and unsubscribe from named PubSub channels, all managed via FCM. Rather than sending a message to a specific device by FCM token, you can instead send a message to a topic and any devices subscribed to that topic will receive the message.

Topics allow you to simplify FCM server integration as you do not need to keep a store of device tokens. There are, however, some things to keep in mind about topics:

-   Messages sent to topics should not contain sensitive or private information. Do not create a topic for a specific user to subscribe to.
-   Topic messaging supports unlimited subscriptions for each topic.
-   One app instance can be subscribed to no more than 2000 topics.
-   The frequency of new subscriptions is rate-limited per project. If you send too many subscription requests in a short period of time, FCM servers will respond with a 429 RESOURCE_EXHAUSTED ("quota exceeded") response. Retry with an exponential backoff.
-   A server integration can send a single message to multiple topics at once. This, however, is limited to 5 topics.

To learn more about how to send messages to devices subscribed to topics, view the [Send messages to topics](https://firebase.flutter.dev/docs/messaging/server-integration#send-messages-to-topics)documentation.

### Subscribing to topics

To subscribe a device, call the [`subscribeToTopic`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/subscribeToTopic.html) method with the topic name:

*// subscribe to topic on each app start-up*

*await* FirebaseMessaging.instance.subscribeToTopic('weather');

### Unsubscribing from topics

To unsubscribe from a topic, call the [`unsubscribeFromTopic`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/unsubscribeFromTopic.html) method with the topic name:

*await* FirebaseMessaging.instance.unsubscribeFromTopic('weather');

FCM via APNs Integration
========================

##### **CAUTION**

**This guide applies to both iOS & macOS Flutter apps, repeat each step for the platforms you require. E.g. if you support iOS & macOS then you will need to integrate twice, once per platform Xcode project.**

Integrating the Cloud Messaging plugin on iOS & macOS devices requires additional setup before your devices receive messages. There are also a number of prerequisites which are required to be able to enable messaging:

-   You must have an active [Apple Developer Account](https://developer.apple.com/membercenter/index.action).
-   For iOS; you must have a physical iOS device to receive messages.

-   Firebase Cloud Messaging integrates with the [Apple Push Notification service (APNs)](https://developer.apple.com/notifications/), however APNs only works with real devices.

Configuring your app
--------------------

Before your application can start to receive messages, you must explicitly enable "Push Notifications" and "Background Modes" within Xcode.

Open your project workspace file via Xcode (`/{ios|macos}/Runner.xcworkspace`). Once open, follow the steps below:

1.  Select your project.
2.  Select the project target.
3.  Select the "Signing & Capabilities" tab.

![Xcode - Navigate to Signing & Capabilities](blob:https://euangoddard.github.io/4b78c0f3-7bc0-4c2b-bd5a-8aa87fa1f254)

Xcode - Navigate to Signing & Capabilities

### Enable Push Notifications

Next the "Push Notifications" capability needs to be added to the project. This can be done via the "Capability" option on the "Signing & Capabilities" tab:

1.  Click on the "+ Capabilities" button.
2.  Search for "Push Notifications".

![Xcode - Enabling the Push Notification capability](blob:https://euangoddard.github.io/cd08ce11-d6ea-4a6a-8af4-698e1d10ee7c)

Xcode - Enabling the Push Notification capability

Once selected, the capability will be shown below the other enabled capabilities. If no option appears when searching, the capability may already be enabled.

### Enable Background Modes (iOS only) 

Next the "Background Modes" capability needs to be enabled, along with both the "Background fetch" and "Remote notifications" sub-modes. This can be added via the "Capability" option on the "Signing & Capabilities" tab:

1.  Click on the "+ Capabilities" button.
2.  Search for "Background Modes".

![Xcode - Enabling Background Modes capability](blob:https://euangoddard.github.io/153c1888-ae31-4004-aa14-fad837219825)

Xcode - Enabling Background Modes capability

Once selected, the capability will be shown below the other enabled capabilities. If no option appears when searching, the capability may already be enabled.

Now ensure that both the "Background fetch" and the "Remote notifications" sub-modes are enabled:

![Xcode - Enabling Background Modes](blob:https://euangoddard.github.io/16af086d-d82a-46c9-830d-edf288219c6d)

Xcode - Enabling Background Modes

Linking APNs with FCM
---------------------

Note: APNs is required for both `foreground` and `background` messaging to function correctly on iOS.

A few steps are required:

1.  [Registering a key](https://firebase.flutter.dev/docs/messaging/overview/#1-registering-a-key).
2.  [Registering an App Identifier](https://firebase.flutter.dev/docs/messaging/overview/#2-registering-an-app-identifier).
3.  [Generating a provisioning profile](https://firebase.flutter.dev/docs/messaging/overview/#3-generating-a-provisioning-profile).

All of these steps require you to have access to your [Apple Developer](https://developer.apple.com/membercenter/index.action) account. Once on the account, navigate to the [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) tab on the account sidebar:

![Apple Developer portal - Certificates, Identifiers & Profiles](blob:https://euangoddard.github.io/837062a0-d672-4245-975d-7a31ac91529e)

Apple Developer portal - Certificates, Identifiers & Profiles

### 1\. Registering a key

A key can be generated which gives the FCM full access over the Apple Push Notification service (APNs). On the "Keys" menu item, register a new key. The name of the key can be anything, however you must ensure the APNs service is enabled:

![Apple Developer portal - Enable Apple Push Notification (APNs)](blob:https://euangoddard.github.io/7b89c840-85b3-454f-9de0-cc8ac6ce5c53)

Apple Developer portal - Enable Apple Push Notification (APNs)

Click "Continue" & then "Save". Once saved, you will be presented with a screen displaying the private "Key ID" & the ability to download the key. Copy the ID, and download the file to your local machine:

![Copy Key ID & Download File](blob:https://euangoddard.github.io/015e94bf-9b85-4bbf-b2bc-9ffe87ae23e3)

Copy Key ID & Download File

The file & Key ID can now be added to your Firebase Project. On the [Firebase Console](https://console.firebase.google.com/project/_/settings/cloudmessaging), navigate to the "Project settings" and select the "Cloud Messaging" tab. Select your iOS application under the "iOS app configuration" heading.

Upload the downloaded file and enter the Key & Team IDs;

![Firebase Console - Upload key](blob:https://euangoddard.github.io/147e8595-d9d3-484a-aaea-1ac499c74ed2)

Firebase Console - Upload key

### 2\. Registering an App Identifier

For messaging to work when your app is built for production, you must create a new App Identifier which is linked to the application that you're developing.

On the Apple Developer portal:

1.  Click the "Identifiers" side menu item.
2.  Click the plus button to register a App Identifier.
3.  Select the "App IDs" option and click "Continue".
4.  Select the "App" type option and click "Continue".

![Apple Developer portal - Register a new identifier](blob:https://euangoddard.github.io/4fba6b6f-fb27-4bd1-8211-4f15de5b91b8)

Apple Developer portal - Register a new identifier

The "Register an App ID" page allows you to link the identifier to your application via the "Bundle ID". This is a unique string which was generated when starting your new Flutter project. Your Bundle ID can be obtained within Xcode, under the "General" tab for your project target:

![Xcode - View Bundle ID](blob:https://euangoddard.github.io/e2a75f61-4c57-48f8-aaf5-ff5be5bde1a1)

Xcode - View Bundle ID

On the Apple Developer portal "Register an App ID" page, follow these steps to create your identifier:

1.  Enter a description for the identifier.
2.  Enter the "Bundle ID" copied from Xcode.
3.  Scroll down and enable the "Push Notifications" capability (along with any others your app uses).

![Apple Developer portal - Register an App ID](blob:https://euangoddard.github.io/93f811c4-6bd3-4c1c-812f-117368d5a26f)

Apple Developer portal - Register an App ID

Save the identifier, it'll be used when creating a provisioning profile in the next step.

### 3\. Generating a provisioning profile

A provisioning profile enables signed communicate between Apple and your application. Since messaging can only be used on real devices, a signed certificate ensures that the app being installed on a device is genuine and has the correct permissions enabled.

On the "Profiles" menu item, register a new Profile. Select the "iOS App Development" (or macOS) checkbox and click "Continue".

If you followed [Step 2](https://firebase.flutter.dev/docs/messaging/overview/#2-registering-an-app-identifier) correctly, your App Identifier will be available in the drop down provided:

![Apple Developer portal - Select App ID](blob:https://euangoddard.github.io/428d4b96-a394-482a-8b24-0bee7fb7684d)

Apple Developer portal - Select App ID

Click "Continue". On the next screen you will be presented with the Certificates on your Apple account. Select the user certificates that you wish to assign this provisioning profile too. If you have not yet created a Certificate, you must set one up on your account.

To create a new Certificate, follow the [Apple documentation](https://help.apple.com/developer-account/#/devbfa00fef7). Once the Certificate has been downloaded, upload it to the Apple Developer console via the "Certificates" menu item.

The created provisioning profile can now be used when building your application (in both debug and release mode) onto a real device (using Xcode). Back within Xcode, select your project target and select the "Signing & Capabilities" tab. If Xcode (via Preferences) is linked to your Apple Account, Xcode can automatically sync the profile created above. Otherwise, you must manually add the profile from the Apple Developer console:

1.  Select the project (usually called 'Runner') in the left hand side project navigator.
2.  Select the project target under the 'Targets' heading, usually called 'Runner'.
3.  Select the Signing & Capabilities tab.
4.  Assign the provisioning profile in the 'Signing' section.

![Xcode - Assign provisioning profile](blob:https://euangoddard.github.io/96e9c0d5-f51f-4642-9ebe-aa462fe82a33)

Xcode - Assign provisioning profile

(Advanced, Optional) Allowing Notification Images
-------------------------------------------------

If you want to know more about the specifics of this setup read the [official Firebase docs](https://firebase.google.com/docs/cloud-messaging/ios/send-image).

On Apple devices, in order for incoming FCM [Notifications](https://firebase.flutter.dev/docs/messaging/notifications) to display images from the FCM payload, you must add an additional notification service extension. This is not a required step.

### Step 1 - Add a notification service extension[#](https://firebase.flutter.dev/docs/messaging/overview/#step-1---add-a-notification-service-extension "Direct link to heading")

-   From Xcode top menu go to: File > New > Target...
-   A modal will present a list of possible targets, scroll down or use the filter to select "Notification Service Extension". Press Next.
-   Add a product name (use `ImageNotification` to follow along), set `Language` to `Objective-C` and click Finish.
-   Enable the scheme by clicking Activate.

![Xcode - Add a notification service extension](blob:https://euangoddard.github.io/cde11d41-682d-46ea-9dd7-ff267573b034)

Xcode - Add a notification service extension

### Step 2 - Add target to the Podfile

Ensure that your new extension has access to Firebase/Messaging pod by adding it in the Podfile:

-   From the Navigator open the Podfile: Pods > Podfile
-   Scroll down to the bottom of the file and add:

target 'ImageNotification' *do*

  use_frameworks!

  pod 'Firebase/Messaging'

*end*

-   Install or update your pods using `pod install` from the `ios` and/or `macos` directory.

![Xcode - Add target to the Podfile](blob:https://euangoddard.github.io/79eeab31-472a-4585-b76e-f48d94ae0f8f)

Xcode - Add target to the Podfile

### Step 3 - Use the extension helper

At this point everything should still be running normally. This is the final step which is invoking the extension helper.

-   From the navigator select your `ImageNotification` extension
-   Open the `NotificationService.m` file
-   At the top of the file import `FirebaseMessaging.h` right after the `NotificationService.h` as shown below:

#import "NotificationService.h"

+ #import "FirebaseMessaging.h"

-   Replace everything from line 24 to 27 with the extension helper:

- // Modify the notification content here...

- self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];

- self.contentHandler(self.bestAttemptContent);

+ [[FIRMessaging extensionHelper] populateNotificationContent:self.bestAttemptContent withContentHandler:contentHandler];

![Xcode - Use the extension helper](blob:https://euangoddard.github.io/5bdf1e35-c235-4a37-a56d-97eda15f0719)

Xcode - Use the extension helper

### Step 4

Your device can now display images in a notification by specifying the `imageUrl` option in your FCM payload. Keep in mind that a 300KB max image size is enforced by the device.

Permissions
===========

Before diving into requesting notification permissions from your users, it is important to understand how Apple and web platforms handle permissions.

Notifications cannot be shown to users if the user has not "granted" your application permission. The overall notification permission of a single application can be either be "not determined", "granted" or "declined". Upon installing a new application, the default status is "not determined".

In order to receive a "granted" status, you must request permission from your user (see below). The user can either accept or decline your request to grant permissions. If granted, notifications will be displayed based on the permission settings which were requested.

If the user declines the request, you cannot re-request permission, trying to request permission again will immediately return a "denied" status without any user interaction - instead the user must manually enable notification permissions from the device Settings UI (or via a custom UI).

Requesting permissions[#](https://firebase.flutter.dev/docs/messaging/overview/#requesting-permissions "Direct link to heading")
--------------------------------------------------------------------------------------------------------------------------------

As explained in the [Usage documentation](https://firebase.flutter.dev/docs/messaging/usage), permission must be requested from your users in order to display remote notifications from FCM, via the [`requestPermission`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/requestPermission.html) API:

FirebaseMessaging messaging = FirebaseMessaging.instance;

NotificationSettings settings = *await* messaging.requestPermission(

  alert: true,

  announcement: false,

  badge: true,

  carPlay: false,

  criticalAlert: false,

  provisional: false,

  sound: true,

);

*if* (settings.authorizationStatus == AuthorizationStatus.authorized) {

  print('User granted permission');

} *else* *if* (settings.authorizationStatus == AuthorizationStatus.provisional) {

  print('User granted provisional permission');

} *else* {

  print('User declined or has not accepted permission');

}

On Apple based platforms, once a permission request has been handled by the user (authorized or denied), it is not possible to re-request permission. The user must instead update permission via the device Settings UI:

-   If the user denied permission altogether, they must enable app permission fully.
-   If the user accepted requested permission (without sound), they must specifically enable the sound option themselves.

Permission settings
-------------------

Although overall notification permission can be granted, the permissions can be further broken down into 'settings'.

Settings are used by the device to control notifications behavior, for example alerting the user with sound. When requesting permission, you can provide arguments if you wish to override the defaults. This is demonstrated in the above example.

The full list of permission settings can be seen in the table below along with their default values:

|

**Permission**

 |

**Default**

 |

**Description**

 |
| --- | --- | --- |
|

`alert`

 |

`true`

 |

Sets whether notifications can be displayed to the user on the device.

 |
|

`announcement`

 |

`false`

 |

If enabled, Siri will read the notification content out when devices are connected to AirPods.

 |
|

`badge`

 |

`true`

 |

Sets whether a notification dot will appear next to the app icon on the device when there are unread notifications.

 |
|

`carPlay`

 |

`true`

 |

Sets whether notifications will appear when the device is connected to [CarPlay](https://www.apple.com/ios/carplay/).

 |
|

`provisional`

 |

`false`

 |

Sets whether provisional permissions are granted. See [Provisional authorization](https://firebase.flutter.dev/docs/messaging/overview/#provisional-authorization) for more information.

 |
|

`sound`

 |

`true`

 |

Sets whether a sound will be played when a notification is displayed on the device.

 |

The settings provided will be stored by the device and will be visible in the iOS Settings UI for your application.

You can also read the current permissions without attempting to request them via the [`getNotificationSettings`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/getNotificationSettings.html)method:

NotificationSettings settings = *await* messaging.getNotificationSettings();

Provisional authorization
-------------------------

Devices on iOS 12+ can use provisional authorization. This type of permission system allows for notification permission to be instantly granted without displaying a dialog to your user. The permission allows notifications to be displayed quietly (only visible within the device notification center).

To enable provision permission, set the `provisional` argument to `true` when requesting permission:

NotificationSettings settings = *await* messaging.requestPermission(

  provisional: true,

);

When a notification is displayed on the device, the user will be presented with several actions prompting to keep receiving notifications quietly, enable full notification permission or turn them off:

![iOS Provisional Notification Example](blob:https://euangoddard.github.io/e665375a-5e6e-4415-ac99-74c1a276bd32)

iOS Provisional Notification Example

Notifications
=============

Notifications are an important tool used on the majority of applications, aimed at improve user experience & used to engage users with your application. The Cloud Messaging module provides basic support for displaying and handling notifications.

##### **IOS SIMULATORS**

FCM via APNs does not work on iOS Simulators. To receive messages & notifications a real device is required.

Displaying notifications
------------------------

As mentioned in the [Usage](https://firebase.flutter.dev/docs/messaging/usage) documentation, message payloads can include a `notification` property which the Firebase SDKs intercept and attempt to display a visible notification to users.

The default behavior on all platforms is to display a notification only when the app is in the background or terminated. If required, you can override this behavior by following the [Foreground Notifications](https://firebase.flutter.dev/docs/messaging/overview/#foreground-notifications) documentation.

FCM based notifications provide the basics for many use cases, such as displaying text and images. They do not support advanced notifications such as actions, styling, foreground service notifications etc. To learn more, view the [Foreground Notifications](https://firebase.flutter.dev/docs/messaging/overview/#foreground-notifications) documentation.

The documentation below outlines a few different ways you can start to send notification based messages to your devices.

### Via Firebase Console

The [Firebase Console](https://console.firebase.google.com/project/_/notification) provides a simple UI to allow devices to display a notification. Using the console, you can:

-   Send a basic notification with custom text and images.
-   Target applications which have been added to your project.
-   Schedule notifications to display at a later date.
-   Send recurring notifications.
-   Assign conversion events for your analytical tracking.
-   A/B test user interaction (called "experiments").
-   Test notifications on your development devices.

The Firebase Console automatically sends a message to your devices containing a notification property which is handled by the Firebase Cloud Messaging package. See [Handling Interaction](https://firebase.flutter.dev/docs/messaging/overview/#handling-interaction) to learn about how to support user interaction.

### Via Admin SDKs

Using one of the [various Firebase Admin SDKs](https://firebase.google.com/docs/reference/admin), you can send customized data payloads to your devices from your own servers.

For example, when using the [`firebase-admin`] package in a Node.js environment to send messages from a server, a `notification` property can be added to the message payload:

*const* admin = require("firebase-admin");

*await* admin.messaging().sendMulticast({

  tokens: ["token_1", "token_2"],

  notification: {

    title: "Weather Warning!",

    body: "A new weather warning has been issued for your location.",

    imageUrl: "https://my-cdn.com/extreme-weather.png",

  },

});

To learn more about how to integrate Cloud Messaging with your own setup, read the [Server Integration](https://firebase.flutter.dev/docs/messaging/server-integration)documentation.

### Via REST

If you are unable to use a [Firebase Admin SDK](https://firebase.google.com/docs/reference/admin), Firebase also provides support for sending messages to devices via HTTP POST requests:

POST https://fcm.googleapis.com/v1/projects/myproject-b5ae1/messages:send HTTP/1.1

Content-Type: application/json

Authorization: Bearer ya29.ElqKBGN2Ri_Uz...HnS_uNreA

{

   "message":{

      "token":"token_1",

      "data":{},

      "notification":{

        "title":"FCM Message",

        "body":"This is an FCM notification message!",

      }

   }

}

To learn more about the REST API, view the [Firebase documentation](https://firebase.google.com/docs/cloud-messaging/send-message#rest), and select the "REST" tab under the code examples.

Handling Interaction
--------------------

Since notifications are a visible cue, it is common for users to interact with it (by pressing them). The default behavior on both Android & iOS is to open the application. If the application is terminated it will be started, if it is in the background it will be brought to the foreground.

Depending on the content of a notification, you may wish to handle the users interaction when the application opens. For example, if a new chat message is sent via a notification and the user presses it, you may want to open the specific conversation when the application opens.

The `firebase-messaging` package provides two ways to handle this interaction:

1.  `getInitialMessage()`: If the application is opened from a terminated state a `Future` containing a `RemoteMessage` will be returned. Once consumed, the `RemoteMessage` will be removed.
2.  `onMessageOpenedApp`: A `Stream` which posts a `RemoteMessage` when the application is opened from a background state.

It is recommended that both scenarios are handled to ensure a smooth UX for your users. The code example below outlines how this can be achieved:

*class* Application *extends* StatefulWidget {

  @override

  State<StatefulWidget> createState() => _Application();

}

*class* _Application *extends* State<Application> {

  *// It is assumed that all messages contain a data field with the key 'type'*

  Future<*void*> setupInteractedMessage() *async* {

    *// Get any messages which caused the application to open from*

    *// a terminated state.*

    RemoteMessage? initialMessage =

        *await* FirebaseMessaging.instance.getInitialMessage();

    *// If the message also contains a data property with a "type" of "chat",*

    *// navigate to a chat screen*

    *if* (initialMessage != *null*) {

      _handleMessage(initialMessage);

    }

    *// Also handle any interaction when the app is in the background via a*

    *// Stream listener*

    FirebaseMessaging.onMessageOpenedApp.listen(_handleMessage);

  }

  *void* _handleMessage(RemoteMessage message) {

    *if* (message.data['type'] == 'chat') {

      Navigator.pushNamed(context, '/chat',

        arguments: ChatArguments(message),

      );

    }

  }

  @override

  *void* initState() {

    *super*.initState();

    *// Run code required to handle interacted messages in an async function*

    *// as initState() must not be async*

    setupInteractedMessage();

  }

  @override

  Widget build(BuildContext context) {

    *return* Text("...");

  }

}

How you handle interaction depends on your application setup, however the example above shows a basic example of using a StatefulWidget.

Advanced Usage
--------------

The below documentation outlines some advanced usage of notifications.

### Foreground Notifications

Foreground notifications (also known as "heads up") are those which display for a brief period of time above existing applications, and should be used for important events.

Android & iOS have different behaviors when handling notifications whilst applications are in the foreground so keep this in mind whilst developing.

#### **iOS Configuration**

Enabling foreground notifications is generally a straightforward process. Call the `setForegroundNotificationPresentationOptions` method with named arguments:

*await* FirebaseMessaging.instance.setForegroundNotificationPresentationOptions(

  alert: true, *// Required to display a heads up notification*

  badge: true,

  sound: true,

);

Set all values back to `false` to revert to the default functionality.

#### **Android configuration**

Android handles incoming notifications differently based on a few different factors:

1.  If the application is in the background or terminated, the assigned "Notification Channel" is used to determine how the notification is displayed.
2.  If the application is currently in the foreground, a visible notification is not presented.

##### **Notification Channels**

On Android, notification messages are sent to [Notification Channels](https://developer.android.com/guide/topics/ui/notifiers/notifications.html#ManageChannels) which are used to control how a notification is delivered. The default FCM channel used is hidden from users, however provides a "default" importance level. Heads up notifications require a "max" importance level.

This means that we need to first create a new channel with a maximum importance level & then assign incoming FCM notifications to this channel. Although this is outside of the scope of FlutterFire, we can take advantage of the [flutter_local_notifications](https://pub.dev/packages/flutter_local_notifications) package to help us:

1.  Add the `flutter_local_notifications` package to your local project.
2.  Create a new `AndroidNotificationChannel` instance:

*const* AndroidNotificationChannel channel = AndroidNotificationChannel(

  'high_importance_channel', *// id*

  'High Importance Notifications', *// title*

  'This channel is used for important notifications.', *// description*

  importance: Importance.max,

);

1.  Create the channel on the device (if a channel with an id already exists, it will be updated):

*final* FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =

    FlutterLocalNotificationsPlugin();

*await* flutterLocalNotificationsPlugin

  .resolvePlatformSpecificImplementation<AndroidFlutterLocalNotificationsPlugin>()

  ?.createNotificationChannel(channel);

Once created, we can now update FCM to use our own channel rather than the default FCM one. To do this, open the `android/app/src/main/AndroidManifest.xml` file for your FlutterProject project. Add the following `meta-data` schema within the `application` component:

<meta-data

  android:name="com.google.firebase.messaging.default_notification_channel_id"

  android:value="high_importance_channel" />

##### **Application in foreground**

If your own application is in the foreground, the Firebase Android SDK will block displaying any FCM notification no matter what Notification Channel has been set. We can however still handle an incoming notification message via the `onMessage` stream and create a custom local notification using [`flutter_local_notifications`](https://pub.dev/packages/flutter_local_notifications):

FirebaseMessaging.onMessage.listen((RemoteMessage message) {

  RemoteNotification notification = message.notification;

  AndroidNotification android = message.notification?.android;

  *// If `onMessage` is triggered with a notification, construct our own*

  *// local notification to show to users using the created channel.*

  *if* (notification != *null* && android != *null*) {

    flutterLocalNotificationsPlugin.*show*(

        notification.hashCode,

        notification.title,

        notification.body,

        NotificationDetails(

          android: AndroidNotificationDetails(

            channel.id,

            channel.name,

            channel.description,

            icon: android?.smallIcon,

            *// other properties...*

          ),

        ));

  }

});

Server Integration
==================

The Cloud Messaging module provides the tools required to enable you to send custom messages directly from your own servers. For example, you could send an FCM message to a specific device when a new chat message is saved to your database and display a notification, or update local device storage, so the message is instantly available.

Firebase provides a number of SDKs in different languages such as [Node.JS](https://www.npmjs.com/package/firebase-admin), [Java](https://firebase.google.com/docs/reference/admin/java/reference/com/google/firebase/messaging/package-summary), [Python](https://firebase.google.com/docs/reference/admin/python/firebase_admin.messaging), [C#](https://firebase.google.com/docs/reference/admin/dotnet/namespace/firebase-admin/messaging) and [Go](https://godoc.org/firebase.google.com/go/messaging). It also supports sending messages over [HTTP](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages). These methods allow you to send messages directly to your user's devices via the FCM servers.

Device tokens
-------------

To send a message to a device, you must access its unique token. A token is automatically generated by the device and can be accessed using the Cloud Messaging module. The token should be saved inside your systems data-store and should be easily accessible when required.

The examples below use a Cloud Firestore database to store and manage the tokens, and Firebase Authentication to manage the users identity. You can however use any datastore or authentication method of your choice.

If using iOS, ensure you have completed the [setup](https://firebase.flutter.dev/docs/messaging/apple-integration) & [requested user permission](https://firebase.flutter.dev/docs/messaging/permissions) before trying to receive messages!

### Saving tokens

Once your application has started, you can call the `getToken` method on the Cloud Messaging module to get the unique device token (if using a different push notification provider, such as Amazon SNS, you will need to call `getAPNSToken` on iOS):

Future<*void*> saveTokenToDatabase(String token) *async* {

  *// Assume user is logged in for this example*

  String userId = FirebaseAuth.instance.currentUser.uid;

  *await* FirebaseFirestore.instance

    .collection('users')

    .doc(userId)

    .update({

      'tokens': FieldValue.arrayUnion([token]),

    });

}

*class* Application *extends* StatefulWidget {

  @override

  State<StatefulWidget> createState() => _Application();

}

*class* _Application *extends* State<Application> {

  String _token;

  Future<*void*> setupToken() *async* {

    *// Get the token each time the application loads*

    String? token = *await* FirebaseMessaging.instance.getToken();

    *// Save the initial token to the database*

    *await* saveTokenToDatabase(token!);

    *// Any time the token refreshes, store this in the database too.*

    FirebaseMessaging.instance.onTokenRefresh.listen(saveTokenToDatabase);

  }

  @override

  *void* initState() {

    *super*.initState();

    setupToken();

  }

  @override

  Widget build(BuildContext context) {

    *return* Text("...");

  }

}

The above code snippet has a single purpose; storing the device FCM token on a remote database. When your application first initializes, the users FCM token is fetched and stored in a database (Cloud Firestore in this example). If the token is refreshed at any point whilst your application is open, the new token is also stored on the database.

It is important to remember a user can have many tokens (from multiple devices, or token refreshes), therefore we use `FieldValue.arrayUnion` to store new tokens. When a message is sent via an admin SDK, invalid/expired tokens will throw an error allowing you to then remove them from the database.

### Using tokens[#](https://firebase.flutter.dev/docs/messaging/overview/#using-tokens "Direct link to heading")

With the tokens stored in a secure datastore, we now have the ability to send messages via FCM to those devices.

The following example uses the Node.JS `firebase-admin` package to send messages to our devices, however any Firebase Admin SDK can be used.

Imagine our application being similar to Instagram. Users are able to upload pictures, and other users can "like" those pictures. Each time a post is liked, we want to send a message to the user that uploaded the picture.

The code below simulates a function which is called with all the information required when a picture is liked:

*// Node.js e.g via a Firebase Cloud Function*

*var* admin = require("firebase-admin");

*// ownerId - who owns the picture someone liked*

*// userId - id of the user who liked the picture*

*// picture - metadata about the picture*

*async* *function* onUserPictureLiked(ownerId, userId, picture) {

  *// Get the owners details*

  *const* owner = admin.firestore().collection("users").doc(ownerId).get();

  *// Get the users details*

  *const* user = admin.firestore().collection("users").doc(userId).get();

  *await* admin.messaging().sendToDevice(

    owner.tokens, *// ['token_1', 'token_2', ...]*

    {

      data: {

        owner: JSON.stringify(owner),

        user: JSON.stringify(user),

        picture: JSON.stringify(picture),

      },

    },

    {

      *// Required for background/quit data-only messages on iOS*

      contentAvailable: true,

      *// Required for background/quit data-only messages on Android*

      priority: "high",

    }

  );

}

Data-only messages are sent as low priority on both Android and iOS and will not trigger the background handler by default. To enable this functionality, you must set the "priority" to `high` on Android and enable the `content-available` flag for iOS in the message payload.

The data property can send an object of key-value pairs totaling 4 KB as string values (hence the `JSON.stringify`calls).

Within the application, you can then handle these messages how you see fit:

FirebaseMessaging.onMessage.listen((RemoteMessage message) {

  Map<String, String> data = message.data;

  Owner owner = Owner.fromMap(jsonDecode(data['owner']));

  User user = User.fromMap(jsonDecode(data['user']));

  Picture picture = Picture.fromMap(jsonDecode(data['picture']));

  print('The user ${user.name} liked your picture "${picture.title}"!');

});

Your application code can then handle messages as you see fit; updating local cache, displaying a notification or updating UI. The possibilities are endless!

Send messages to topics
-----------------------

When devices [subscribe to topics](https://firebase.flutter.dev/docs/messaging/usage#subscribing-to-topics), you can send messages without specifying/storing any device tokens.

Using the `firebase-admin` Admin SDK as an example, we can send a message to devices subscribed to a topic:

*// Node.js e.g via a Firebase Cloud Function*

*const* admin = require("firebase-admin");

*const* message = {

  data: {

    type: "warning",

    content: "A new weather warning has been created!",

  },

  topic: "weather",

};

admin

  .messaging()

  .send(message)

  .then((response) => {

    console.log("Successfully sent message:", response);

  })

  .*catch*((error) => {

    console.log("Error sending message:", error);

  });

### Conditional topics[#](https://firebase.flutter.dev/docs/messaging/overview/#conditional-topics "Direct link to heading")

To send a message to a combination of topics, specify a condition, which is a boolean expression that specifies the target topics. For example, the following condition will send messages to devices that are subscribed to `weather`and either `news` or `traffic`:

*const* admin = require("firebase-admin");

*const* message = {

  data: {

    content: "New updates are available!",

  },

  condition: "'weather' in topics && ('news' in topics || 'traffic' in topics)",

};

admin

  .messaging()

  .send(message)

  .then((response) => {

    console.log("Successfully sent message:", response);

  })

  .*catch*((error) => {

    console.log("Error sending message:", error);

  });
