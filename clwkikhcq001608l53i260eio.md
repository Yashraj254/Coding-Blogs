---
title: "Exploring Android Services: Definitions, Types, and Use Cases"
seoTitle: "Understanding Android Services: Types, Definitions, and Practical Appl"
seoDescription: "Discover the essentials of Android services, including their types, definitions, and real-world applications. Enhance your understanding of mobile app devel"
datePublished: Fri May 24 2024 10:03:12 GMT+0000 (Coordinated Universal Time)
cuid: clwkikhcq001608l53i260eio
slug: exploring-android-services-definitions-types-and-use-cases
canonical: https://blog.yashraj.dev/exploring-android-services-definitions-types-and-use-cases
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/f0dJjQMhfXo/upload/683d29731ca2edf422cbb4329a03244e.jpeg
tags: android-app-development, android, kotlin, services-in-android

---

# Introduction

Services is an Android app component that is used to perform long-running task in the background that does not have a UI. It can be used for various functionalities which don't need user input such as playing music, downloading data, watching network traffic and much more. Services can also be used for building a continuous connection to a distant server, for various purposes such as sending or receiving messages from a chat server.

# Types of Services in Android

* Foreground Services
    
* Background Services
    
* Bound Services
    

## Foreground Services

Foreground services are those services that notify the users about ongoing operations by displaying a notification in the notification bar. Users can interact with the service through the notification provided by the task. These services can keep running even if the app is terminated. Example use cases:

1. A music streaming app uses a foreground service for continuous playback and player controls.
    
2. A video call app employs a foreground service for maintaining an active call session with video and audio.
    
3. An audio recording app starts a foreground service for capturing high-quality audio recordings.
    

Here is a simple example code for a Foreground Service:

**Service Class (ForegroundService.kt):**

```kotlin
class ForegroundService : Service() {
    private val TAG = "ForegroundService"
    companion object {
        var isServiceRunning = false
    }

    override fun onCreate() {
        super.onCreate()
        showToast("Service is being created")
    }

    @RequiresApi(Build.VERSION_CODES.O)
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        showToast("Service is starting")
        isServiceRunning = true
        val notificationManager =
            this.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        // Create a notification channel
        val channel = NotificationChannel(
            "notification-channel-id",
            "notification-channel-name",
            NotificationManager.IMPORTANCE_DEFAULT
        )
        notificationManager.createNotificationChannel(channel)

        // Build the notification
        val notification =
            NotificationCompat.Builder(this, "notification-channel-id")
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentText("Foreground Service Running!")
                .setContentTitle("Foreground Service")
                .build()
        
        // Start the service in the foreground with the notification   
        startForeground(1, notification)

        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        isServiceRunning = false
        showToast("Service is being destroyed")
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }

    fun showToast(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }
}
```

**Activity Class (ServiceActivity.kt):**

```kotlin
class ServiceActivity : AppCompatActivity() {

    private lateinit var binding: Activity2serviceBinding
    private lateinit var foregroundIntent: Intent

    @RequiresApi(Build.VERSION_CODES.O)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = Activity2serviceBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Create an intent to start ForegroundService
        foregroundIntent= Intent(this, ForegroundService::class.java)

        binding.startService.setOnClickListener {
             // Check if ForegroundService is already running
            if (ForegroundService.isServiceRunning) {
                Toast.makeText(this, "Service is already running", Toast.LENGTH_SHORT).show()
            } else {
                // Start the ForegroundService if not already running
                startService(foregroundIntent)
            }
        }
        binding.stopService.setOnClickListener {
            // Stop the ForegroundService
            stopService(foregroundIntent)
        }  
    }
}
```

**Manifest File (AndroidManifest.kt):**

```kotlin
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" >

<!-- Permission to post notifications. Required for foreground services to show notifications. -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

<!-- Permission to run foreground services. Required to start services in the foreground. -->
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<application>
        <!-- Other application components like activities and receivers -->
...


<!-- Declaration of the ForegroundService. This tells the system about the service so it can be started and used by the app. -->
<service android:name=".ForegroundService"/>
</application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716491243134/647d02e2-ea03-4732-97ed-0be381b99cac.gif align="center")

## Background Services

Background Services are those that perform tasks without directly notifying the users or requiring interaction from users. These services work in the background, i.e., they can continue running even when the app is not actively visible on the screen or when the device is locked. However, it's important to note that if the app is stopped or no longer running, background services associated with that app will also stop running, as they are tied to the lifecycle of the app. Example use cases:

1. A backup app initiates background services for scheduled backups of user data to cloud storage.
    
2. A news app uses background services to fetch and update news feeds at set intervals.
    
3. An app that syncs data with a server uses background services for periodic synchronization without user intervention.
    

Here is a simple example code for a Background Service:

**Service Class (BackgroundService.kt):**

```kotlin
class BackgroundService : Service() {

