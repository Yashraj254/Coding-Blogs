---
title: "Broadcasts and Broadcast Receivers in Android: A Comprehensive Guide"
seoTitle: "Understanding Broadcast and Broadcast Receivers in Android Development"
seoDescription: "Learn about broadcast and broadcast receivers in Android development, and how they facilitate communication between app components."
datePublished: Sat May 18 2024 22:21:03 GMT+0000 (Coordinated Universal Time)
cuid: clwcoa8wb00030aku80uj7pdz
slug: broadcasts-and-broadcast-receivers-in-android-a-comprehensive-guide
canonical: https://blog.yashraj.dev/broadcasts-and-broadcast-receivers-in-android-a-comprehensive-guide
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/GQwy-axAHBE/upload/bb52ecf02cb0cac6cf3685e9e5957db7.jpeg
tags: android-app-development, android-development, android, broadcast-receiver

---

# Introduction

Broadcasts in Android are system-wide messages sent by the system or other apps to signal events like battery changes or incoming messages. Broadcast receivers are components that listen for these broadcasts and respond accordingly, enabling apps to react to various events dynamically.

# What are Broadcasts?

Broadcasts are messaging components used to communicate between different applications and the Android system when an event of interest occurs. For example, the Android system sends a broadcast event when the system boots, when the phone is connected to a charger, or when headphones are plugged in or unplugged. Broadcasts can be sent either globally or locally, and any app that has registered to receive them can act on these broadcasts. Additionally, Android applications can send broadcasts to notify other apps or components about events, such as when new data is downloaded.

## Types of Broadcasts:

* System or Implicit broadcasts that are delivered by the system.
    
* Custom or Explicit broadcasts that are delivered by your app.
    

### System(Implicit) Broadcasts

System broadcasts in Android are messages sent by the Android system to inform applications about various events. They are wrapped in an Intent object whose action string identifies the event that occurred. Applications can register to receive these broadcasts either statically in the manifest file or dynamically at runtime.

For example, android.intent.action.HEADSET\_PLUG, which is sent when a wired headset is connected or disconnected.

### Custom(Explicit) Broadcasts

Custom broadcasts are the events that are sent by your app. Use a custom broadcast when you want your app to trigger certain behaviours or notify other apps about specific events without needing to launch an activity.

For example, if your app downloads new data from the internet, you can send a custom broadcast to inform other apps that the data is available.

> When a system broadcast is sent, any broadcast receiver registered to listen for that specific event will receive it. This registration can be done either statically in the manifest file or dynamically in the code.

## **Different Ways to Send a Broadcast:**

### Normal Broadcast

The `sendBroadcast()` method sends broadcasts to all the registered receivers at the same time, in an undefined order.

```kotlin
val intent = Intent()
intent.setAction("com.example.blogs.CUSTOM_BROADCAST")
// Set the optional additional information in extra field.
intent.putExtra("message", "This is a custom broadcast")
sendBroadcast(intent)
```

### Ordered Broadcast

The `sendOrderedBroadcast()` is used to send broadcasts to one receiver at a time. Let's explore it with the help of an example:

Define your Broadcast Receiver classes:

```kotlin
class Receiver1 : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Modify the received data
        var message = intent.getStringExtra("message")
        message = "$message -> Sent by Receiver1"
        
        val resultExtras = getResultExtras(true)
        resultExtras.putString("data", message)

        Log.d("Receiver1", "Received broadcast and modified message")
    }
}

class Receiver2 : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {

        // Read the modified data 
        val resultExtras = getResultExtras(true)
        val data = resultExtras.getString("data")

        Log.d("Receiver2", "Received broadcast with message: $data")
    }
}
```

Register the Broadcast receivers(will be explained later) in the Manifest file of the receiver app.

```kotlin
<application>
...
    
    <receiver android:name="com.example.blogs.Receiver1" android:priority="2">
        <intent-filter>
            <action android:name="com.example.blogs.ORDERED_BROADCAST" />
        </intent-filter>
    </receiver>

    <receiver android:name="com.example.blogs.Receiver2" android:priority="1">
        <intent-filter>
            <action android:name="com.example.blogs.ORDERED_BROADCAST" />
        </intent-filter>
    </receiver>

</application>
```

* The `android:priority` attribute that's specified in the intent filter determines the order in which the broadcast is sent.
    
* **Receiver1**: This receiver has a higher priority (2). It receives the broadcast first, modifies the message, and logs the action.
    
* **Receiver2**: This receiver has a lower priority (1). It receives the broadcast after `Receiver1` and logs the modified message.
    
