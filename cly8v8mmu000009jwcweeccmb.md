---
title: "Understanding Intents and Intent Filters in Android"
seoTitle: "Understanding Intents and Intent Filters in Android"
seoDescription: "Learn how to use intents and intent filters in Android to launch activities and handle implicit and explicit intents for dynamic, interactive apps."
datePublished: Fri Jul 05 2024 15:44:05 GMT+0000 (Coordinated Universal Time)
cuid: cly8v8mmu000009jwcweeccmb
slug: understanding-intents-and-intent-filters-in-android
canonical: https://blog.yashraj.dev/understanding-intents-and-intent-filters-in-android
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720193503499/27b4fe7e-1d9f-458f-89fd-530ebc986b8e.png
tags: android-app-development, android, kotlin, intent, intents

---

# What are Intents?

An **Intent** is a message object that represents a request for an action to be performed. It can be used to start activities, services, or broadcast receivers. In simple terms, Intents act as a bridge to pass information and instructions between different components of your application or between different applications.

### How Intents Work

When an Intent is created and sent, the Android system uses it to figure out which component should respond to the request. Depending on the type of Intent and how it's used, the system may:

* **Start an Activity**: Move the user to a different screen.
    
* **Start a Service**: Perform background tasks without user interaction.
    
* **Deliver a Broadcast**: Send messages to multiple components interested in system-wide or app-wide events.
    

# Types of Intents

1. Explicit Intents
    
2. Implicit Intents
    

## Explicit Intent

An Explicit Intent specifies the exact component to start by name. This is commonly used within an application when you know which activity or service you want to start.

Example 1: Here’s how to launch another activity within the same app using explicit intent.

```kotlin
class IntentSenderActivity: AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_intent_sender)

        val btnLaunch = findViewById<Button>(R.id.btn_launch)

        btnLaunch.setOnClickListener {
            // Create an explicit Intent to start IntentReceiverActivity
           val intent = Intent(this, IntentReceiverActivity::class.java)
            // Start the activity specified by the Intent
           startActivity(intent)
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720023401938/f11e0bbf-59a8-4162-b858-42b66fad4031.gif align="center")

Example 2: Launching a Calculator App Using Explicit Intent

Here we will launch the calculator app installed on your phone. To do so, you need the name of the package and the name of the class that implements the component.

Steps to Retrieve Package Name and Main Activity:

1. Open a terminal and connect to your Android device:
    

```kotlin
adb shell
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720173162506/145bb883-204e-4477-8b40-35587ae021bf.png align="center")

2. List all installed packages:
    

```kotlin
pm list packages
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720128345996/c725e586-9182-4d47-adc4-ca9ae68231c6.png align="center")

3. Filter the list to find the calculator package:
    

```kotlin
pm list packages | grep calculator
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720128294441/1d6c7119-636a-4e09-a592-e0b4f1370ea2.png align="center")

Here you will get the package name of the calculator app, i.e., `com.oneplus.calculator`.

4. Get the main activity of the calculator app:
    

