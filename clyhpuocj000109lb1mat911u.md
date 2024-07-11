---
title: "Fragments in Android"
seoTitle: "Understanding Android Fragment Lifecycle for Dynamic UI Development"
seoDescription: "Discover how to leverage Android fragments to create dynamic and flexible user interfaces. Learn the lifecycle and benefits for managing complex layouts."
datePublished: Thu Jul 11 2024 20:23:12 GMT+0000 (Coordinated Universal Time)
cuid: clyhpuocj000109lb1mat911u
slug: fragments-in-android
canonical: https://blog.yashraj.dev/fragments-in-android
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720726329799/b77e7d5c-dd4f-44b7-b7a2-1b539d7df61e.png
tags: android-app-development, android, lifecycle, fragment

---

### What are Fragments?

In Android, a fragment is a reusable portion of your app's user interface. You can think of a fragment as a mini-activity within an activity. Like activities, fragments have their own lifecycle, meaning they go through stages such as creation, starting, pausing, stopping, and destroying. A fragment receives its own input events, and you can add or remove it while the activity is running.

### Why Use Fragments?

Here are some key reasons why fragments are useful:

1. **Modularity**: Fragments allow you to divide your UI into smaller, manageable pieces. This makes your code easier to manage and maintain.
    
2. **Reusability**: You can reuse fragments in multiple activities, reducing redundancy and saving time.
    
3. **Adaptability**: Fragments help you create flexible UIs that work on different screen sizes and orientations.
    
    For example, a tablet can display multiple fragments side by side, while a phone might show them one at a time.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720527672096/bdaac120-7d15-473c-9283-8e589ef336fa.png align="center")

Image Source: [https://developer.android.com/guide/fragments](https://developer.android.com/guide/fragments)

### How to Create a Fragment?

Creating a fragment is similar to creating an activity. Here’s a basic example:

1. Create a new class `DemoFragment` that extends `Fragment` and override its methods to insert your app logic.
    

```kotlin
class DemoFragment : Fragment() {

    private val TAG = "DemoFragment"

    override fun onAttach(context: Context) {
        super.onAttach(context)
        Log.d(TAG, "onAttach: Fragment attached to activity")
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Log.d(TAG, "onCreate: Fragment create")
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        
        Log.d(TAG, "onCreateView: Fragment create view")
       // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_demo, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Log.d(TAG, "onViewCreated: Fragment view created")
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart: Fragment started")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume: Fragment resumed")
    }-

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause: Fragment paused")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop: Fragment stopped")
    }

    override fun onDestroyView() {
        super.onDestroyView()
        Log.d(TAG, "onDestroyView: Fragment view destroyed")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy: Fragment destroyed")
    }

    override fun onDetach() {
        super.onDetach()
        Log.d(TAG, "onDetach: Fragment detached from activity")
    }

}
```

2. In your activity, instantiate and add the fragment using `FragmentManager`.
    

```kotlin
class FragmentHolderActivity : AppCompatActivity() {

    private val TAG = "FragmentHolderActivity"
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_fragment_holder)
        
        if (savedInstanceState == null) {
        // Get the FragmentManager to interact with fragments associated with this activity
            val fragmentManager: FragmentManager = supportFragmentManager
        // Begin a new fragment transaction
            val fragmentTransaction: FragmentTransaction = fragmentManager.beginTransaction()
        // Create a new instance of DemoFragment
            val fragment = DemoFragment()
        // Add the fragment to the container specified by its ID (R.id.fragment_container)
            fragmentTransaction.add(R.id.fragment_container, fragment)
        // Commit the transaction to apply the changes
            fragmentTransaction.commit()
        }
        Log.d(TAG, "onCreate: Activity created")
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart: Activity started")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume: Activity resumed")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause: Activity paused")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop: Activity stopped")
    }

    override fun onRestart() {
        super.onRestart()
        Log.d(TAG, "onRestart: Activity restarted")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy: Activity destroyed")
    }
}
```

> Note: The fragment transaction is created only when `savedInstanceState` is null. This ensures the fragment is added just once when the activity starts initially. When a configuration change happens and the activity is recreated, `savedInstanceState` isn't null anymore. In this case, the fragment isn't added again; it's automatically restored from `savedInstanceState`.

3. Include a `FragmentContainerView` in your activity's layout file where the fragment will be placed.
    

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/fragment_container" />
```

**FragmentManager**: The main class to manage fragment operations like adding, removing, replacing fragments, and handling the fragment back stack.

**FragmentTransaction**: Used to perform a set of fragment operations (like adding, replacing, or removing fragments) in a single atomic action.

* **Adding Fragments**: Attach fragments to an activity.
    
* **Removing Fragments**: Detach fragments from an activity.
    
* **Replacing Fragments**: Replace existing fragments with new ones.
    
* **Finding Fragments**: Find fragments by their ID or tag.
    

### Fragment Lifecycle

The lifecycle of fragment goes through the following stages:

1. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720727885942/05eb5ffe-53d9-4a5d-b5cb-4b2978bcdd5f.png align="center")
    
    **onAttach()**: Called when the fragment is first attached to its host activity.
    
2. **onCreate()**: The fragment is created. The saved instance state can be used to restore the fragment's previous state.
    
3. **onCreateView()**: Called to create the view hierarchy associated with the fragment.
    
4. **onViewCreated()**: Called immediately after `onCreateView()`. You can perform any additional setup of the fragment's view here.
    
5. **onStart()**: Called when the fragment is visible to the user.
    
6. **onResume()**: Called when the fragment is visible and actively running.
    
7. **onPause()**: Called when the fragment is not actively running.
    
8. **onStop()**: Called when the fragment is no longer visible to the user.
    
9. **onDestroyView()**: Called to clean up resources associated with the view.
    
10. **onDestroy()**: Called when the fragment is destroyed.
    
11. **onDetach()**: Called when the fragment has been detached from the activity.
    

* **Lifecycle methods execution when the app is launched** (refer to the code above on how to create a fragment)**:**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720540516795/ce9d28ad-f0fe-440f-a3b7-04d4cccf4067.gif align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720539293310/f0a62cff-e862-4ef2-8835-62a62d75a0fd.png align="center")

This demonstrates the sequence of fragment and activity lifecycle methods executed when the app is launched.

* **Lifecycle methods execution when the app is closed and removed from the background** (refer to the code above on how to create a fragment)**.**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720540527898/b9e9a462-7767-408a-b663-35f43fdaa38f.gif align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1720539485162/e04d5475-9847-4967-bdca-3c8659e90ed8.png align="center")

This demonstrates the order of fragment and activity lifecycle methods executed when the app is closed and removed from the background.

### Conclusion

Fragments serve as modular components that allow you to build dynamic and [f](http://stackoverflow.com/questions/39418249/meaning-of-fragment-having-its-own-lifecycle)lexible UIs in Android. They make it easier to manage complex interfaces and ensure your app works well on various devices. By using fragments, you can create better and more adaptable Android applications.