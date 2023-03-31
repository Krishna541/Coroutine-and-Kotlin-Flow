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
     - val job = GlobalScope.launch(Dispatchers.Default) {
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

