**Firebase Messaging with APN**

To get started with FCM, build out the simplest use case: sending a test notification message from the[Notifications composer](https://console.firebase.google.com/project/_/notification) to a development device when the app is in the background on the device. This page lists all the steps to achieve this, from setup to verification --- it may cover steps you already completed if you have [set up an Apple client app](https://firebase.google.com/docs/cloud-messaging/ios/client) for FCM.

**Add Firebase to your Apple project**

This section covers tasks you may have completed if you have already enabled other Firebase features for your app. For FCM specifically, you'll need to [upload your APNs authentication key](https://firebase.google.com/docs/cloud-messaging#upload_your_apns_authentication_key) and [register for remote notifications](https://firebase.google.com/docs/cloud-messaging#register_for_remote_notifications).

**Prerequisites**

-   Install the following:

-   Xcode 13.3.1 or later

-   Make sure that your project meets these requirements:

-   Your project must target these platform versions or later:

-   iOS 11
-   macOS 10.13
-   tvOS 12
-   watchOS 6

-   Set up a *physical Apple device* to run your app, and complete these tasks:

-   Obtain an Apple Push Notification Authentication Key for your [Apple Developer account](https://developer.apple.com/account).
-   Enable Push Notifications in XCode under **App > Capabilities**.

-   [Sign into Firebase](https://console.firebase.google.com/) using your Google account.

If you don't already have an Xcode project and just want to try out a Firebase product, you can download one of our [quickstart samples](https://firebase.google.com/docs/samples).

**Create a Firebase project**

Before you can add Firebase to your Apple app, you need to create a Firebase project to connect to your app. Visit[Understand Firebase Projects](https://firebase.google.com/docs/projects/learn-more) to learn more about Firebase projects.

**Create a Firebase project**

**Register your app with Firebase**

To use Firebase in your Apple app, you need to register your app with your Firebase project. Registering your app is often called "adding" your app to your project.

**Note:** Check out our [best practices](https://firebase.google.com/docs/projects/dev-workflows/general-best-practices) for adding apps to a Firebase project, including how to handle multiple variants.

1.   Go to the [Firebase console](https://console.firebase.google.com/).

2.   In the center of the project overview page, click the **iOS+** icon to launch the setup workflow.

If you've already added an app to your Firebase project, click **Add app** to display the platform options.

3.   Enter your app's bundle ID in the **bundle ID** field.

What's a bundle ID, and where do you find it?

Make sure to enter the bundle ID that your app is actually using. The bundle ID value is case-sensitive, and it cannot be changed for this Firebase Apple app after it's registered with your Firebase project.

4.   *(Optional)* Enter other app information: **App nickname** and **App Store ID**.

How are the *App nickname* and the *App Store ID* used within Firebase?

5.   Click **Register app**.

**Add a Firebase configuration file**

1.   Click **Download GoogleService-Info.plist** to obtain your Firebase Apple platforms config file (GoogleService-Info.plist).

What do you need to know about this config file?

2.   Move your config file into the root of your Xcode project. If prompted, select to add the config file to all targets.

If you have multiple bundle IDs in your project, you must associate each bundle ID with a registered app in the Firebase console so that each app can have its own GoogleService-Info.plist file.

**Add Firebase SDKs to your app**

Use Swift Package Manager to install and manage Firebase dependencies.

Visit [our installation guide](https://firebase.google.com/docs/ios/installation-methods) to learn about the different ways you can add Firebase SDKs to your Apple project, including importing frameworks directly and using CocoaPods.

1.   In Xcode, with your app project open, navigate to **File > Add Packages**.

2.   When prompted, add the Firebase Apple platforms SDK repository:

  https://github.com/firebase/firebase-ios-sdk

**Note:** New projects should use the default (latest) SDK version, but you can choose an older version if needed.

3.   Choose the Firebase Cloud Messaging library.

4.   For an optimal experience with Firebase Cloud Messaging, we recommend [enabling Google Analytics](https://support.google.com/firebase/answer/9289399#linkga) in your Firebase project and adding the Firebase SDK for Google Analytics to your app. You can select either the library without IDFA collection or with IDFA collection.

5.   When finished, Xcode will automatically begin resolving and downloading your dependencies in the background.

**Upload your APNs authentication key**

Upload your APNs authentication key to Firebase. If you don't already have an APNs authentication key, make sure to create one in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action).

1.   Inside your project in the Firebase console, select the gear icon, select **Project Settings**, and then select the**Cloud Messaging** tab.

2.   In **APNs authentication key** under **iOS app configuration**, click the **Upload** button.

3.   Browse to the location where you saved your key, select it, and click **Open**. Add the key ID for the key (available in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action)) and click **Upload**.

**Initialize Firebase in your app**

You'll need to add Firebase initialization code to your application. Import the Firebase module and configure a shared instance as shown:

1.   Import the FirebaseCore module in your UIApplicationDelegate, as well as any other [Firebase modules](https://firebase.google.com/docs/ios/setup#available-pods)your app delegate uses. For example, to use Cloud Firestore and Authentication:

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

import SwiftUI\
import FirebaseCore\
import FirebaseFirestore\
import FirebaseAuth\
// ...

2.   Configure a [FirebaseApp](https://firebase.google.com/docs/reference/swift/firebasecore/api/reference/Classes/FirebaseApp) shared instance in your app delegate'sapplication(_:didFinishLaunchingWithOptions:) method:

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

// Use Firebase library to configure APIs\
FirebaseApp.configure()

3.   If you're using SwiftUI, you must create an application delegate and attach it to your App struct via UIApplicationDelegateAdaptor or NSApplicationDelegateAdaptor. You must also disable app delegate swizzling. For more information, see the [SwiftUI instructions](https://firebase.google.com/docs/ios/learn-more#swiftui).

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)

@main\
struct YourApp: App {\
  // register app delegate for Firebase setup\
  @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate

  var body: some Scene {\
    WindowGroup {\
      NavigationView {\
        ContentView()\
      }\
    }\
  }\
}

**Register for remote notifications**

Either at startup, or at the desired point in your application flow, register your app for remote notifications. CallregisterForRemoteNotifications as shown:Note: SwiftUI apps should use the **UIApplicationDelegateAdaptor** or **NSApplicationDelegateAdaptor** property wrappers to provide a type corresponding to the appropriate app delegate protocol.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)\
UNUserNotificationCenter.current().delegate = self

let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]\
UNUserNotificationCenter.current().requestAuthorization(\
  options: authOptions,\
  completionHandler: { _, _ in }\
)

application.registerForRemoteNotifications()

You must assign the UNUserNotificationCenter's [**delegate**](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649522-delegate) property and FIRMessaging's [**delegate**](https://firebase.google.com/docs/reference/ios/firebasemessaging/api/reference/Classes/FIRMessaging#delegate) property. For example, in an iOS app, assign it in the **applicationWillFinishLaunchingWithOptions:** or**applicationDidFinishLaunchingWithOptions:** method of the app delegate.

**Access the registration token**

To send a message to a specific device, you need to know that device's registration token. Because you'll need to enter the token in a field in the [Notifications composer](https://console.firebase.google.com/project/_/notification) to complete this tutorial, make sure to copy the token or securely store it after you retrieve it.

By default, the FCM SDK generates a registration token for the client app instance on app launch. Similar to the APNs device token, this token allows you to send targeted notifications to any particular instance of your app.

In the same way that Apple platforms typically deliver an APNs device token on app start, FCM provides a registration token via FIRMessagingDelegate's messaging:didReceiveRegistrationToken: method. The FCM SDK retrieves a new or existing token during initial app launch and whenever the token is updated or invalidated. In all cases, the FCM SDK calls messaging:didReceiveRegistrationToken: with a valid token.

Apps still using deprecated Instance ID APIs for token management should update all token logic to use the FCM APIs described here.

The registration token may change when:

-   The app is restored on a new device
-   The user uninstalls/reinstall the app
-   The user clears app data.

**Set the messaging delegate**

To receive registration tokens, implement the messaging delegate protocol and set FIRMessaging's delegateproperty after calling [FIRApp configure]. For example, if your application delegate conforms to the messaging delegate protocol, you can set the delegate on application:didFinishLaunchingWithOptions: to itself.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

Messaging.messaging().delegate = self

**Fetching the current registration token**

Registration tokens are delivered via the method messaging:didReceiveRegistrationToken:. This method is called generally once per app start with registration token. When this method is called, it is the ideal time to:

-   If the registration token is new, send it to your application server.
-   Subscribe the registration token to topics. This is required only for new subscriptions or for situations where the user has re-installed the app.

You can retrieve the token directly using [token(completion:)](https://firebase.google.com/docs/reference/swift/firebasemessaging/api/reference/Classes/Messaging#tokencompletion:). A non null error is provided if the token retrieval failed in any way.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

Messaging.messaging().token { token, error in\
  if let error = error {\
    print("Error fetching FCM registration token: \(error)")\
  } else if let token = token {\
    print("FCM registration token: \(token)")\
    self.fcmRegTokenMessage.text  = "Remote FCM registration token: \(token)"\
  }\
}

You can use this method at any time to access the token instead of storing it.

**Monitor token refresh**

To be notified whenever the token is updated, supply a delegate conforming to the messaging delegate protocol. The following example registers the delegate and adds the proper delegate method:

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {\
  print("Firebase registration token: \(String(describing: fcmToken))")

  let dataDict: [String: String] = ["token": fcmToken ?? ""]\
  NotificationCenter.default.post(\
    name: Notification.Name("FCMToken"),\
    object: nil,\
    userInfo: dataDict\
  )\
  // TODO: If necessary send token to application server.\
  // Note: This callback is fired at each app startup and whenever a new token is generated.\
}

Alternatively, you can listen for an NSNotification namedkFIRMessagingRegistrationTokenRefreshNotification rather than supplying a delegate method. The token property always has the current token value.

**Send a notification message**

1.   Install and run the app on the target device. On Apple devices, you'll need to accept the request for permission to receive remote notifications.

2.   Make sure the app is in the background on the device.

3.   In the Firebase console, open the [Messaging page](https://console.firebase.google.com/project/_/messaging/).

4.   If this is your first message, select **Create your first campaign**.

a.   Select **Firebase Notification messages** and select **Create**.

5.   Otherwise, on the **Campaigns** tab, select **New campaign** and then **Notifications**.

6.   Enter the message text. All other fields are optional.

7.   Select **Send test message** from the right pane.

8.   In the field labeled **Add an FCM registration token**, enter the registration token you obtained in a previous section of this guide.

9.   Select **Test**.

After you select **Test**, the targeted client device (with the app in the background) should receive the notification.

For insight into message delivery to your app, see the [FCM reporting dashboard](https://console.firebase.google.com/project/_/notification/reporting), which records the number of messages sent and opened on Apple and Android devices, along with data for "impressions" (notifications seen by users) for Android apps.

FCM via APNs Integration
========================

This guide applies to both iOS & macOS Flutter apps, repeat each step for the platforms you require. E.g. if you support iOS & macOS then you will need to integrate twice, once per platform Xcode project.

Integrating the Cloud Messaging plugin on iOS & macOS devices requires additional setup before your devices receive messages. There are also a number of prerequisites which are required to be able to enable messaging:

-   You must have an active [Apple Developer Account](https://developer.apple.com/membercenter/index.action).
-   For iOS; you must have a physical iOS device to receive messages.

-   Firebase Cloud Messaging integrates with the [Apple Push Notification service (APNs)](https://developer.apple.com/notifications/), however APNs only works with real devices.

Configuring your app[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#configuring-your-app "Direct link to heading")
-------------------------------------------------------------------------------------------------------------------------------------

Before your application can start to receive messages, you must explicitly enable "Push Notifications" and "Background Modes" within Xcode.

Open your project workspace file via Xcode (`/{ios|macos}/Runner.xcworkspace`). Once open, follow the steps below:

1.   Select your project.

2.   Select the project target.

3.   Select the "Signing & Capabilities" tab.

![Xcode - Navigate to Signing & Capabilities](blob:https://euangoddard.github.io/fb65fd8f-72be-446b-a37b-2e7161d70c3b)

Xcode - Navigate to Signing & Capabilities

### Enable Push Notifications[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#enable-push-notifications "Direct link to heading")

Next the "Push Notifications" capability needs to be added to the project. This can be done via the "Capability" option on the "Signing & Capabilities" tab:

1.   Click on the "+ Capabilities" button.

2.   Search for "Push Notifications".

![Xcode - Enabling the Push Notification capability](blob:https://euangoddard.github.io/1d71a438-a32a-41b9-a627-805aab449a75)

Xcode - Enabling the Push Notification capability

Once selected, the capability will be shown below the other enabled capabilities. If no option appears when searching, the capability may already be enabled.

### Enable Background Modes (iOS only)[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#enable-background-modes-ios-only "Direct link to heading")

Next the "Background Modes" capability needs to be enabled, along with both the "Background fetch" and "Remote notifications" sub-modes. This can be added via the "Capability" option on the "Signing & Capabilities" tab:

1.   Click on the "+ Capabilities" button.

2.   Search for "Background Modes".

![Xcode - Enabling Background Modes capability](blob:https://euangoddard.github.io/40ec4ecd-8ee5-486c-aa92-7eb507bf0b9a)

Xcode - Enabling Background Modes capability

Once selected, the capability will be shown below the other enabled capabilities. If no option appears when searching, the capability may already be enabled.

Now ensure that both the "Background fetch" and the "Remote notifications" sub-modes are enabled:

![Xcode - Enabling Background Modes](blob:https://euangoddard.github.io/7da39e8e-42a0-4250-9d71-0d8dda4f4f59)

Xcode - Enabling Background Modes

Linking APNs with FCM[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#linking-apns-with-fcm "Direct link to heading")
---------------------------------------------------------------------------------------------------------------------------------------

Note: APNs is required for both `foreground` and `background`messaging to function correctly on iOS.

A few steps are required:

1.   [Registering a key](https://firebase.flutter.dev/docs/messaging/apple-integration/#1-registering-a-key).

2.   [Registering an App Identifier](https://firebase.flutter.dev/docs/messaging/apple-integration/#2-registering-an-app-identifier).

3.   [Generating a provisioning profile](https://firebase.flutter.dev/docs/messaging/apple-integration/#3-generating-a-provisioning-profile).

All of these steps require you to have access to your [Apple Developer](https://developer.apple.com/membercenter/index.action) account. Once on the account, navigate to the [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) tab on the account sidebar:

![Apple Developer portal - Certificates, Identifiers & Profiles](blob:https://euangoddard.github.io/f6cf75d8-842e-402a-b96c-363d4fde8cb0)

Apple Developer portal - Certificates, Identifiers & Profiles

### 1\. Registering a key[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#1-registering-a-key "Direct link to heading")

A key can be generated which gives the FCM full access over the Apple Push Notification service (APNs). On the "Keys" menu item, register a new key. The name of the key can be anything, however you must ensure the APNs service is enabled:

![Apple Developer portal - Enable Apple Push Notification (APNs)](blob:https://euangoddard.github.io/cfd8a68b-cff6-480e-96fc-78819fea3ac4)

Apple Developer portal - Enable Apple Push Notification (APNs)

Click "Continue" & then "Save". Once saved, you will be presented with a screen displaying the private "Key ID" & the ability to download the key. Copy the ID, and download the file to your local machine:

![Copy Key ID & Download File](blob:https://euangoddard.github.io/6c337b7d-d0a2-4ac6-a22a-9cca1cc594ec)

Copy Key ID & Download File

The file & Key ID can now be added to your Firebase Project. On the [Firebase Console](https://console.firebase.google.com/project/_/settings/cloudmessaging), navigate to the "Project settings" and select the "Cloud Messaging" tab. Select your iOS application under the "iOS app configuration" heading.

Upload the downloaded file and enter the Key & Team IDs;

![Firebase Console - Upload key](blob:https://euangoddard.github.io/b21110b1-f8cc-4590-9d1a-6967c4918332)

Firebase Console - Upload key

### 2\. Registering an App Identifier[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#2-registering-an-app-identifier "Direct link to heading")

For messaging to work when your app is built for production, you must create a new App Identifier which is linked to the application that you're developing.

On the Apple Developer portal:

1.   Click the "Identifiers" side menu item.

2.   Click the plus button to register a App Identifier.

3.   Select the "App IDs" option and click "Continue".

4.   Select the "App" type option and click "Continue".

![Apple Developer portal - Register a new identifier](blob:https://euangoddard.github.io/cebd77b3-893e-41c9-ac44-2871b93a03b0)

Apple Developer portal - Register a new identifier

The "Register an App ID" page allows you to link the identifier to your application via the "Bundle ID". This is a unique string which was generated when starting your new Flutter project. Your Bundle ID can be obtained within Xcode, under the "General" tab for your project target:

![Xcode - View Bundle ID](blob:https://euangoddard.github.io/81f72d01-9f79-4ab0-86ec-97b5dfe6c80b)

Xcode - View Bundle ID

On the Apple Developer portal "Register an App ID" page, follow these steps to create your identifier:

1.   Enter a description for the identifier.

2.   Enter the "Bundle ID" copied from Xcode.

3.   Scroll down and enable the "Push Notifications" capability (along with any others your app uses).

![Apple Developer portal - Register an App ID](blob:https://euangoddard.github.io/3baf025a-5ba9-4326-b25e-05afa676bfac)

Apple Developer portal - Register an App ID

Save the identifier, it'll be used when creating a provisioning profile in the next step.

### 3\. Generating a provisioning profile[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#3-generating-a-provisioning-profile "Direct link to heading")

A provisioning profile enables signed communicate between Apple and your application. Since messaging can only be used on real devices, a signed certificate ensures that the app being installed on a device is genuine and has the correct permissions enabled.

On the "Profiles" menu item, register a new Profile. Select the "iOS App Development" (or macOS) checkbox and click "Continue".

If you followed [Step 2](https://firebase.flutter.dev/docs/messaging/apple-integration/#2-registering-an-app-identifier) correctly, your App Identifier will be available in the drop down provided:

![Apple Developer portal - Select App ID](blob:https://euangoddard.github.io/c1ab4639-8309-47b6-8c5c-d470479822cd)

Apple Developer portal - Select App ID

Click "Continue". On the next screen you will be presented with the Certificates on your Apple account. Select the user certificates that you wish to assign this provisioning profile too. If you have not yet created a Certificate, you must set one up on your account.

To create a new Certificate, follow the [Apple documentation](https://help.apple.com/developer-account/#/devbfa00fef7). Once the Certificate has been downloaded, upload it to the Apple Developer console via the "Certificates" menu item.

The created provisioning profile can now be used when building your application (in both debug and release mode) onto a real device (using Xcode). Back within Xcode, select your project target and select the "Signing & Capabilities" tab. If Xcode (via Preferences) is linked to your Apple Account, Xcode can automatically sync the profile created above. Otherwise, you must manually add the profile from the Apple Developer console:

1.   Select the project (usually called 'Runner') in the left hand side project navigator.

2.   Select the project target under the 'Targets' heading, usually called 'Runner'.

3.   Select the Signing & Capabilities tab.

4.   Assign the provisioning profile in the 'Signing' section.

![Xcode - Assign provisioning profile](blob:https://euangoddard.github.io/40b001d3-54a4-4b91-91c7-bee36ed4bc87)

Xcode - Assign provisioning profile

(Advanced, Optional) Allowing Notification Images
-------------------------------------------------

If you want to know more about the specifics of this setup read the [official Firebase docs](https://firebase.google.com/docs/cloud-messaging/ios/send-image).

On Apple devices, in order for incoming FCM [Notifications](https://firebase.flutter.dev/docs/messaging/notifications) to display images from the FCM payload, you must add an additional notification service extension. This is not a required step.

### Step 1 - Add a notification service extension[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#step-1---add-a-notification-service-extension "Direct link to heading")

-   From Xcode top menu go to: File > New > Target...
-   A modal will present a list of possible targets, scroll down or use the filter to select "Notification Service Extension". Press Next.
-   Add a product name (use `ImageNotification` to follow along), set `Language` to `Objective-C` and click Finish.
-   Enable the scheme by clicking Activate.

![Xcode - Add a notification service extension](blob:https://euangoddard.github.io/875369f7-76de-41dd-87a5-ac9366ce990b)

Xcode - Add a notification service extension

### Step 2 - Add target to the Podfile[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#step-2---add-target-to-the-podfile "Direct link to heading")

Ensure that your new extension has access to Firebase/Messaging pod by adding it in the Podfile:

-   From the Navigator open the Podfile: Pods > Podfile
-   Scroll down to the bottom of the file and add:

target 'ImageNotification' *do*

  use_frameworks!

  pod 'Firebase/Messaging'

*end*

-   Install or update your pods using `pod install` from the `ios` and/or `macos` directory.

![Xcode - Add target to the Podfile](blob:https://euangoddard.github.io/bcf0be29-4a00-4c76-ab0d-7a38eaa6ddef)

Xcode - Add target to the Podfile

### Step 3 - Use the extension helper[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#step-3---use-the-extension-helper "Direct link to heading")

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

![Xcode - Use the extension helper](blob:https://euangoddard.github.io/2c9855ce-0016-43e6-9295-286b3355244e)

Xcode - Use the extension helper

### Step 4[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#step-4 "Direct link to heading")

Your device can now display images in a notification by specifying the `imageUrl` option in your FCM payload. Keep in mind that a 300KB max image size is enforced by the device.

Permissions

Before diving into requesting notification permissions from your users, it is important to understand how Apple and web platforms handle permissions.

Notifications cannot be shown to users if the user has not "granted" your application permission. The overall notification permission of a single application can be either be "not determined", "granted" or "declined". Upon installing a new application, the default status is "not determined".

In order to receive a "granted" status, you must request permission from your user (see below). The user can either accept or decline your request to grant permissions. If granted, notifications will be displayed based on the permission settings which were requested.

If the user declines the request, you cannot re-request permission, trying to request permission again will immediately return a "denied" status without any user interaction - instead the user must manually enable notification permissions from the device Settings UI (or via a custom UI).

**Requesting permissions**

As explained in the [Usage documentation](https://firebase.flutter.dev/docs/messaging/usage), permission must be requested from your users in order to display remote notifications from FCM, via the [requestPermission](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/requestPermission.html) API:

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

Permission settings[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#permission-settings "Direct link to heading")

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

alert

 |

true

 |

Sets whether notifications can be displayed to the user on the device.

 |
|

announcement

 |

false

 |

If enabled, Siri will read the notification content out when devices are connected to AirPods.

 |
|

badge

 |

true

 |

Sets whether a notification dot will appear next to the app icon on the device when there are unread notifications.

 |
|

carPlay

 |

true

 |

Sets whether notifications will appear when the device is connected to [CarPlay](https://www.apple.com/ios/carplay/).

 |
|

provisional

 |

false

 |

Sets whether provisional permissions are granted. See [Provisional authorization](https://firebase.flutter.dev/docs/messaging/apple-integration/#provisional-authorization) for more information.

 |
|

sound

 |

true

 |

Sets whether a sound will be played when a notification is displayed on the device.

 |

The settings provided will be stored by the device and will be visible in the iOS Settings UI for your application.

You can also read the current permissions without attempting to request them via the [getNotificationSettings](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/getNotificationSettings.html) method:

NotificationSettings settings = *await* messaging.getNotificationSettings();

Provisional authorization[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#provisional-authorization "Direct link to heading")

Devices on iOS 12+ can use provisional authorization. This type of permission system allows for notification permission to be instantly granted without displaying a dialog to your user. The permission allows notifications to be displayed quietly (only visible within the device notification center).

To enable provision permission, set the provisional argument to truewhen requesting permission:

NotificationSettings settings = *await* messaging.requestPermission(

  provisional: true,

);

When a notification is displayed on the device, the user will be presented with several actions prompting to keep receiving notifications quietly, enable full notification permission or turn them off:

![iOS Provisional Notification Example](blob:https://euangoddard.github.io/3fcc910b-24dc-4a97-b2ff-bc82e1a6d659)

iOS Provisional Notification Example

Notifications
=============

Notifications are an important tool used on the majority of applications, aimed at improve user experience & used to engage users with your application. The Cloud Messaging module provides basic support for displaying and handling notifications.

##### **IOS SIMULATORS**

FCM via APNs does not work on iOS Simulators. To receive messages & notifications a real device is required.

Displaying notifications[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#displaying-notifications "Direct link to heading")
---------------------------------------------------------------------------------------------------------------------------------------------

As mentioned in the [Usage](https://firebase.flutter.dev/docs/messaging/usage) documentation, message payloads can include a `notification` property which the Firebase SDKs intercept and attempt to display a visible notification to users.

The default behavior on all platforms is to display a notification only when the app is in the background or terminated. If required, you can override this behavior by following the [Foreground Notifications](https://firebase.flutter.dev/docs/messaging/apple-integration/#foreground-notifications) documentation.

FCM based notifications provide the basics for many use cases, such as displaying text and images. They do not support advanced notifications such as actions, styling, foreground service notifications etc. To learn more, view the [Foreground Notifications](https://firebase.flutter.dev/docs/messaging/apple-integration/#foreground-notifications) documentation.

The documentation below outlines a few different ways you can start to send notification based messages to your devices.

### Via Firebase Console[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#via-firebase-console "Direct link to heading")

The [Firebase Console](https://console.firebase.google.com/project/_/notification) provides a simple UI to allow devices to display a notification. Using the console, you can:

-   Send a basic notification with custom text and images.
-   Target applications which have been added to your project.
-   Schedule notifications to display at a later date.
-   Send recurring notifications.
-   Assign conversion events for your analytical tracking.
-   A/B test user interaction (called "experiments").
-   Test notifications on your development devices.

The Firebase Console automatically sends a message to your devices containing a notification property which is handled by the Firebase Cloud Messaging package. See [Handling Interaction](https://firebase.flutter.dev/docs/messaging/apple-integration/#handling-interaction) to learn about how to support user interaction.

### Via Admin SDKs[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#via-admin-sdks "Direct link to heading")

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

To learn more about how to integrate Cloud Messaging with your own setup, read the [Server Integration](https://firebase.flutter.dev/docs/messaging/server-integration) documentation.

### Via REST[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#via-rest "Direct link to heading")

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

Cloud Messaging
===============

To start using the Cloud Messaging package within your project, import it at the top of your project files:

*import* 'package:firebase_messaging/firebase_messaging.dart';

Before using Firebase Cloud Messaging, you must first have ensured you have [initialized FlutterFire](https://firebase.flutter.dev/docs/overview#initializing-flutterfire).

To create a new Messaging instance, call the [`instance`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/instance.html) getter on[`FirebaseMessaging`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging-class.html):

FirebaseMessaging messaging = FirebaseMessaging.instance;

Messaging currently only supports usage with the default Firebase App instance.

Receiving messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#receiving-messages "Direct link to heading")
---------------------------------------------------------------------------------------------------------------------------------

##### **IOS SIMULATORS**

FCM via APNs does not work on iOS Simulators. To receive messages & notifications a real device is required.

The Cloud Messaging package connects applications to the [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging) service. You can send message payloads directly to devices at no cost. Each message payload can be up to 4 KB in size, containing pre-defined or custom data to suit your applications requirements.

Common use-cases for using messages could be:

-   Displaying a notification (see [Notifications](https://firebase.flutter.dev/docs/messaging/notifications)).
-   Syncing message data silently on the device (e.g. via [shared_preferences](https://flutter.dev/docs/cookbook/persistence/key-value)).
-   Updating the application's UI.

To learn about how to send messages to devices from your own server setup, view the [Server Integration](https://firebase.flutter.dev/docs/messaging/server-integration) documentation.

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

### Requesting permission (Apple & Web)[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#requesting-permission-apple--web "Direct link to heading")

On iOS, macOS & web, before FCM payloads can be received on your device, you must first ask the user's permission. Android applications are not required to request permission.

The `firebase_messaging` package provides a simple API for requesting permission via the [`requestPermission`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/requestPermission.html) method. This API accepts a number of named arguments which define the type of permissions you'd like to request, such as whether messaging containing notification payloads can trigger a sound or read out messages via Siri. By default, the method requests sensible default permissions. The reference API provides full documentation on what each permission is for.

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

Handling messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#handling-messages "Direct link to heading")
-------------------------------------------------------------------------------------------------------------------------------

Once permission has been granted & the different types of device state have been understood, your application can now start to handle the incoming FCM payloads.

### Web Tokens[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#web-tokens "Direct link to heading")

On the web, before a message can be sent to the browser you must do two things.

1.   Create an initial handshake with Firebase by passing in the public `vapidKey` to `messaging.getToken(vapidKey: 'KEY')` method. Head over to the [Firebase Console](https://console.firebase.google.com/project/_/settings/cloudmessaging) and create a new "Web Push Certificate". A key will be provided, which you can provide to the method:

FirebaseMessaging messaging = FirebaseMessaging.instance;

*// use the returned token to send messages to users from your custom server*

String token = *await* messaging.getToken(

  vapidKey: "BGpdLRs......",

);

2.   Create a `firebase-messaging-sw.js` file inside the `web/` directory in the root of your project. In your `web/index.html` file, please ensure this file is referenced and registered as a `serviceWorker` as demonstrated below:

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

For a complete `firebase_messaging` web demonstration, please run our example app found [here](https://github.com/FirebaseExtended/flutterfire/tree/master/packages/firebase_messaging/firebase_messaging/example). Take care to ensure you have setup the `web/firebase-messaging-sw.js` file found [here](https://github.com/FirebaseExtended/flutterfire/blob/master/packages/firebase_messaging/firebase_messaging/example/web/firebase-messaging-sw.js).

### Message types[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#message-types "Direct link to heading")

A message payload can be viewed as one of three types:

1.   Notification only message: The payload contains a `notification`property, which will be used to [present a visible notification to the user](https://firebase.flutter.dev/docs/messaging/notifications).

2.   Data only message: Also known as a "silent message", this payload contains custom key/value pairs within the `data` property which can be used how you see fit. These messages are considered "low priority" (more on this later).

3.   Notification & Data messages: Payloads with both `notification` and `data` properties.

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

`onBackgroundMessage`(***see below***)

 |

`onBackgroundMessage`(***see below***)

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

### Foreground messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#foreground-messages "Direct link to heading")

To listen to messages whilst your application is in the foreground, listen to the [`onMessage`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/onMessage.html) stream.

FirebaseMessaging.onMessage.listen((RemoteMessage message) {

  print('Got a message whilst in the foreground!');

  print('Message data: ${message.data}');

  *if* (message.notification != *null*) {

    print('Message also contained a notification: ${message.notification}');

  }

});

The stream contains a `RemoteMessage`, detailing various information about the payload, such as where it was from, the unique ID, sent time, whether it contained a notification & more. Since the message was retrieved whilst your application is in the foreground, you can directly access your Flutter application's state & context.

#### **Foreground & Notification messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#foreground--notification-messages "Direct link to heading")**

Notification messages which arrive whilst the application is in the foreground will not display a visible notification by default, on both Android & iOS. It is, however, possible to override this behavior:

-   On Android, you must create a "High Priority" notification channel.
-   On iOS, you can update the presentation options for the application.

More details on this are discussed in the [Notification: Foreground notifications](https://firebase.flutter.dev/docs/messaging/notifications#foreground-notifications)documentation.

### Background messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#background-messages "Direct link to heading")

The process of handling background messages is currently different on Android/Apple & web based platforms. We're working to see if it's possible to align these flows.

-       Android/Apple

-       Web

Handling messages whilst your application is in the background is a little different. Messages can be handled via the [`onBackgroundMessage`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/onBackgroundMessage.html) handler. When received, an isolate is spawned (Android only, iOS/macOS does not require a separate isolate) allowing you to handle messages even when your application is not running.

There are a few things to keep in mind about your background message handler:

1.   It must not be an anonymous function.

2.   It must be a top-level function (e.g. not a class method which requires initialization).

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

#### **Notifications[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#notifications "Direct link to heading")**

If your message is a notification one (includes a `notification` property), the Firebase SDKs will intercept this and display a visible notification to your users (assuming you have requested permission & the user has notifications enabled). Once displayed, the background handler will be executed (if provided).

To learn about how to handle user interaction with a notification, view the [Notifications](https://firebase.flutter.dev/docs/messaging/notifications) documentation.

#### **Debugging & Hot Reload[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#debugging--hot-reload "Direct link to heading")**

FlutterFirebase Messaging does now support debugging and hot reloading for background isolates, but only if your main isolate is also being debugged, (e.g. run your application in debug and then background it by switching apps so it's no longer in the foreground).

For viewing additional logs on iOS, the "console.app" application on your Mac also displays system logs for your iOS device, including those from Flutter.

#### **Low priority messages[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#low-priority-messages "Direct link to heading")**

As mentioned above, data only messages are classed as "low priority". Devices can throttle and ignore these messages if your application is in the background, terminated, or a variety of other conditions such as low battery or currently high CPU usage.

You should not rely on data only messages to be delivered. They should only be used to support your application's non-critical functionality, e.g. pre-fetching data so the next time the user opens your app the data is ready to be displayed and if the message never gets delivered then your app still functions and fetches data on open.

To help improve delivery, you can bump the priority of messages. Note: this still does not guarantee delivery.

For example, if using the `firebase-admin` NodeJS SDK package to send notifications via your server, add additional properties to the message payload:

*const* admin = require("firebase-admin");

admin.initializeApp({

  credential: admin.credential.cert(require("./service-account-file.json")),

  databaseURL: "https://....firebaseio.com",

});

admin.messaging().send({

  token: "device token",

  data: {

    hello: "world",

  },

  *// Set Android priority to "high"*

  android: {

    priority: "high",

  },

  *// Add APNS (Apple) config*

  apns: {

    payload: {

      aps: {

        contentAvailable: true,

      },

    },

    headers: {

      "apns-push-type": "background",

      "apns-priority": "5", *// Must be `5` when `contentAvailable` is set to true.*

      "apns-topic": "io.flutter.plugins.firebase.messaging", *// bundle identifier*

    },

  },

});

This configuration provides the best chance that your data-only message will be delivered to the device. You can read full descriptions on the use of the properties on the [official Firebase documentation](https://firebase.google.com/docs/reference/admin/node/firebase-admin.messaging.tokenmessage).

Apple has very strict undisclosed polices on data only messages, which are very frequently ignored. For example, sending too many in a certain time period will cause the device to block messages, or if CPU consumption is high they will also be blocked.

For Android, you can view Logcat logs which will give a descriptive message on why a notification was not delivered. On Apple platforms the "console.app" application will display "CANCELED" logs for those it chose to ignore, however doesn't provide a description as to why.

Topics[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#topics "Direct link to heading")
---------------------------------------------------------------------------------------------------------

Topics are a mechanism which allow a device to subscribe and unsubscribe from named PubSub channels, all managed via FCM. Rather than sending a message to a specific device by FCM token, you can instead send a message to a topic and any devices subscribed to that topic will receive the message.

Topics allow you to simplify FCM server integration as you do not need to keep a store of device tokens. There are, however, some things to keep in mind about topics:

-   Messages sent to topics should not contain sensitive or private information. Do not create a topic for a specific user to subscribe to.
-   Topic messaging supports unlimited subscriptions for each topic.
-   One app instance can be subscribed to no more than 2000 topics.
-   The frequency of new subscriptions is rate-limited per project. If you send too many subscription requests in a short period of time, FCM servers will respond with a 429 RESOURCE_EXHAUSTED ("quota exceeded") response. Retry with an exponential backoff.
-   A server integration can send a single message to multiple topics at once. This, however, is limited to 5 topics.

To learn more about how to send messages to devices subscribed to topics, view the [Send messages to topics](https://firebase.flutter.dev/docs/messaging/server-integration#send-messages-to-topics) documentation.

### Subscribing to topics[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#subscribing-to-topics "Direct link to heading")

To subscribe a device, call the [`subscribeToTopic`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/subscribeToTopic.html) method with the topic name:

*// subscribe to topic on each app start-up*

*await* FirebaseMessaging.instance.subscribeToTopic('weather');

### Unsubscribing from topics[#](https://firebase.flutter.dev/docs/messaging/apple-integration/#unsubscribing-from-topics "Direct link to heading")

To unsubscribe from a topic, call the [`unsubscribeFromTopic`](https://pub.dev/documentation/firebase_messaging/latest/firebase_messaging/FirebaseMessaging/unsubscribeFromTopic.html) method with the topic name:

*await* FirebaseMessaging.instance.unsubscribeFromTopic('weather');

To get started with FCM, build out the simplest use case: sending a test notification message from the[Notifications composer](https://console.firebase.google.com/project/_/notification) to a development device when the app is in the background on the device. This page lists all the steps to achieve this, from setup to verification --- it may cover steps you already completed if you have [set up an Apple client app](https://firebase.google.com/docs/cloud-messaging/ios/client) for FCM.

**Add Firebase to your Apple project**

This section covers tasks you may have completed if you have already enabled other Firebase features for your app. For FCM specifically, you'll need to [upload your APNs authentication key](https://firebase.google.com/docs/cloud-messaging#upload_your_apns_authentication_key) and [register for remote notifications](https://firebase.google.com/docs/cloud-messaging#register_for_remote_notifications).

**Prerequisites**

-   Install the following:

-   Xcode 13.3.1 or later

-   Make sure that your project meets these requirements:

-   Your project must target these platform versions or later:

-   iOS 11
-   macOS 10.13
-   tvOS 12
-   watchOS 6

-   Set up a *physical Apple device* to run your app, and complete these tasks:

-   Obtain an Apple Push Notification Authentication Key for your [Apple Developer account](https://developer.apple.com/account).
-   Enable Push Notifications in XCode under **App > Capabilities**.

-   [Sign into Firebase](https://console.firebase.google.com/) using your Google account.

If you don't already have an Xcode project and just want to try out a Firebase product, you can download one of our [quickstart samples](https://firebase.google.com/docs/samples).

**Create a Firebase project**

Before you can add Firebase to your Apple app, you need to create a Firebase project to connect to your app. Visit[Understand Firebase Projects](https://firebase.google.com/docs/projects/learn-more) to learn more about Firebase projects.

**Create a Firebase project**

**Register your app with Firebase**

To use Firebase in your Apple app, you need to register your app with your Firebase project. Registering your app is often called "adding" your app to your project.

**Note:** Check out our [best practices](https://firebase.google.com/docs/projects/dev-workflows/general-best-practices) for adding apps to a Firebase project, including how to handle multiple variants.

6.   Go to the [Firebase console](https://console.firebase.google.com/).

7.   In the center of the project overview page, click the **iOS+** icon to launch the setup workflow.

If you've already added an app to your Firebase project, click **Add app** to display the platform options.

8.   Enter your app's bundle ID in the **bundle ID** field.

What's a bundle ID, and where do you find it?

Make sure to enter the bundle ID that your app is actually using. The bundle ID value is case-sensitive, and it cannot be changed for this Firebase Apple app after it's registered with your Firebase project.

9.   *(Optional)* Enter other app information: **App nickname** and **App Store ID**.

How are the *App nickname* and the *App Store ID* used within Firebase?

10.                Click **Register app**.

**Add a Firebase configuration file**

3.   Click **Download GoogleService-Info.plist** to obtain your Firebase Apple platforms config file (GoogleService-Info.plist).

What do you need to know about this config file?

4.   Move your config file into the root of your Xcode project. If prompted, select to add the config file to all targets.

If you have multiple bundle IDs in your project, you must associate each bundle ID with a registered app in the Firebase console so that each app can have its own GoogleService-Info.plist file.

**Add Firebase SDKs to your app**

Use Swift Package Manager to install and manage Firebase dependencies.

Visit [our installation guide](https://firebase.google.com/docs/ios/installation-methods) to learn about the different ways you can add Firebase SDKs to your Apple project, including importing frameworks directly and using CocoaPods.

6.   In Xcode, with your app project open, navigate to **File > Add Packages**.

7.   When prompted, add the Firebase Apple platforms SDK repository:

  https://github.com/firebase/firebase-ios-sdk

**Note:** New projects should use the default (latest) SDK version, but you can choose an older version if needed.

8.   Choose the Firebase Cloud Messaging library.

9.   For an optimal experience with Firebase Cloud Messaging, we recommend [enabling Google Analytics](https://support.google.com/firebase/answer/9289399#linkga) in your Firebase project and adding the Firebase SDK for Google Analytics to your app. You can select either the library without IDFA collection or with IDFA collection.

10.                When finished, Xcode will automatically begin resolving and downloading your dependencies in the background.

**Upload your APNs authentication key**

Upload your APNs authentication key to Firebase. If you don't already have an APNs authentication key, make sure to create one in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action).

4.   Inside your project in the Firebase console, select the gear icon, select **Project Settings**, and then select the**Cloud Messaging** tab.

5.   In **APNs authentication key** under **iOS app configuration**, click the **Upload** button.

6.   Browse to the location where you saved your key, select it, and click **Open**. Add the key ID for the key (available in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action)) and click **Upload**.

**Initialize Firebase in your app**

You'll need to add Firebase initialization code to your application. Import the Firebase module and configure a shared instance as shown:

4.   Import the FirebaseCore module in your UIApplicationDelegate, as well as any other [Firebase modules](https://firebase.google.com/docs/ios/setup#available-pods)your app delegate uses. For example, to use Cloud Firestore and Authentication:

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

import SwiftUI\
import FirebaseCore\
import FirebaseFirestore\
import FirebaseAuth\
// ...

5.   Configure a [FirebaseApp](https://firebase.google.com/docs/reference/swift/firebasecore/api/reference/Classes/FirebaseApp) shared instance in your app delegate'sapplication(_:didFinishLaunchingWithOptions:) method:

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

// Use Firebase library to configure APIs\
FirebaseApp.configure()

6.   If you're using SwiftUI, you must create an application delegate and attach it to your App struct via UIApplicationDelegateAdaptor or NSApplicationDelegateAdaptor. You must also disable app delegate swizzling. For more information, see the [SwiftUI instructions](https://firebase.google.com/docs/ios/learn-more#swiftui).

[SwiftUI](https://firebase.google.com/docs/cloud-messaging#swiftui)

@main\
struct YourApp: App {\
  // register app delegate for Firebase setup\
  @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate

  var body: some Scene {\
    WindowGroup {\
      NavigationView {\
        ContentView()\
      }\
    }\
  }\
}

**Register for remote notifications**

Either at startup, or at the desired point in your application flow, register your app for remote notifications. CallregisterForRemoteNotifications as shown:Note: SwiftUI apps should use the **UIApplicationDelegateAdaptor** or **NSApplicationDelegateAdaptor** property wrappers to provide a type corresponding to the appropriate app delegate protocol.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)\
UNUserNotificationCenter.current().delegate = self

let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]\
UNUserNotificationCenter.current().requestAuthorization(\
  options: authOptions,\
  completionHandler: { _, _ in }\
)

application.registerForRemoteNotifications()

You must assign the UNUserNotificationCenter's [**delegate**](https://developer.apple.com/documentation/usernotifications/unusernotificationcenter/1649522-delegate) property and FIRMessaging's [**delegate**](https://firebase.google.com/docs/reference/ios/firebasemessaging/api/reference/Classes/FIRMessaging#delegate) property. For example, in an iOS app, assign it in the **applicationWillFinishLaunchingWithOptions:** or**applicationDidFinishLaunchingWithOptions:** method of the app delegate.

**Access the registration token**

To send a message to a specific device, you need to know that device's registration token. Because you'll need to enter the token in a field in the [Notifications composer](https://console.firebase.google.com/project/_/notification) to complete this tutorial, make sure to copy the token or securely store it after you retrieve it.

By default, the FCM SDK generates a registration token for the client app instance on app launch. Similar to the APNs device token, this token allows you to send targeted notifications to any particular instance of your app.

In the same way that Apple platforms typically deliver an APNs device token on app start, FCM provides a registration token via FIRMessagingDelegate's messaging:didReceiveRegistrationToken: method. The FCM SDK retrieves a new or existing token during initial app launch and whenever the token is updated or invalidated. In all cases, the FCM SDK calls messaging:didReceiveRegistrationToken: with a valid token.

Apps still using deprecated Instance ID APIs for token management should update all token logic to use the FCM APIs described here.

The registration token may change when:

-   The app is restored on a new device
-   The user uninstalls/reinstall the app
-   The user clears app data.

**Set the messaging delegate**

To receive registration tokens, implement the messaging delegate protocol and set FIRMessaging's delegateproperty after calling [FIRApp configure]. For example, if your application delegate conforms to the messaging delegate protocol, you can set the delegate on application:didFinishLaunchingWithOptions: to itself.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

Messaging.messaging().delegate = self

**Fetching the current registration token**

Registration tokens are delivered via the method messaging:didReceiveRegistrationToken:. This method is called generally once per app start with registration token. When this method is called, it is the ideal time to:

-   If the registration token is new, send it to your application server.
-   Subscribe the registration token to topics. This is required only for new subscriptions or for situations where the user has re-installed the app.

You can retrieve the token directly using [token(completion:)](https://firebase.google.com/docs/reference/swift/firebasemessaging/api/reference/Classes/Messaging#tokencompletion:). A non null error is provided if the token retrieval failed in any way.

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

Messaging.messaging().token { token, error in\
  if let error = error {\
    print("Error fetching FCM registration token: \(error)")\
  } else if let token = token {\
    print("FCM registration token: \(token)")\
    self.fcmRegTokenMessage.text  = "Remote FCM registration token: \(token)"\
  }\
}

You can use this method at any time to access the token instead of storing it.

**Monitor token refresh**

To be notified whenever the token is updated, supply a delegate conforming to the messaging delegate protocol. The following example registers the delegate and adds the proper delegate method:

[Swift](https://firebase.google.com/docs/cloud-messaging#swift)[Objective-C](https://firebase.google.com/docs/cloud-messaging#objective-c)

func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {\
  print("Firebase registration token: \(String(describing: fcmToken))")

  let dataDict: [String: String] = ["token": fcmToken ?? ""]\
  NotificationCenter.default.post(\
    name: Notification.Name("FCMToken"),\
    object: nil,\
    userInfo: dataDict\
  )\
  // TODO: If necessary send token to application server.\
  // Note: This callback is fired at each app startup and whenever a new token is generated.\
}

Alternatively, you can listen for an NSNotification namedkFIRMessagingRegistrationTokenRefreshNotification rather than supplying a delegate method. The token property always has the current token value.

**Send a notification message**

10.                Install and run the app on the target device. On Apple devices, you'll need to accept the request for permission to receive remote notifications.

11.                Make sure the app is in the background on the device.

12.                In the Firebase console, open the [Messaging page](https://console.firebase.google.com/project/_/messaging/).

13.                If this is your first message, select **Create your first campaign**.

a.   Select **Firebase Notification messages** and select **Create**.

14.                Otherwise, on the **Campaigns** tab, select **New campaign** and then **Notifications**.

15.                Enter the message text. All other fields are optional.

16.                Select **Send test message** from the right pane.

17.                In the field labeled **Add an FCM registration token**, enter the registration token you obtained in a previous section of this guide.

18.                Select **Test**.

After you select **Test**, the targeted client device (with the app in the background) should receive the notification.

For insight into message delivery to your app, see the [FCM reporting dashboard](https://console.firebase.google.com/project/_/notification/reporting), which records the number of messages sent and opened on Apple and Android devices, along with data for "impressions" (notifications seen by users) for Android apps.
