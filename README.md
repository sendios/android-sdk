# Sendios SDK for Android apps
Email Marketing and Push platform for your key product metrics https://sendios.io

- [Introduction](#introduction)
- [Installation](#installation)
- [Adding Credentials](#adding-credentials)
- [Init SDK](#init-sdk)
- [Attach User](#attach-user)
- [Log events](#log-events)
- [Push Notifications](#push-notifications)
  - [Push Token](#push-token)
  - [Push delivered and Push delivered but unseen](#Push-delivered-but-unseen)
  - [Push delivered and clicked](#push-delivered-and-clicked)
  - [Unsubscribe](#unsubscribe)
  - [Push Notification Payload](#push-notification-payload)

# Introduction
Use the Sendios SDK to design and send push notifications, track and report events occurred in your application.
Developers using the Sendios SDK with their app are required to register for a credential, and to specify the credential (apiKey) in their application. Failure to do so results in blocked access to certain features and degradation in the quality of other services.
To obtain these credentials, visit the developer portal at https://sendios.io and register for a license.
Credentials are unique to your application's bundle identifier. Do not reuse credentials across multiple applications.

# Installation
Requirements

Sendios uses Volley HTTP library to deal with networking.
Add the following to your dependencies section.
```build.gradle
dependencies {
  implementation 'com.android.volley:volley:1.1.1'
}
```

Manual integration
- Download SDK for Android package from this repo: [sendios.aar](https://github.com/sendios/android-sdk/blob/master/sendios-full-release.aar).
- Add the sendios archive to your Android studio project. Open up the project structure by right-clicking on your project and choosing “Open Module Settings” or choosing “File” > “Project Structure…”. Click the “+” button to add a new module. Choose “Import .JAR or .AAR Package” and click the “Next” button. Find your file and click “Finish”. Gradle will sync, which may take a few minutes.

# Adding Credentials
Ensure that you have provided the appId, clientId, clientToken before using the Sendios SDK.

You can add your credentials as <meta-data/> attributes to AndroidManifest.xml file as follows:
1. In your development environment double-click your project's AndroidManifest.xml file and
ensure that you are viewing the file in text editor mode.
2. Within the <application></application> block of tags, add the following markup directly
beneath the <activity></activity> tag:

```build.gradle
     <meta-data android:name="io.corp.sendios.appId"
      android:value="{YOUR_APP_ID}"/>
     <meta-data android:name="io.corp.sendios.appClientId"
      android:value="{YOUR_CLIENT_ID}"/>
     <meta-data android:name="io.corp.sendios.clientToken"
      android:value="{YOUR_CLIENT_TOKEN}"/>
```
3. Replace {YOUR_APP_ID}, {YOUR_APP_CODE} and {YOUR_CLIENT_TOKEN} with the appropriate credentials for your application.

# Init SDK
Also creates session in analytics

```kotlin
import io.corp.sendios.Sendios

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Sendios.startInit(applicationContext)
    }

}
```

# Attach User
Log a user's email and unique id if any.
Additional you can provide more product information for analytics and etc.

```kotlin
Sendios.logUser("AlexSendios@gmail.com")
```

# Log events
Log an event. You can track whatever you want.
When you find yourself debating whether or not a user action or detail is important
When you doubt then just track. The data will be available to you, you will decide
at a later point in time that you do in fact want to analyze.

```kotlin
Sendios.logEvent("some_event_name", "some_event_value")
```

# Push Notifications

## Push Token
Upon receiving push-token just forward it to Sendios

```java
import io.corp.sendios.Sendios;

import static android.content.ContentValues.TAG;

public class AppFirebaseMessagingService extends FirebaseMessagingService {

    @Override
    public void onNewToken(String token) {
        Log.d(TAG, "Refreshed token: " + token);
        Sendios.INSTANCE.pushToken(token);
    }
}
```

## Push delivered and Push delivered but unseen
There are few scenarios might occur upon a push being received. For instance,
push has been delivered when app in foreground state and not being shown to user.
To distinct different scenarios there are set of functions have to be invoked.

- Push delivered
```kotlin
class AppFirebaseMessagingService : FirebaseMessagingService() {

  override fun onMessageReceived(remoteMessage: RemoteMessage) {

    val senderId = remoteMessage.data[“sender_id”] ?: return // sender_id for android platform is 2

    val notificationContent = if (senderId.toInt() == SENDER_ID_SENDIOS) {
      Sendios.logDelivedPush(remoteMessage.data)
    } else {
      null
    }

    if (notificationContent != null) {
      if (appStateManager.isAppForeground()) {
        Sendios.logUnseenPush(pushPayload)
    } else {

    }
}
```

## Push delivered and clicked
Upon push clicking the push payload has to be sent
```java
  Sendios.INSTANCE.logPush(pushPayload)
```

## Unsubscribe

The user must first subscribe through the native prompt or app settings. It does not officially subscribe or unsubscribe them from the app settings, it unsubscribes them from receiving push from Sendios.
You can only call this method with false to opt out users from receiving notifications through Sendios. You can pass true later to opt users back into notifications.

```kotlin
Sendios.setSubscription(false)
```

## Push Notification Payload
Payload Key Reference.
Table lists the keys that payload may include.

|Key                  |Value type    |  Description     |
|---------------------|:------------:| -----------------
|data                 |Dictionary    | Data messages have only custom key-value pairs with no reserved key names

Table lists the keys that you may receive in the data Dictionary.

|Key                  | Value type   |     Description  |
|---------------------|:------------:| -----------------
|push_id              |String        |  Sender id information
|type                 |String        |  Might be any. For instance 'route'
|route                |String        |  null|chat|profile|correspondence|dialogs|search|me|credits|welcome-package|activity
|from_user_id         |String        |  For route type only. chat|profile|correspondence, otherwise is null
|image_url            |String        |  The name of the launch image file to display.
|sender_id            |String        |  It is platform specific option. For iOS by default is 2.
|logic_id             |String        |  Logic identifier
|template_id          |String        |  Template's name
|mf_split_group       |String        |  Split group name. By default is nil.
|notification         |Dictionary    |  Have a predefined set of user-visible keys and an optional data payload of custom key-value pairs.

Table lists the keys that you may receive in the notification Dictionary.

|Key                  | Value type   |     Description  |
|---------------------|:------------:| -----------------
|title                | String       |   The title of the notification.
|body                 | String       |   Additional information that explains the purpose of the notification.
|image_url            | String       |   The name of the launch image file to display.
|image_url_big        | String       |   The name of the big launch image file to display.
