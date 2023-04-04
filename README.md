# Coroutine-and-Kotlin-Flow
## What is Coroutine 
- A coroutine is a concurrency design pattern that you can use on Android to simplify code that executes asynchronously.
### Features
#### Lightweight: 
- You can run many coroutines on a single thread due to support for suspension, which doesn't block the thread where the coroutine is running. Suspending saves memory over blocking while supporting many concurrent operations.
#### Fewer memory leaks: 
- Use structured concurrency to run operations within a scope.
#### Built-in cancellation support: 
- Cancellation is propagated automatically through the running coroutine hierarchy.
#### Jetpack integration: 
- Many Jetpack libraries include extensions that provide full coroutines support. Some libraries also provide their own coroutine scope that you can use for structured concurrency.
#### dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
}

#### Suspend Function / launch / async: 
 ### suspend function : 
-  When we add the suspend keyword in the function, all the cooperations are automatically done for us. We don't have to use when or switch case. When we write like below:

  - suspend fun fetchUser(): User {
     // make network call
     // return user
}
- fun showUser(user: User) {
    // show user
}


override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    CoroutineScope(Dispatchers.IO).launch {
        val user = fetchUser() // fetch on IO thread
        showUser(user) // back on UI thread
    }

}

#### launch / async :
 - launch is the functions in Kotlin to start the Coroutines.
 - The difference is that the launch{} returns a Job and does not carry any resulting value whereas the async{} returns an instance of Deferred<T>, which has an await() function that returns the result of the coroutine like we have future in Java in which we do future.get() to get the result.

In other words:

 #### launch:  - fire and forget.
#### async: - perform a task and return a result.
Let's take an example to learn launch vs async.

We can use the launch as below:
     *** val job = GlobalScope.launch(Dispatchers.Default) {
    // do something and do not return result
}

But when we need the result back, we need to use the async.
    val deferredJob = GlobalScope.async(Dispatchers.Default) {
    // do something and return result, for example 10 as a result
    return@async 10
}
val result = deferredJob.await()

 - Here, we get the result using the await().

 - In async also, we can use the Deferred job object to get a job's status or to cancel it.


| launch              | async                                                                                                                        |
| --------------------|:---------------------------------------:|
| Fire and forget.                                              | Perform a task and return a result.
| launch{} returns a Job and does not carry any resulting value.| async{} returns an instance of Deferred<T>, which has an await() function that returns the result of the coroutine.                                                   
| If an y exception comes inside the launch block, it crashes the application if we have not handled it. | If any exception comes inside the async block, it is stored inside the resulting Deferred and is not delivered anywhere else, it will get silently dropped unless we handle it. |                                                   
    
 ### withContext dispatchers scope : 
    - withContext is a suspend function through which we can do a task by providing the Dispatchers on which we want the task to be done.

 - withContext does not create a new coroutine, it only shifts the context of the existing coroutine and it's a suspend function whereas launch and async create a new      coroutine and they are not suspend functions.

 -  Let's see the code for the withContext.
    
``` private suspend fun doLongRunningTask(): Int {
    return withContext(Dispatchers.Default) {
        // your code for doing a long running task
        // Added delay to simulate
        delay(2000)
        return@withContext 10
 }
 }
```
# Dispatchers in Kotlin Coroutines 
- Dispatchers help Coroutines in deciding the thread on which the task has to be done. We use Coroutines to perform certain tasks efficiently. 
      Coroutines run the task on a particular thread. This is where the Dispatchers come into play. 
      Coroutines take the help of Dispatchers in deciding the thread on which the task has to be done.
    
 - Very frequently we use the following Dispatchers in our Android project:
 ## Dispatchers.Default
 #### We should use Dispatchers.Default to perform CPU-intensive tasks.
 - Example use cases:
 - Doing heavy calculations like Matrix multiplications.
 - Doing any operations on a bigger list present in the memory like sorting, filtering, searching, etc.
 - Applying the filter on the Bitmap present in the memory, NOT by reading the image file present on the disk.
 - Parsing the JSON available in the memory, NOT by reading the JSON file present on the disk.
 - Scaling the bitmap already present in the memory, NOT by reading the image file present on the disk.
 - Any operations on the bitmap that are already present in the memory, NOT by reading the image file present on the disk.
    
 ``` 
 launch(Dispatchers.Default) {
    // Your CPU-intensive task
}
```
## Dispatchers.IO
#### We should use Dispatchers.IO to perform disk or network I/O-related tasks.

- Example use cases:

- Any network operations like making a network call.
- Downloading a file from the server.
- Moving a file from one location to another on disk.
- Reading from a file.
- Writing to a file.
- Making a database query.
- Loading the Shared Preferences.
```
launch(Dispatchers.IO) {
    // Your IO related task
}
```
## Dispatchers.IO
#### We should use Dispatchers.Main to run a coroutine on the main thread of Android. We all know where we use the main thread of Android. Mainly at the places where we interact with the UI and perform small tasks.

- Example use cases:
- Performing UI-related tasks.
- Any small tasks like any operations on a smaller list present in the memory like sorting, filtering, searching, etc.
    
```
launch(Dispatchers.Main) {
    // Your main thread related task
}
```
# lifecycleScope, viewModelScope, GlobalScope
## viewModelScope
- Coroutine scope attached with your viewmodel
- Coroutines in this scope will be cancelled automatically when viewmodel is cleared. we don't need to manually cancel the coroutine.
````
 init {
    viewModelScope.launch{
    delay(3000)
    Log.d("","Hello viewModelScope")
    }
    }
   override fun onCleared{
    Log.d("","viewModel Destroyed")
    }
````
## lifeCycleScope
- Coroutine scope attached with lifecycle (Activity or Fragment)
- Coroutines in this scope will be cancelled automatically when lifecycle is destroyed. we don't need to manually cancel the coroutine.
````
 lifecycleScope.launch{
    delay(2000)
    val intent = Intent(this@MainActivity,SecondActivity::class.java)
    startActivity(intent)
    finish()
    }
````
# GlobalScope
- A global CoroutineScope not bound to any job. Global scope is used to launch top-level coroutines which are operating on the whole application lifetime and are not cancelled prematurely.

- Active coroutines launched in GlobalScope do not keep the process alive. They are like daemon threads.

- This is a delicate API. It is easy to accidentally create resource or memory leaks when GlobalScope is used. A coroutine launched in GlobalScope is not subject to the principle of structured concurrency, so if it hangs or gets delayed due to a problem (e.g. due to a slow network), it will stay working and consuming resources. For example, consider the following code:
```
fun loadConfiguration() {
    GlobalScope.launch {
        val config = fetchConfigFromServer() // network request
        updateConfiguration(config)
    }
}
```
- A call to loadConfiguration creates a coroutine in the GlobalScope that works in background without any provision to cancel it or to wait for its completion.

 



    