```kotlin
//dumpsys package <package_name> | grep -A 1 "MAIN"
dumpsys package com.oneplus.calculator | grep -A 1 "MAIN"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720128052188/ad05e263-2f32-49fa-a62c-ef5b912fd7ae.png align="center")

Here we get the package name `com.oneplus.calculator` and the name of the main component which is the launcher activity `com.android.calculator2.activity.MiniActivity` .

```kotlin
class IntentSenderActivity: AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_intent_sender)

        val btnLaunch = findViewById<Button>(R.id.btn_launch)
        btnLaunch.setOnClickListener {
            // Create an implicit Intent with the ACTION_MAIN action
            val intent = Intent(Intent.ACTION_MAIN)
            // Set the component to target the calculator app
            intent.setComponent(ComponentName("com.oneplus.calculator", "com.android.calculator2.Calculator"))
            try {
                // Attempt to start the calculator activity
                startActivity(intent)
            } catch (e: Exception) {
                // Show a Toast message if the calculator app is not found
                Toast.makeText(this, "Calculator app not found", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720023371774/58a47b68-8a35-49eb-9326-66f1d8ed2048.gif align="center")

## Implicit Intent

Used when you declare an action to be performed. The system determines the appropriate component(s) to handle the intent based on the intent filter declarations in the manifest of other apps.

Example: Here’s how to launch the dialer app using an implicit intent.

```kotlin
class IntentReceiverActivity: AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_intent_receiver)

        val btnLaunch = findViewById<Button>(R.id.btn_launch)

        btnLaunch.setOnClickListener {
            // Create an implicit Intent with the ACTION_DIAL action
            val intent = Intent(Intent.ACTION_DIAL)
            // Set the data to a phone number URI
            intent.data = Uri.parse("tel:1234567890")
            // Create a chooser dialog for the Intent
            val chooser = Intent.createChooser(intent, "Choose Dialer")
            // Start the activity specified by the chooser
            startActivity(chooser)
        }
    }
}
```

In this example, a chooser is created to display a chooser dialog, allowing you to select the desired dialer app. If you do not create a chooser as shown in the code below,

```kotlin
val intent = Intent(Intent.ACTION_DIAL)
intent.data = Uri.parse("tel:1234567890")
startActivity(intent)
```

and directly launch the intent, the system will automatically use the default app set for that functionality (in this case, the dialer app). As a result, the chooser dialog will not appear, and the default app will be launched instead.

However, if no default app is set and if multiple matches are found, the system presents a chooser dialog to the user, allowing them to select the preferred app.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720023301743/846d5121-5101-4952-820e-27c192a31b38.gif align="center")

# Intent Filters

An **Intent Filter** is an expression in an app's manifest file that specifies the types of Intents the component can respond to. It allows the system to determine which components can handle a given Intent.

### Key Elements of Intent Filters

1. `action`: Specifies the general action to be performed, such as `ACTION_VIEW`, `ACTION_EDIT`, or `ACTION_SEND`.
    
2. `category`: Provides additional information about the action to be performed, like `CATEGORY_DEFAULT` or `CATEGORY_BROWSABLE`.
    
3. `data`: Specifies the type of data the intent is expecting, using schemes, host names, MIME types, etc.
    

When an app sends an Implicit Intent, the Android system matches it against the Intent Filters of all installed apps. If a match is found, the system launches the corresponding component. If multiple matches are found, the system presents a chooser dialog to the user, allowing them to select the preferred app.

Example: Here's how you can receive data from another app using intent filter.

1. Create an activity where we will be receiving an image which will be shared from a browser app.
    

```kotlin
class IntentReceiverActivity : AppCompatActivity() {

    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_intent_receiver)

        imageView = findViewById(R.id.imageView)

        // Handle the initial Intent that started the activity
        handleIntent(intent)
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)

        // Handle any new Intents received while the activity is already running
        handleIntent(intent)
    }

    // Function to handle the Intent and extract the URI of the image
    private fun handleIntent(intent: Intent) {

        // Get the URI from the Intent extras
        val uri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            intent.getParcelableExtra(Intent.EXTRA_STREAM, Uri::class.java)
        } else {
            intent.getParcelableExtra(Intent.EXTRA_STREAM)
        }

        // If the URI is not null, set it as the image source for the ImageView
        uri?.let {
            imageView.setImageURI(it)
        }
    }
}
```

When sharing an image from the browser app, if the activity is not already running, the `onCreate()` method is called. Inside `onCreate()`, the `handleIntent()` method processes the data (image URI). If the activity is already running, `onCreate()` is not called; instead, `onNewIntent()` is triggered, where we again call the `handleIntent()` method to handle the new data.

2. Inside the manifest file, declare the `IntentReceiverActivity` and the intent filters.
    

```kotlin
<manifest>
    <application ...>
        ...
        <activity
            android:name="._05intent.IntentReceiverActivity"
            android:exported="true"
            android:label="Intents Receiver"
            android:launchMode="singleTop" >
            <intent-filter>

                <!-- Filter for SEND action -->
                <action android:name="android.intent.action.SEND" />

                <!-- Category default is required for most implicit intents -->
                <category android:name="android.intent.category.DEFAULT" />

                <!-- Specify the MIME type for the data the activity can handle -->
                <data android:mimeType="image/*" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

1. `<activity>`:
    
    * Declares an activity that is part of the application.
        
    * `android:name`: Specifies the class name of the activity.
        
    * `android:exported`: Indicates whether the activity can be launched by components of other applications.
        
    * `android:label`: Specifies the user-readable name for the activity, displayed in the UI.
        
    * `android:launchMode`: Determines how the activity is launched, with `"singleTop"` preventing multiple instances if one already exists at the top of the stack.v
        
2. `<intent-filter>`:
    
    * Describes the types of intents the activity can respond to.
        
    * `<action android:name="android.intent.action.SEND" />`:
        
        * Specifies that the activity can handle the `SEND` action, which is used when an app sends data to another app.
            
    * `<category android:name="android.intent.category.DEFAULT" />`:
        
        * Indicates that the activity can handle the default category of intents. This is required for most implicit intents.
            
    * `<data android:mimeType="image/*" />`:
        
        * Specifies the type of data the activity can handle. In this case, the activity is set up to handle image files.
            

This configuration ensures that the `IntentReceiverActivity` can be launched when an image is shared using an `Intent` with the `SEND` action and the MIME type of `image/*`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720023279251/b655a755-95c8-4682-a4ab-37db4948b13b.gif align="center")

# Conclusion

Understanding intents and intent filters is essential for developing Android apps that communicate effectively with each other and provide a seamless user experience. Intents allow you to start activities, services, and broadcast receivers, enabling different app components to work together. By using explicit intents, you can target specific components, while implicit intents allow the system to choose the best component to handle the action. Intent filters in the manifest help declare what types of intents an activity can handle. With this knowledge, you can create more dynamic and interactive Android applications.