---
title: "How to Use Content Provider and Content Resolver for data sharing between apps"
seoTitle: "Android Content Providers: Secure Data Sharing Between Apps Explained"
seoDescription: "Learn to use Android Content Providers for secure inter-app data sharing. Explore ContentResolvers, Cursors, and custom providers with SharedPreferences."
datePublished: Wed Jun 26 2024 20:27:59 GMT+0000 (Coordinated Universal Time)
cuid: clxwaf1sv00020al5ghv91ccz
slug: how-to-use-content-provider-and-content-resolver-for-data-sharing-between-apps
canonical: https://blog.yashraj.dev/how-to-use-content-provider-and-content-resolver-for-data-sharing-between-apps
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/w33-zg-dNL4/upload/d21b9574bf8ad970dd2bc9b5a6907242.jpeg
tags: android, content-provider, android-component, content-resolver

---

# Introduction

A content provider in Android provides secure access to a central repository of data that can be stored in a variety of formats, such as a SQLite database, a file, or a web service. It provides a standardized interface for other applications to securely access or modify the data as needed. In activities or fragments, we typically use a ContentResolver object within our application context to interact with content providers. The ContentResolver communicates with the provider, which is an instance of a class implementing ContentProvider.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719351964068/8c526b4d-3fd0-40e7-9dc4-1d5f06a288ce.png align="center")

# What is a Cursor?

When querying a ContentProvider, the result is typically returned as a Cursor object. A Cursor provides read-write access to the result set of a database query. It allows you to move through the data, retrieve column values, and perform other operations on the result set. When using a ContentResolver to query a ContentProvider, you'll typically receive and work with a Cursor to access the returned data.

# What is a Content Provider?

A content provider in Android helps to securely share data between applications by providing an abstraction layer. It exposes a set of CRUD (Create, Read, Update, Delete) operations that can be performed on the data, allowing applications to access data from different sources such as a database, a file, or a web service.

### Example Implementation

Here I have created a custom ContentProvider which will be sharing the data with other apps. Also, I have used SharedPreference for storage here, you can use SQLite database or any other means of storage.

#### Provider App

1. **ContentProvider Implementation (PreferencesContentProvider.kt):**
    

```kotlin
// PreferencesContentProvider class to expose SharedPreferences via a ContentProvider
class PreferencesContentProvider : ContentProvider() {

    // SharedPreferences instance
    private lateinit var preferences: SharedPreferences

    // Called when the provider is created
    override fun onCreate(): Boolean {
        preferences = PreferenceManager.getDefaultSharedPreferences(context)
        return true
    }

    // Handle query requests from clients
    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor? {
        // Create a cursor to hold the key-value pairs from SharedPreferences
        val matrixCursor = MatrixCursor(arrayOf("key", "value"))
        val allEntries = preferences.all
        for ((key, value) in allEntries) {
            // Add each key-value pair to the cursor
            matrixCursor.addRow(arrayOf(key, value.toString()))
        }
        return matrixCursor
    }

    // Return the MIME type of data in the content provider
    override fun getType(uri: Uri): String? {
        return "vnd.android.cursor.dir/vnd.${PreferencesContract.AUTHORITY}.preferences"
    }

    // Handle requests to insert a new row
    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        val editor = preferences.edit()
        values?.let {
            for (key in it.keySet()) {
                val value = it.getAsString(key)
                Log.d("TAG", "insert: $key $value")
                // Save each key-value pair to SharedPreferences
                editor.putString(key, value)
            }
        }
        editor.apply()
        // Notify any listeners that the data has changed
        context?.contentResolver?.notifyChange(uri, null)
        return uri
    }

    // Handle requests to delete one or more rows
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<out String>?): Int {
        val editor = preferences.edit()
        selectionArgs?.forEach { key ->
            // Remove each key specified in the selectionArgs
            editor.remove(key)
        }
        editor.apply()
        // Notify any listeners that the data has changed
        context?.contentResolver?.notifyChange(uri, null)
        return selectionArgs?.size ?: 0
    }

    // Handle requests to update one or more rows
    override fun update(uri: Uri, values: ContentValues?, selection: String?, selectionArgs: Array<out String>?): Int {
        // Use the insert method to handle updates as well
        return insert(uri, values)?.let { 1 } ?: 0
    }
}

// Object to define the contract for accessing the content provider
object PreferencesContract {
    const val AUTHORITY = "com.example.blogs._3content_providers"
    val CONTENT_URI: Uri = Uri.parse("content://$AUTHORITY/preferences")
}
```