    override fun onCreate() {
        super.onCreate()
        showToast("Service is being created")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        showToast("Service is starting")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        showToast("Service is being destroyed")
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }

    fun showToast(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }
}
```

**Activity Class (ServiceActivity.kt):**

```kotlin
class ServiceActivity : AppCompatActivity() {

    private lateinit var binding: Activity2serviceBinding
    private lateinit var backgroundIntent: Intent

    @RequiresApi(Build.VERSION_CODES.O)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = Activity2serviceBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Create an intent to start BackgroundService
        backgroundIntent = Intent(this, BackgroundService::class.java)

        binding.startService.setOnClickListener {     
            // Start the BackgroundService if not already running
            startService(backgroundIntent)          
        }
        binding.stopService.setOnClickListener {
            // Stop the BackgroundService 
            stopService(backgroundIntent)
        }  
    }

    override fun onDestroy() {
        // Stop the BackgroundService (prevent memory leaks)
        stopService(backgroundIntent)
        super.onDestroy()
    }
}
```

**Manifest File (AndroidManifest.kt):**

```xml
<manifest>
<application>
...
<service android:name=".BackgroundService"/>
</application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716491025688/cdfb3bc4-e7bf-451b-9b22-860d4a669e96.gif align="center")

## Bound Services

Bound Services allows app components like services to bind themselves to the service. These services perform the task as long as the component is bound to it. Multiple components can bind themselves with a service at a time. To bind a service to the component `bindService()` method is used and components should unbind from the service when they no longer require its functionality to release resources and avoid memory leaks. Example use cases:

1. A messaging app binds to a service for message delivery and synchronization across devices.
    
2. A fitness tracker app binds to a service for real-time location tracking during workouts.
    
3. A file transfer app binds to a service for managing file uploads and downloads in the background.
    

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">All the services run on the main thread or the UI thread by default. So, if you are performing some long-running tasks make sure to create a background thread for that or simply use Coroutines if working in kotlin.</div>
</div>

# Lifecycle of a Service

In Android, there are two categories of services bound services and started services and each of these has a different lifecycle.

### Started Service Lifecycle

This service will start when an app component calls the `startService()` method. After that, the service can keep on running even if the component that started the service is destroyed. To stop a running service we can use the `stopService()` method or the service can stop itself using the `stopSelf()` method.

1. onCreate(): This method gets called when the service is first created using `startService()` method. It is used to perform one-time setup procedures such as initializing resources that will be used by the service.
    
2. onStartCommand(): This method gets called every time when the service is started using `startService()` method. If the service is running already, this method will be called again with a new intent. This is where you define what the service should do in response to the intent passed by `startService()`.
    
3. onDestroy(): The `onDestroy()` method is called when the service is no longer used and is destroyed. The service is destroyed using the `stopService()` method which is called by the app component that was used to start the service or the service can stop itself using the `stopSelf()` method. This is where we should clean up all the resources that are used in the service like removing listeners, unregistering receivers etc.
    

You can check the Foreground Service or Background Service for examples.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716504224733/3c940e53-38d8-4222-b666-87c4bb18b10f.png align="center")

### Bound Service Lifecycle

1. onCreate(): This method gets called when the service is first created using `startService()` method or `bindService()` method.
    
2. onStartCommand(): \[same as in started service\]. It won't get called if `bindService()` is used to start the service.
    
3. onBind(): The `onBind()` method is called when a component binds to the service using the `bindService()` method. If the service is not running already then `onCreate()` will be called first then `onBind()` else the `onBind()` will be called directly. It returns an **IBinder** instance that is used by the component to communicate with the service. The **IBinder** acts as an interface for the component to call method within the service.
    
    What binding really means here is that a component (for example, an activity) establishes a connection with the Service class to call its methods for performing some actions or retrieving data.
    
4. onUnbind(): The `onUnbind()` method is called when the service is `unBound()` from the component using the `unBindServce()` method. This is where we can clean up the resources related to binding that are no longer needed. It returns a boolean value that decides if you want to call the `onRebind(`) method, returns true if you want to call `onRebind()` else return false.
    
