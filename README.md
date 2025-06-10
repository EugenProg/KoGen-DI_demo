# KoGen Di: User Guide

**KoGen Di** is a library for Android that uses code generation (KSP) to create a simple, independent, and type-safe Dependency Injection container, perfectly suited for multi-module projects.

**Core Principles:**
* **Zero Initialization:** Does not require binding to the `Application` class.
* **Independence:** Each module can have its own, isolated DI container.
* **Minimal Code:** Most dependencies are registered automatically using annotations.

---

## 🚀 Installation and Setup

The library is published on **Maven Central**.

### Step 1: Apply the KSP Plugin

First, make sure the KSP plugin is applied to your project.

**In your root `build.gradle.kts` file:**
```kotlin
plugins {
    // ...
    id("com.google.devtools.ksp") version "2.0.0-1.0.22" apply false // Use the KSP version that matches your Kotlin version
}
```

**In your module's `build.gradle.kts` file:**
```kotlin
plugins {
    // ...
    id("com.google.devtools.ksp")
}
```

### Step 2: Add Dependencies

```kotlin
dependencies {
    // Your version may vary
    implementation("io.github.eugenprog:android-di:1.0.1")
    ksp("io.github.eugenprog:android-di:1.0.1")

    // Optional: for ViewModel support
    implementation(platform("androidx.compose:compose-bom:2024.06.00"))
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.2")
}
```
**Important:** *The `implementation` and `ksp` versions of your library must match.*

### Step 3: Configure Code Generation

In your module's `build.gradle.kts`, add a `ksp` block to configure the generator.
```kotlin
ksp {
    // Required parameter: your project's package name
    arg("packageName", "com.myawesome.project")
    // Optional parameter to enable ViewModel support
    arg("includeViewModelInjector", "true") 
}
```
* `packageName` (**required**) – Needed so that the generated classes are placed in the correct namespace of your project.
* `includeViewModelInjector` (optional) – Accepts `true` or `false`. Enables code generation for `ViewModel` support. Defaults to `false`.

---

## ⚙️ How to Use

### 1. Declaring Dependencies

There are two ways to declare a dependency:

#### A) For Regular Classes (`@KoGenComponent`)
Simply annotate your class (e.g., a repository or service implementation) with `@KoGenComponent`. Set `singleton` to `true` if it should be reused for every request.
```kotlin
@KoGenComponent(singleton = true)
class UserProfileServiceImpl(
    private val source: UserProfileSource,
) : UserProfileService {
    // ...
}
```

#### B) For Dependencies with Complex Creation (`@KoGenBean`)
For objects that require complex creation logic (e.g., from `Retrofit` or `Room`), create a function that returns the object and annotate it with `@KoGenBean`.
```kotlin
@KoGenBean(singleton = true)
fun provideUserProfileSource(
    context: Context,
): UserProfileSource {
    // ... object creation logic
}
```

#### C) For ViewModels (`@KoGenViewModel`)
Annotate your `ViewModel` class with the corresponding annotation.
```kotlin
@KoGenViewModel
class MyScreenViewModel(
    private val userProfileService: UserProfileService
) : ViewModel() {
    // ...
}
```

### 2. Retrieving Dependencies

#### Main Entry Point: `inject()`
Use this global function in your `Activity`, `Fragment`, and other classes.
```kotlin
class MyActivity : AppCompatActivity() {
    // Lazy injection
    private val userProfileService: UserProfileService by inject()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Direct injection
        val anotherService: AnotherService = inject()
    }
}
```

#### Second Entry Point: `koGenViewModel()`
Use this helper in your Composable screens.
```kotlin
@Composable
fun MyScreen(
    viewModel: MyScreenViewModel = koGenViewModel()
) {
    // ...
}
```
---

## ⚠️ Important Notes

1.  **Code Appears After the First Build.** The functions `inject()` and `koGenViewModel()` are physically absent from the code until you build the project at least once. Don't be alarmed if the IDE complains about their absence.

2.  **ViewModel Support is Optional.** To enable it, you must pass the `includeViewModelInjector` argument in your KSP settings as shown in the setup guide.

3.  **KSP Can "Go Crazy".** In rare cases, the KSP cache can become corrupted. The standard treatment is a full project clean (`./gradlew clean`) and a rebuild.

```