MatrixCursor is a mutable cursor implementation backed by an array of objects. Unlike cursors tied to database results, MatrixCursor allows for the dynamic addition of rows directly in memory, making it ideal for creating custom, in-memory datasets. This feature is particularly useful in scenarios where data does not originate from a database, such as data from SharedPreferences. In this ContentProvider implementation, MatrixCursor is used to create a cursor-like interface for SharedPreferences data, enabling seamless integration and manipulation as if it were retrieved from a database. It allows us to:

* Define custom columns ("key" and "value") that match our SharedPreferences structure.
    
* Dynamically add rows for each key-value pair in SharedPreferences.
    
* Return a Cursor object that client application can use to read the data, maintaining a consistent interface with other ContentProviders.
    
* Avoid the need for an actual database, as SharedPreferences is already an efficient key-value store.
    

2. Declare ContentProvider in AndroidManifest.xml:
    

```kotlin
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.blogs">

    <application 
        ...>
        ...
        <provider
         android:name="com.example.blogs._3content_providers.PreferencesContentProvider"
         android:authorities="com.example.blogs._3content_providers"
         android:enabled="true"
         android:exported="true"
         android:grantUriPermissions="true" />

    </application>
</manifest>
```

3. Activity to Save Data (ContentProviderActivity.kt):
    

```kotlin
class ContentProviderActivity : AppCompatActivity() {

    private lateinit var binding: ActivityContentProviderBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityContentProviderBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.apply {
            btnSaveData.setOnClickListener {
                insertPreference("name", etSampleData.text.toString())
                etSampleData.text.clear()
            }
        }
    }

    private fun insertPreference(key: String, value: String) {
        val uri = PreferencesContract.CONTENT_URI
        val values = ContentValues().apply {
            put(key, value)
        }
        contentResolver.insert(uri, values)
    }
}
```

### Some Built-in Content Providers in Android

* **ContactsContract:**
    
    This built-in content provider is used to provide access to the device's contact data such as a phone number or an email address.
    
* **MediaStore:**
    
    This built-in content provider is used to provide access to the device's media files such as videos, photos, and also audio recordings.
    

# What is a Content Resolver?

A `ContentResolver` is an object that acts as an intermediary to communicate with content providers. When we need to access a content provider, we use a `ContentResolver` object within our application context. The `ContentResolver` sends data requests to the provider, which performs the requested actions and returns the results back to the `ContentResolver`. This process allows secure data sharing between different applications.

The `ContentResolver` is used to perform operations like querying, inserting, updating, and deleting data through the ContentProvider. It uses URIs to specify the content and operations.

Here's how you can use the `ContentResolver` to query the data exposed by the `PreferencesContentProvider`. In the code below, I have shown how you can receive the data shared by our provider app:

```kotlin
class ResolverActivity : AppCompatActivity() {

    private lateinit var binding: ActivityResolverBinding
    private lateinit var text: String

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityResolverBinding.inflate(layoutInflater)
        setContentView(binding.root)

        queryPreferences()
        binding.btnGetData.setOnClickListener {
            queryPreferences()
            binding.tvProviderData.text = text
        }
    }

    private fun queryPreferences() {
        // Define the URI for the content to access
        val uri = PreferencesContract.CONTENT_URI
    
        // Define the columns to retrieve
        val projection = arrayOf("key", "value")
    
        // Use the content resolver to query the content provider
        val cursor = contentResolver.query(uri, projection, null, null, null)

        cursor?.apply {
            // Iterate through the cursor rows
            while (moveToNext()) {
                // Get the key and value from the current row
                val key = getString(getColumnIndexOrThrow("key"))
                val value = getString(getColumnIndexOrThrow("value"))
            
            // Assign the value to text variable
                Log.d("TAG", "queryPreferences: Key: $key, Value: $value")
                text = value
            }
            // Close the cursor to release resources
            close()
        }
    }
}

// Define the PreferencesContract object to store constants related to the content provider
object PreferencesContract {
    // The authority of the content provider
    const val AUTHORITY = "com.example.blogs._3content_providers"
    
    // The URI for the preferences table
    val CONTENT_URI: Uri = Uri.parse("content://$AUTHORITY/preferences")
}
```

In the manifest file, we will write the package name of our provider app

```kotlin
<manifest>
   <queries>
      <package android:name="com.example.blogs" />
   </queries>
   <application
        ...>
        ....
    </application>
<manifest>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1719260751161/183bc59b-3068-41f7-b1b2-9ee2b982827e.gif align="center")

# Conclusion

By using a ContentProvider, you can securely share data between applications, providing a standardized interface for CRUD operations. The example shows how to create a custom ContentProvider to expose SharedPreferences data and access it from another application using a ContentResolver. Using proper URI matching and handling ensures secure and organized access to the shared data. Understanding the role of the ContentResolver is critical, as it serves as the bridge between the client application and the ContentProvider, allowing seamless data access and manipulation.