5. onRebind(): The `onRebind()` method is called only if `onUnbind()` returns true. This method is used to handle new binding requests after the service has been unbound or to reinitialize the resources that have been cleaned up in the `onUnbind()` method.
    
6. onDestroy(): \[same as in started service\]
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716506560217/90e031b5-6056-4dc8-8790-494c9f50ff71.png align="center")

Here is a simple example code for a Background Service:

**Service Class (BoundService.kt):**

```kotlin
class BoundService: Service() {

    private val TAG = "BoundService"
    private val binder = LocalBinder()

    inner class LocalBinder : Binder() {
        // Return this instance of MyBoundService so clients can call public methods
        fun getService(): BoundService = this@BoundService
    }

    // Called when the service is first created.
    override fun onCreate() {
        super.onCreate()
        showToast("Service is being created")
    }

    // Called every time a component explicitly starts the service using startService()
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        showToast("Service is starting")
        return super.onStartCommand(intent, flags, startId)
    }

    // Called when a component binds to the service using bindService()
    override fun onBind(intent: Intent?): IBinder? {
        showToast("Service is being bound")
        return binder
    }

    // Called when all components have unbound from the service
    override fun onUnbind(intent: Intent?): Boolean {
        showToast("Service is being unbound")
        return super.onUnbind(intent)
    }

    // Called when a client re-binds to the service after it had previously unbound
    override fun onRebind(intent: Intent?) {
        showToast("Service is being re-bound")
        super.onRebind(intent)
    }

    // Called when the service is no longer used and is being destroyed
    override fun onDestroy() {
        showToast("Service is being destroyed")
        super.onDestroy()
    }

    fun showToast(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
    }

    fun getData(): String {
        return "Data from Bound Service"
    }
}
```

**Activity Class (ServiceActivity.kt):**

```kotlin
class ServiceActivity : AppCompatActivity() {

    private lateinit var binding: Activity2serviceBinding
    private lateinit var boundIntent: Intent
    // Reference to the bound service
    private var boundService: BoundService? = null 
    // Flag to track if the service is bound
    private var isBound: Boolean = false

    // ServiceConnection to manage connection with the BoundService
    private val connection = object : ServiceConnection {
        override fun onServiceConnected(className: ComponentName, service: IBinder) {
            val binder = service as BoundService.LocalBinder
            boundService = binder.getService()
            isBound = true
            Log.d(TAG, "Service Connected")
        }

        override fun onServiceDisconnected(arg0: ComponentName) {
            boundService = null
            isBound = false
            Log.d(TAG, "Service Disconnected")
        }
    }

    @RequiresApi(Build.VERSION_CODES.O)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = Activity2serviceBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Create an intent to start BoundService
        boundIntent = Intent(this, BoundService::class.java)

        binding.startService.setOnClickListener {
            // Start the BoundService
            startService(boundIntent)
        }

        binding.stopService.setOnClickListener {
            // Stop the BoundService
            stopService(boundIntent)
        }

        binding.bindService.setOnClickListener {
            // Bind to the BoundService
            bindService(boundIntent, connection, BIND_AUTO_CREATE)
        }

        binding.unBindService.setOnClickListener {
            if (isBound) {
                // Unbind from the BoundService
                unbindService(connection)
                isBound = false
            } else
                Toast.makeText(this, "Service is not bound", Toast.LENGTH_SHORT).show()
        }

        binding.dataFromBoundService.setOnClickListener {
            if (isBound) {
                // Get data from the bound service
                val data = boundService?.getData()
                binding.textView.text = data
            } else
                Toast.makeText(this, "Service is not bound", Toast.LENGTH_SHORT).show()
        }
    }

    override fun onDestroy() {
        // Stop the BoundService (prevent memory leaks)
        stopService(boundIntent)
        super.onDestroy()
    }
}
```

**Manifest File (AndroidManifest.xml):**

```kotlin
<manifest>
<application>
...
<service android:name=".BoundService"/>
</application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716491010012/77b31574-f6ab-4cc1-8f41-c47d64ed010a.gif align="center")

# Conclusion

Services in Android let apps do tasks in the background, keeping the app responsive. There are different types of services: background services run until stopped, bound services let other parts of the app interact with them, and foreground services show notifications to keep users informed. Managing the lifecycle of services (like creating, starting, and destroying them) is important to use resources wisely and avoid issues such as memory leaks. Using threads for long-running tasks prevents the app from freezing. Understanding these concepts helps build better, smoother-running apps.