* If more than one receiver with the same priority is present, the sending order is random.
    

```kotlin
val intent = Intent("com.example.blogs.ORDERED_BROADCAST")
// set the package name of receiver app
intent.setPackage("com.example.blogs")
// Set a unique action string prefixed by your app package name.
intent.putExtra("message", "Broadcast Message.");
// Deliver the Intent.
sendOrderedBroadcast(intent, null)
```

In the Logcat:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716159024918/9656993c-143f-4671-bb85-4461f987091a.png align="center")

### Local Broadcast

The `LocalBroadcastManager.sendBroadcast()` method is used to send broadcasts to receivers within your app and not outside the scope of your app.

How to send a local broadcast?

* Get an instance of LocalBroadcastManager, by calling `getInstance()` and pass in the application context.
    
* Call sendBroadcast() on the instance. Pass in the intent that you want to broadcast.
    

```kotlin
 val intent = Intent("local-broadcast")
 intent.putExtra("message", "Local Broadcast Received!")
 LocalBroadcastManager.getInstance(this).sendBroadcast(intent );
```

Although LocalBroadcastManager is deprecated, it is still usable. However, it is recommended to use LiveData, RxJava, or Flows (if working in Kotlin) as alternatives.

# What are Broadcast Receivers?

Broadcast receivers are app components that listen for broadcast events from your app, other apps, or the system itself. When an event occurs, the registered broadcast receivers are notified through an Intent.

Use broadcast receivers to respond to messages that broadcast from apps or the Android system.

To create a custom broadcast receiver extend the BroadcastReceiver class and override the `onReceive()` method:

```kotlin
//Subclass of the BroadcastReceiver class.
class CustomReceiver : BroadcastReceiver() {
    // Override the onReceive method to receive the broadcasts
    override fun onReceive(context: Context, intent: Intent) {
       //Check the Intent action and perform the required operation
       if (intent.action == "com.example.blogs.CUSTOM_BROADCAST") {
      Toast.makeText(context, "Broadcast Received!", Toast.LENGTH_SHORT).show()
     }
   }
}
```

We can register this broadcast either statically in the manifest file or dynamically in your Java or Kotlin code.

In this example, the CustomReceiver class is a subclass of BroadcastReceiver. If the incoming broadcast intent has the **com.example.blogs.CUSTOM\_BROADCAST** action, the CustomReceiver class shows a toast message.

If you need to perform a long-running operation inside BroadcastReceiver, use WorkManager to schedule a job. When you schedule a task with WorkManager, the task is guaranteed to run. WorkManager chooses the appropriate way to run your task, based on such factors as the device API level and the app state.

---

***Previously, we discussed how to send a LocalBroadcast. Now, let's explore how to register a LocalBroadcast receiver.***

Broadcast Receiver registered with LocalBroadcastManager can only receive broadcasts sent with LocalBroacastManager. If the broadcast is sent with an Activity or Service `sendBroacast()` method, it won't be received by the LocalBroadcast Receiver.

To register a receiver for local broadcasts:

1. Get an instance of **LocalBroadcastManager** by calling the `getInstance()` method.
    
2. Call `registerReceiver()`, passing in the receiver and an IntentFilter object.
    

```kotlin
class BlogActivity : AppCompatActivity() {

   // Declare an instance of the custom broadcast receiver
   private val localReceiver = MyLocalBroadcastReceiver()
   private lateinit var binding: ActivityBlogBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityBlogBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Register the local receiver with the LocalBroadcastManager to 
        // listen for "local-broadcast" intents
        LocalBroadcastManager.getInstance(this).registerReceiver(
                localReceiver,
                IntentFilter("local-broadcast")
            )

        binding.button.setOnClickListener {
           // Create a new intent with the action "local-broadcast"
            val intent = Intent("local-broadcast")
            intent.putExtra("message", "Local Broadcast Received!")
            // Send the broadcast using the LocalBroadcastManager
            LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // Unregister the local receiver when the activity is destroyed
        // to prevent memory leaks
        unregisterReceiver(localReceiver)
    }
}
```

Define the receiver class:

