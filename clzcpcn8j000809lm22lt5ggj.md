---
title: "Implementing MVVM Architecture in Android"
seoTitle: "Mastering MVVM Architecture in Android: Comprehensive Guide"
seoDescription: "Learn best practices, step-by-step implementation, and how MVVM can streamline your app development process for better efficiency and code quality."
datePublished: Fri Aug 02 2024 12:50:02 GMT+0000 (Coordinated Universal Time)
cuid: clzcpcn8j000809lm22lt5ggj
slug: implementing-mvvm-architecture-in-android
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722602679603/5541fd1d-00dd-4903-aa14-b3ba43d8036c.png
tags: android-app-development, repository, android, mvvm, repository-pattern, android-studio, mvvmarchitecture

---

Model-View-ViewModel (MVVM) has become a popular architectural pattern in Android development. This blog post will dive deep into MVVM, exploring its components, benefits, and implementation in Android applications.

# What is MVVM?

MVVM is an architectural pattern that separates an application into three main components:

1. Model
    
2. View
    
3. ViewModel
    

The purpose is to separate the user interface logic from the business logic, making the code more modular and easier to maintain.

# Components of MVVM

### Model

* Represents the data structure and business logic of the application.
    
* Responsible for managing data, performing calculations, and enforcing business rules.
    
* Can include data classes, repositories, and data sources.
    
* Should be independent of the UI and can be reused across different parts of the application.
    

### View

* Represents the UI elements that the user interacts with.
    
* This typically includes Activities, Fragments, and XML layout files.
    
* Observes and reacts to data changes in the ViewModel.
    
* Should be focusing only on displaying data and capturing user inputs.
    
* Doesn't contain any business logic or data manipulation.
    

### ViewModel

* Acts as an intermediary between the Model and View.
    
* Holds and processes all the data needed for the View.
    
* Exposes data to the View through observable properties (often using LiveData or StateFlow).
    
* Handles user interactions passed from the View.
    
* Contains the presentation logic but no direct reference to View components.
    
* It is lifecycle aware, and survives configuration changes (like screen rotations or switching between light/dark themes).
    
* Helps in separating UI logic from UI controllers, making the code more testable and maintainable.
    

# Data Flow in MVVM

a) View to ViewModel:

* User interacts with the View (e.g., button click)
    
* View calls a method in the ViewModel
    
* ViewModel processes the request and updates its data if necessary
    

b) ViewModel to Model:

* ViewModel requests data from the Model
    
* Model fetches or updates data (e.g., from a database or network)
    
* Model returns data to the ViewModel
    

c) ViewModel to View:

* ViewModel updates its observable properties
    
* View observes these changes and updates the UI accordingly
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721930347951/7545b91a-6746-4f67-b55b-033a092a536c.png align="center")

**Example**:

1. **Model Layer**
    

#### User.kt

```kotlin
data class User(val name: String, val email: String)
```

#### UserApi.kt

```kotlin

object UserApi {
    fun getUser(): User {
        // Here you can write the code to fetch data from internet
        // For now, I am returning sample data
        return User("Yashraj", "yash@mail.com")
    }
}
```

2. **ViewModel Layer**
    

UserViewModel.kt

```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user

    fun loadUser() {
        _user.value = UserApi.getUser()
    }
}
```

3. **View Layer**
    

**activity\_main.xml**

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv_data"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Sample"
        android:textSize="34sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/btn_load"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Load Data"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_data" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### MainActivity.kt

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: UserViewModel
    private lateinit var tvData: TextView
    private lateinit var btnLoadData: Button

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tvData = findViewById(R.id.tv_data)

        // ViewModelProvider(this) initializes the provider
        // [UserViewModel::class.java] gets the UserViewModel instance from the provider.
        viewModel = ViewModelProvider(this)[UserViewModel::class.java]

        viewModel.user.observe(this) { user ->
            tvData.text = "${user.name} \n ${user.email}"
        }

        btnLoadData.setOnClickListener {
            viewModel.loadUser()
        }
    }
}
```

However, MVVM is generally used with the **Repository pattern** in Android because this combination provides a robust, scalable, and maintainable architecture.

> The Repository Pattern is a design pattern that provides an abstraction of data sources, such as databases, network services, or caches. It centralizes data access logic and provides a clean API for the rest of the application.

The combination is very effective because of:

* **Separation of Concerns:** MVVM handles UI logic, while Repository manages data access.
    
* **Abstraction of Data Sources:** Repository hides complexity of multiple data sources.
    
* **Improved Testability:** Easier to test ViewModels and data access logic independently.
    
* **Offline-First Approach:** Repository can implement caching for offline use.
    
* **Scalability:** Easier to add new features or data sources without changing ViewModels.
    
* Lifecycle Management: ViewModel handles UI-related data, Repository manages long-running operations.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722086006282/27986b04-2d88-454e-9d8d-1b6013f0f8cb.png align="center")

**Example**:

1. **Model Layer**
    

**User.kt**

```kotlin
data class User(val name: String, val email: String)
```

**UserApi.kt**

```kotlin
object UserApi {
    fun getUser(): User {
        // Here you can write the code to fetch data from internet
        // For now, I am returning sample data
        return User("Yashraj", "yash@mail.com")
    }
}
```

**UserRepository.kt**

```kotlin
class UserRepository(private val api: UserApi) {
    fun getUser(): User {
        // The data you fetch from api can be cache here in local database
        // For now, I am directly calling it
        return api.getUser()
    }
}
```

2. **ViewModel Layer**
    

**UserViewModel.kt**

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user

     fun loadUser() {
        val userData = repository.getUser()
        _user.postValue(userData)
    }
}
```

