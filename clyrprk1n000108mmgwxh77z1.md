---
title: "Tasks, Backstack and Launch Modes"
seoTitle: "Learn about Tasks, Backstack, and Launch Modes in Android"
seoDescription: "Learn to manage tasks, backstack, and launch modes in Android for better app navigation and performance."
datePublished: Thu Jul 18 2024 20:18:28 GMT+0000 (Coordinated Universal Time)
cuid: clyrprk1n000108mmgwxh77z1
slug: tasks-backstack-and-launch-modes
canonical: https://blog.yashraj.dev/tasks-backstack-and-launch-modes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721248033390/d51e5a88-6c2c-49ea-a802-4dd400f3a0a7.png
tags: android-app-development, android, tasks, launch-mode, backstack

---

# What is a Task?

A task in Android is a collection of activities that users interact with when trying to do something in your app, representing an application’s workflow where each activity is a focused action that a user can do. A task usually starts when a user launches an application and can include multiple activities across different applications. When you tap the app's launcher icon, the system searches for an existing task that matches the Intent and Activity. If it finds one, it resumes that task, bringing you back to where you left off. If no matching task is found, a new task is created with the newly launched activity as the base activity on the task’s back stack.

Here is an example showing how the app closes on Activity 3 and, after the app is launched again, it resumes from the same activity:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720882664769/0c0d5293-0476-4ad4-811f-1cbf9a12816d.gif align="center")

# Understanding the Backstack

The backstack in Android is a Last In, First Out (LIFO) stack that maintains the order of activities the user interacts with. When a new activity starts, it is pushed onto the backstack. Pressing the back button pops the topmost activity off the stack, resuming the previous activity.

Here’s an example showing how activities are added to the backstack:

```kotlin
class Activity1 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_1)

        val btnLaunch = findViewById<Button>(R.id.btn_launch_activity_2)
        btnLaunch.setOnClickListener {
            startActivity(Intent(this, Activity2::class.java))
        }
    }
}
```

```kotlin
class Activity2 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_2)

        val btnLaunch = findViewById<Button>(R.id.btn_launch_activity_3)
        btnLaunch.setOnClickListener {
            startActivity(Intent(this, Activity3::class.java))
        }
    }
}
```

```kotlin
class Activity3 : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_3)
    }
}
```

1. When the app is first launched, Activity1 is added to the backstack.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720876997925/09ba6c50-c507-407f-933f-8c47abee2e50.gif align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720876965442/00bf8115-4c6e-4a33-b65f-4bd264f03def.png align="center")

2. Upon clicking "Launch Activity 2" button in Activity1, Activity2 is launched and added to the backstack.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720877009120/66f5ed54-08e9-4796-9c7d-698389b329cf.gif align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720877019214/496e7f3f-0888-4416-941d-beba8531436a.png align="center")

3. Similarly, clicking "Launch Activity 3" button in Activity2 launches Activity3, adding it to the backstack.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720877038692/b0dccf6d-2d4f-4ffd-b1f9-327eebae1ea8.gif align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720877055391/0be4eda2-25a2-4788-9b19-26cabb57d011.png align="center")

In this example, starting from `Activity1`, launching `Activity2` adds it to the backstack. Similarly, launching `Activity3` from `Activity2` pushes it onto the backstack. If you press the back button in `Activity3` will pop it off the stack, resuming `Activity2`.

# Launch Modes

Launch modes determine how activities are launched and associated with tasks. To specify launch modes in an Android application, you define them in the `AndroidManifest.xml` file within the `<activity>` element using the `android:launchMode` attribute. Example:

```xml
<manifest>
    <application ...>
        ...
        <activity
            android:name=".DemoActivity"
            android:launchMode="standard">
        </activity>
    </application>
</manifest>
```

There are four launch modes in Android:

1. **Standard (default)**
    
2. **SingleTop**
    
3. **SingleTask**
    
4. **SingleInstance**
    

### Standard

This default launch mode creates a new instance of the activity each time it’s launched, even if an instance already exists. For example, if activities A, B, and C are in a task, and B is launched again, the task becomes A → B → C → B, with a new instance of B. You do not need to set anything for the standard launch mode since it is the default. However, for clarity, you can explicitly set it as follows:

```xml
<manifest>
    <application ...>
        ...
        <activity
            android:name=".Activity_B"
            android:launchMode="standard">
        </activity>
    </application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721151092832/9b49cd05-39fb-4df5-bfd4-c5634eb0b28e.png align="center")

### SingleTop

If an instance of the activity is already at the top of the backstack, it is not recreated; instead, the existing instance receives the intent via the `onNewIntent()` method. If a new instance is not present at the top then a new instance will be created. For example, we have activities A, B, and C, launching B will create a new instance of activity B as it is not at the top then we will get A → B → C → B, launching B again results in A → B → C → B (same instance), where `onNewIntent()` is called. To specify the `SingleTop` launch mode:

```xml
<manifest>
    <application ...>
        ...
        <activity
            android:name=".Activity_B"
            android:launchMode="singleTop">
        </activity>
    </application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721151005049/776c4cbf-b6df-4d0d-9cce-cd72e35ccc2c.png align="center")

### SingleTask

An activity with the singleTask launch mode can have only one instance in the system at a time. If no instance exists, a new one is created. If an instance is already present, instead of creating a new one, the existing instance handles the intent through the `onNewIntent()` method. For example, with activities A, B, and C, launching D (singleTask) results in A → B → C → D, a new instance of D is created. If B is also singleTask and is launched, the task becomes A → B, removing C and D. This means that the existing instance of B is reused, and the intent data is routed through the `onNewIntent()` method. To specify the `SingleTask` launch mode:

```xml
<manifest>
    <application ...>
        ...
        <activity
            android:name=".Activity_B"
            android:launchMode="singleTask">
        </activity>
    </application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721150990383/436d67b3-7850-418c-ba91-c4c6e9830fa0.png align="center")

### SingleInstance

Similar to SingleTask but stricter, this mode allows the activity to be the only one in its task, making it completely isolated. For example, launching D (singleInstance) from a task with A, B, and C results in two separate tasks: Task 1 (A → B → C) and Task 2 (D). If D is launched again, the existing instance handles the intent with `onNewIntent()`. To specify the `SingleInstance` launch mode:

```xml
<manifest>
    <application ...>
        ...
        <activity
            android:name=".Activity_D"
            android:launchMode="singleInstance">
        </activity>
    </application>
</manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721151457054/55ae8af5-d1ce-4424-9b87-ed014c994b26.png align="center")