```kotlin
class MyLocalBroadcastReceiver : BroadcastReceiver() {
    // Override the onReceive method to receive the broadcasts
    override fun onReceive(context: Context, intent: Intent) {
        //Check the Intent action and perform the required operation
        if (intent.action == "local-broadcast") {
            Toast.makeText(context, intent.getStringExtra("message"),
                 Toast.LENGTH_LONG).show()
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716114314484/90689272-f99c-4ff3-9c24-8b8533775476.gif align="center")

## Types of broadcast receivers:

* Static receivers, which you register in the Android manifest file.
    
* Dynamic receivers, which you register using a context.
    

### Static Broadcast receivers

Static receivers are also called manifest-declared receivers. They can respond to system-wide events even when the app is not running. To register a static receiver, include the following attributes inside the element in your AndroidManifest.xml file:

The following code snippet shows static registration of a broadcast receiver that listens for a system broadcast Intent with the action "ACTION\_POWER\_CONNECTED":

```xml
<receiver
     android:name=".NotificationReceiver"
     android:exported="false">
     <intent-filter>
         <action android:name="android.intent.action.ACTION_POWER_CONNECTED" />
     </intent-filter>
</receiver>
```

* The receiver's name is the name of the BroadcastReceiver subclass (.NotificationReceiver).
    
* The receiver is not exported, meaning that no other apps can deliver broadcasts to this app.
    
* The intent filter checks whether incoming intents include an action named ACTION\_POWER\_CONNECTED, which is a system Intent action indicating that the device is connected to power.
    

In our receiver class, we'll send a notification when our device connects to a power source.

```kotlin
class NotificationReceiver: BroadcastReceiver() {
    // Override the onReceive method to receive the broadcasts
    @RequiresApi(Build.VERSION_CODES.O)
    override fun onReceive(context: Context, intent: Intent) {
        //Check the Intent action and perform the required operation
        if (intent.action == Intent.ACTION_POWER_CONNECTED) {
            val notificationManager =
                context.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

            // Create a notification channel
            val channel = NotificationChannel(
                "notification-channel-id",
                "notification-channel-name",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            notificationManager.createNotificationChannel(channel)

            // Build the notification
            val notification =
                NotificationCompat.Builder(context, "notification-channel-id")
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setContentText("Dynamic Broadcast Received!")
                    .setContentTitle("Notification Broadcast")
                    .build()

            notificationManager.notify(1, notification)
        }
    }
}
```

Note: Make sure to add the push notifications permission in the manifest file to send notifications.

```xml
 <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716112084000/2767c9c9-6c67-4074-aa08-d60a5b49b7b4.gif align="center")

### Dynamic Broadcast Receivers

Dynamic receivers enable an application to register for system or app-level events at runtime instead of in the manifest file. They can only respond to events when the app is running. They are also called context-registered receivers and are registered using either an application context or an Activity context. These receivers continue to receive broadcasts as long as the context used for registration remains valid.

1. Create an IntentFilter and add the Intent actions that you want your app to listen for. You can add more than one action to the same IntentFilter object.
    

```kotlin
val intentFilter = IntentFilter()
intentFilter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
```

2. Register the receiver by calling the `registerReceiver()` method on the context. Pass in the BroadcastReceiver object and the IntentFilter object.
    

```kotlin
val airplaneModeReceiver = AirplaneModeReceiver()

// Register it in the onCreate() method
this.registerReceiver(airplaneModeReceiver, intentFilter)
```

Define the receiver class:

```kotlin
class AirplaneModeReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        if (intent?.action == Intent.ACTION_AIRPLANE_MODE_CHANGED) {
            val isAirplaneModeEnabled = Settings.Global.getInt(
                context?.contentResolver,
                Settings.Global.AIRPLANE_MODE_ON
            ) != 0
            if(isAirplaneModeEnabled) {
                Toast.makeText(context, "Airplane mode enabled", Toast.LENGTH_LONG).show()
            } else {
                Toast.makeText(context, "Airplane mode disabled", Toast.LENGTH_LONG).show()
            }
        }
    }
}
```

Note: When working with dynamic broadcast receivers, make sure to unregister the receivers to prevent them from registering multiple times.

```kotlin
// In onDestroy() method
unregisterReceiver(airplaneModeReceiver)
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716114361830/26ed24cd-45ed-4be4-ba05-83f023b0eb62.gif align="center")

The placement of unregisterReceiver() calls depends on the intended lifecycle of your BroadcastReceiver object:

If the receiver is only required when your activity is visible, such as for disabling a network function when the network isn't available, then register the receiver in onResume() and unregister it in onPause(). Alternatively, you can opt for the onStart()/onStop() or onCreate()/onDestroy() method pairs if they better suit your specific use case.

# Conclusion

Broadcasts in Android are like messages that apps can send and receive to know about important events. For example, when your phone's battery is low, or when you start playing music on an app. These messages are received by components called broadcast receivers, which help applications react to these events. By using broadcast receivers, apps can stay updated and respond to changes in the system or user actions, making them more interactive and useful.