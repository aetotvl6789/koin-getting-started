---
title: Android ViewModel
---

> This tutorial lets you write an Android/Kotlin application and use Koin inject and retrieve your ViewModel components.

## Get the code

:::info
[The source code is available at on Github](https://github.com/InsertKoinIO/koin-getting-started/tree/main/getting-started-koin-android)
:::

## Gradle Setup

Add the Koin Android dependency like below:

```groovy
// Add Maven Central to your repositories if needed
repositories {
	mavenCentral()    
}
dependencies {
    // Koin for Android
    implementation "io.insert-koin:koin-android:$koin_version"

    // Koin Test
    testImplementation "io.insert-koin:koin-test:$koin_version"
    testImplementation "io.insert-koin:koin-test-junit4:$koin_version"
}
```

## Our components

Let's create a HelloRepository to provide some data:

```kotlin
interface HelloRepository {
    fun giveHello(): String
}

class HelloRepositoryImpl() : HelloRepository {
    override fun giveHello() = "Hello Koin"
}
```

Let's create a ViewModel class, for consuming this data:

```kotlin
class MyViewModel(val repo : HelloRepository) : ViewModel() {

    fun sayHello() = "${repo.giveHello()} from $this"
}
```

## Writing the Koin module

Use the `module` function to declare a module. Let's declare our first component. 

Classical DSL way:

```kotlin
val appModule = module {

    // single instance of HelloRepository
    single<HelloRepository> { HelloRepositoryImpl() }

    // MyViewModel ViewModel
    viewModel { MyViewModel(get()) }
}
```

Constructor DSL way:

```kotlin
val appModule = module {

    // single instance of HelloRepository
    singleOf(::HelloRepositoryImpl) { bind<HelloRepository>() }

    // MyViewModel ViewModel
    viewModelOf(::MyViewModel)
}
```

:::info
we declare our MyViewModel class as a `viewModel` in a `module`. Koin will give a `MyViewModel` to the lifecycle ViewModelFactory and help bind it to the current component.
:::

## Start Koin

Now that we have a module, let's start it with Koin. Open your application class, or make one (don't forget to declare it in your manifest.xml). Just call the `startKoin()` function:

```kotlin
class MyApplication : Application(){
    override fun onCreate() {
        super.onCreate()
        // Start Koin
        startKoin{
            androidLogger()
            androidContext(this@MyApplication)
            modules(appModule)
        }
    }
}
```

## Injecting dependencies

The `MyViewModel` component will be created with `HelloRepository` instance. To get it into our Activity, let's inject it with the `by viewModel()` delegate injector: 

```kotlin
class MyViewModelActivity : AppCompatActivity() {
    
    // Lazy Inject ViewModel
    val myViewModel: MyViewModel by viewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_simple)

        //...
    }
}
```

:::info
>The `by viewModel()` function allows us to retrieve a ViewModel instance from Koin, linked to the Android ViewModelFactory.

> The `getViewModel()` function is here to retrieve directly an instance (non lazy)
:::

## Verifying your graph

We can ensure that our Koin configuration is good before launching our app, by verifying our Koin configuration with a simple JUnit Test.

You need to declare a `MockProviderRule` to declare how you mock a class (here for example, we use Mockito). 

The `checkModules` function allow to verify the given Koin modules:

```kotlin
class CheckModulesTest : KoinTest {

    // Declare Mock with Mockito
    @get:Rule
    val mockProvider = MockProviderRule.create { clazz ->
        Mockito.mock(clazz.java)
    }

    // verify the Koin configuration
    @Test
    fun checkAllModules() = checkModules {
        modules(appModule)
    }
}
```