**ViewModelFactory.kt**

```kotlin
// Custom factory to create UserViewModel instances with UserRepository.
class UserViewModelFactory(private val userRepository: UserRepository) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {

        // Checks if the requested ViewModel class is UserViewModel.
        // If true, creates UserViewModel with UserRepository.
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(userRepository) as T
        }
        // else throws an exception.
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

3. **View Layer**
    

(The activity\_main.xml layout code will remain same as of the above given example.)

**MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var viewModelFactory: UserViewModelFactory
    private lateinit var tvData: TextView
    private lateinit var btnLoadData: Button
    private lateinit var viewModel: UserViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tvData = findViewById(R.id.tv_data)
        btnLoadData = findViewById(R.id.btn_load)

        // UserRepository(UserApi) creates the repository with the API.
        val repository = UserRepository(UserApi)

        // UserViewModelFactory(UserRepository(UserApi)) creates the factory with the repository.
        viewModelFactory = UserViewModelFactory(repository)

        // ViewModelProvider(this, viewModelFactory) initializes the provider with the factory.
        // [UserViewModel::class.java] gets the UserViewModel instance from the provider.
        viewModel = ViewModelProvider(this,viewModelFactory)[UserViewModel::class.java]
        
        // Observe the LiveData from the ViewModel to update the UI when data changes
        viewModel.user.observe(this) { user ->
            tvData.text = "${user.name} \n ${user.email}"
        }

        btnLoadData.setOnClickListener {
            viewModel.loadUser()
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722085999061/abe72059-a7f7-4b4c-9bb0-44176974aada.png align="center")

The output will be the same for both of the above examples in here.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1722173582510/10654a9a-cc57-4d19-8164-a365f1e991c3.gif align="center")

## Advantages and Disadvantages of MVVM in Android

While MVVM offers many benefits, it's important to consider both its advantages and disadvantages when deciding whether to use it in your Android project.

### Advantages

1. **Separation of Concerns**: MVVM clearly separates the user interface (View) from the business logic (ViewModel) and data (Model). This separation makes the code more organized and easier to maintain.
    
2. **Testability**: The clear separation of components makes unit testing much easier, especially for the ViewModel and Model. You can test business logic without dealing with Android framework dependencies.
    
3. **Reusability**: ViewModels can be shared across multiple Views (Activities or Fragments), promoting code reuse and consistency.
    
4. **Lifecycle Management**: ViewModels are lifecycle-aware and survive configuration changes like screen rotations, helping to preserve data and state.
    
5. **Data Binding Support**: MVVM works seamlessly with Android's data binding library, which can significantly reduce boilerplate code in the View layer.
    
6. **Scalability**: As your app grows, MVVM's structure makes it easier to add new features or modify existing ones without affecting other parts of the app.
    

### Disadvantages

1. **Overengineering for Simple Apps**: For very small or simple applications, implementing MVVM might be overkill and could unnecessarily complicate the codebase.
    
2. **Boilerplate Code**: While MVVM can reduce boilerplate in some areas (especially with data binding), it can increase it in others, particularly when setting up the initial architecture.
    
3. **Risk of bulky ViewModels**: Without careful design, ViewModels can become bulky with too much logic, defeating the purpose of separation of concerns.
    
4. **Overuse of Data Binding**: While data binding can simplify View code, overuse can lead to complex XML layouts that are hard to understand and maintain.
    

# Conclusion

MVVM architecture, especially when paired with the Repository pattern, offers a robust solution for Android app development. It effectively separates concerns by dividing the app into View (UI), ViewModel (UI logic and state management), and Model (data and business logic) components, with the Repository acting as a centralized data management layer. This separation enhances testability, maintainability, and scalability of the app.