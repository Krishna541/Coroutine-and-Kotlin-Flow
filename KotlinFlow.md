

# Kotlin Flow API
- Flow API in Kotlin is a better way to handle the stream of data asynchronously that executes sequentially.
- Add the following in the app's build.gradle,
```
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.3"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.3"
```
- and in the project's build.gradle add,
```
classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.61"
```
## Flow , FlowOn , emit , collect , map , filter 
```
setupFlow()
```
- Here, setupFlow () is the function where we will define the flow .
- We will declare a lateinit variable of Flow of Int type,
```
lateinit var flow: Flow<Int>
fun setupFlow(){
    flow = flow {
        Log.d(TAG, "Start flow")
        (0..10).forEach {
            // Emit items with 500 milliseconds delay
            delay(500)
            Log.d(TAG, "Emitting $it")
            emit(it)

        }
    }.flowOn(Dispatchers.Default)
}
```
Here,

- We will emit numbers from 0 to 10 at 500ms delay.
- To emit the number we will use emit() which collects the value emitted . It is part of FlowCollector which can be used as a receiver.
   and, at last, we use flowOn operator which means that shall be used to change the context of the flow emission. Here, we can use different Dispatchers like IO,        Default, etc.
```
private fun setupClicks() {
    button.setOnClickListener {
        CoroutineScope(Dispatchers.Main).launch {
            flow.collect {
                Log.d(TAG, it.toString())
            }
        }
    }
}
```
When we click the button we will print the values one by one.

Here,

- flow.collect now will start extracting/collection the value from the flow on the Main thread as Dispatchers.Main is used in launch coroutine builder in                  coroutineScope
Now, the screen looks like,


![screen-ui-kotlin-flow](https://user-images.githubusercontent.com/16682754/230375474-c37f6b42-a016-40de-82cb-d19203383abb.png)

- And the output which will be printed in Logcat is,
```
D/MainActivity: Start flow
D/MainActivity: Emitting 0
D/MainActivity: 0
D/MainActivity: Emitting 1
D/MainActivity: 1
D/MainActivity: Emitting 2
D/MainActivity: 2
D/MainActivity: Emitting 3
D/MainActivity: 3
D/MainActivity: Emitting 4
D/MainActivity: 4
D/MainActivity: Emitting 5
D/MainActivity: 5
D/MainActivity: Emitting 6
D/MainActivity: 6
D/MainActivity: Emitting 7
D/MainActivity: 7
D/MainActivity: Emitting 8
D/MainActivity: 8
D/MainActivity: Emitting 9
D/MainActivity: 9
D/MainActivity: Emitting 10
D/MainActivity: 10
```
- Let's say we update the setupFlow () function like,
```
private fun setupFlow() {
    flow = flow {
        Log.d(TAG, "Start flow")
        (0..10).forEach {
            // Emit items with 500 milliseconds delay
            delay(500)
            Log.d(TAG, "Emitting $it")
            emit(it)
        }
    }.map {
        it * it
    }.flowOn(Dispatchers.Default)
}
```
- Here you can see we added map operator which will take each and every item will square itself and print the value.

- Anything, written above flowOn will run in background thread.

# Builders in Flow
Flow builders are nothing but the way to build Flows. There are 4 types of flow builders,
### flowOf() 
-It is used to create flow from a given set of values. For Eg.
```
flowOf(4, 2, 5, 1, 7).onEach { delay(400) }.flowOn(Dispatchers.Default)
```
- Here , flowOf() takes fixed values and prints each of them after a delay of 400ms. When we attach a collector to the flow, we get the output
```
D/MainActivity: 4
D/MainActivity: 2
D/MainActivity: 5
D/MainActivity: 1
D/MainActivity: 7
```
### asFlow()
- It is an extension function that helps to convert type into flows. For Eg,
```
D/MainActivity: 1
D/MainActivity: 2
D/MainActivity: 3
D/MainActivity: 4
D/MainActivity: 5
```
- The output is the number being printed from 1 to 5, whichever were present in the range.
#### flow{} 
- This example has been explained in the Android Example above. This is a builder function to construct arbitrary flows.
#### channelFlow{} - 
- This builder creates cold-flow with the elements using send provided by the builder itself. For Example,

````
channelFlow {
    (0..10).forEach {
        send(it)
    }
}.flowOn(Dispatchers.Default)
````
- This will print,
```
D/MainActivity: 0
D/MainActivity: 1
D/MainActivity: 2
D/MainActivity: 3
D/MainActivity: 4
D/MainActivity: 5
D/MainActivity: 6
D/MainActivity: 7
D/MainActivity: 8
D/MainActivity: 9
D/MainActivity: 10
```
# Zip Operator
- First, we will declare two lateinit variables of Flow of String type,
```
lateinit var flowOne: Flow<String>
lateinit var flowTwo: Flow<String>
```
Now, the two functions setupFlow () and setupClicks () will be called in onCreate() of MainActivity.
```
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    setupFlow()
    setupClicks()

}
```
and in the setupFlow() we will initialize two flows to the two variables,
```
private fun setupFlow() {
    flowOne = flowOf("Himanshu", "Amit", "Janishar").flowOn(Dispatchers.Default)
    flowTwo = flowOf("Singh", "Shekhar", "Ali").flowOn(Dispatchers.Default)

}
```
and in setupClicks () we will zip both the flows using zip operator,
```
private fun setupClicks() {
    button.setOnClickListener {
        CoroutineScope(Dispatchers.Main).launch {
            flowOne.zip(flowTwo)
            { firstString, secondString ->
                "$firstString $secondString"
            }.collect {
                Log.d(TAG, it)
            }
        }
    }
}
```
Here,

- when the button is clicked the scope is launched.
- flowOne is zipped with flowTwo to give a pair of values that we have created a string and,
- then we collect it and print it in Logcat. The resulting output will be,
```
D/MainActivity: Himanshu Singh
D/MainActivity: Amit Shekhar
D/MainActivity: Janishar Ali
```

# Cold Flow | Hot Flow

| Cold Flow             | hot Flow                                                        
| --------------------|:---------------------------------------:|
| It emits data only when there is a collector.                  | It emits data even when there is no collector.
| It does not store data.	                                     | It can store data.
| It can't have multiple collectors.	 | It can have multiple collectors.

In Cold Flow, in the case of multiple collectors, the complete flow will begin from the beginning for each one of the collectors, do the task and emit the values to their corresponding collectors. It's like 1-to-1 mapping. 1 Flow for 1 Collector. It means a cold flow can't have multiple collectors as it will create a new flow for each of the collectors.

In Hot Flow, in the case of multiple collectors, the flow will keep on emitting the values, collectors get the values from where they have started collecting. It's like 1-to-N mapping. 1 Flow for N Collectors. It means a hot flow can have multiple collectors.

Let's understand all of those above points from the example code.

Note: I have just written the pseudo-code, this is not actual code. This is just written for the sake of understanding this topic in the simplest way.

### Cold Flow example

Suppose, we have a Cold Flow that emits 1 to 5 at an interval of 1 second.

```
fun getNumbersColdFlow(): ColdFlow<Int> {
    return someColdflow {
        (1..5).forEach {
            delay(1000)
            emit(it)
        }
    }
}
```
Now, we are collecting:

```
val numbersColdFlow = getNumbersColdFlow()

numbersColdFlow
    .collect {
        println("1st Collector: $it")
    }

delay(2500)

numbersColdFlow
    .collect {
        println("2nd Collector: $it")
    }
```
The output will be:

```
1st Collector: 1
1st Collector: 2
1st Collector: 3
1st Collector: 4
1st Collector: 5

2nd Collector: 1
2nd Collector: 2
2nd Collector: 3
2nd Collector: 4
2nd Collector: 5

```
Both the collector will get all the values from the beginning. For both collectors, the corresponding Flow starts from the beginning.

### Hot Flow example

Suppose, we have a Hot Flow that emits 1 to 5 at an interval of 1 second.

```
fun getNumbersHotFlow(): HotFlow<Int> {
    return someHotflow {
        (1..5).forEach {
            delay(1000)
            emit(it)
        }
    }
}
```
Now, we are collecting:

```
val numbersHotFlow = getNumbersHotFlow()

numbersHotFlow
    .collect {
        println("1st Collector: $it")
    }

delay(2500)

numbersHotFlow
    .collect {
        println("2nd Collector: $it")
    }
```
The output will be:

```
1st Collector: 1
1st Collector: 2
1st Collector: 3
1st Collector: 4
1st Collector: 5

2nd Collector: 3
2nd Collector: 4
2nd Collector: 5
```
The collectors will get the values from where they have started collecting. Here the 1st collector gets all the values. But the 2nd collector gets only those values that got emitted after 2500 milliseconds as it started collecting after 2500 milliseconds.

Also, we can configure the Hot Flow to store the data. For example, we can configure it to store the last emitted value.

For example, we configured the above example to store only one last emitted value.

```
fun getNumbersHotFlow(): HotFlow<Int> {
    return someHotflow {
        (1..5).forEach {
            delay(1000)
            emit(it)
        }
    }.store(count = 1)
}
```
Now, if we collect:

```
val numbersHotFlow = getNumbersHotFlow()

numbersHotFlow
    .collect {
        println("1st Collector: $it")
    }

delay(2500)

numbersHotFlow
    .collect {
        println("2nd Collector: $it")
    }
```
The output will be:

```
1st Collector: 1
1st Collector: 2
1st Collector: 3
1st Collector: 4
1st Collector: 5

2nd Collector: 2
2nd Collector: 3
2nd Collector: 4
2nd Collector: 5
```
The collectors will get an extra value in addition to the values from where they have started collecting. Here the 1st collector gets all the values. But the 2nd collector will also get "2"(as it stores the last emitted value) in addition to those values that got emitted after 2500 milliseconds even though it started collecting after 2500 milliseconds.

# State Flow & Shared Flow

| StateFlow              | SharedFlow                                                                                                                        
| --------------------|:---------------------------------------:|
| Hot Flow	                                          | Hot Flow
| Needs an initial value and emits it as soon as the collector starts collecting.| Does not need an initial value so does not emit any value by default.
| val stateFlow = MutableStateFlow(0) | val sharedFlow = MutableSharedFlow<Int>() .
| Only emits the last known value.	  | Can be configured to emit many previous values using the replay operator. 
| It has the value property, we can check the current value. It keeps a history of one value that we can get directly without collecting.	 | It does not have the value property.
| It does not emit consecutive repeated values. It emits the value only when it is distinct from the previous item.	 | It emits all the values and does not care about the distinct from the previous item. It emits consecutive repeated values also. 
| Similar to LiveData except for the Lifecycle awareness of the Android component. We should use repeatOnLifecycle scope with StateFlow to add the Lifecycle awareness to it, then it will become exactly like LiveData. | Not similar to LiveData.
    
    
StateFlow	SharedFlow

Let's understand all of the above points from the example code.

### StateFlow example

Suppose we have StateFlow as below:
```
val stateFlow = MutableStateFlow(0)
```
And, we start collecting on it:
```
stateFlow.collect {
    println(it)
}
```
As soon as we start collecting, we will get:
```
0
```
    
We get this "0" as it takes an initial value and emits it immediately.

And then, if we keep on setting the values as below:

```
stateFlow.value = 1
stateFlow.value = 2
stateFlow.value = 2
stateFlow.value = 1
stateFlow.value = 3
``` 
    
we will get the following output:
```
1
2
1
3
```
Notice here that we are getting "2" only once, not twice. As it does not emit consecutive repeated values.

Suppose we add a new collector now:
```
stateFlow.collect {
    println(it)
}
```
We will get:
```
3
```
As the StateFlow stores the last value and emits it as soon as a new collector starts collecting.

### SharedFlow example

Suppose we have SharedFlow as below:
```
val sharedFlow = MutableSharedFlow<Int>()
```
And, we start collecting on it:
```
sharedFlow.collect {
    println(it)
}
```
As soon as we start collecting, we will not get anything as it does not take an initial value.

And then, if we keep on emitting the values as below:
```
sharedFlow.emit(1)
sharedFlow.emit(2)
sharedFlow.emit(2)
sharedFlow.emit(1)
sharedFlow.emit(3)
```
we will get the following output:
```
1
2
2
1
3
```
Notice here that we are getting "2" twice. As it emits consecutive repeated values also.

Suppose we add a new collector now:
```
sharedFlow.collect {
    println(it)
}
```
We will not get anything as the SharedFlow does not store the last value.

Now, that we have seen the examples of both of them. We can understand the below points.

StateFlow is a type of SharedFlow. StateFlow is a specialization of SharedFlow.

StateFlow is a SharedFlow with a fixed replay = 1 with some more additions. That means new collectors will immediately get the current state as soon as they start collecting.

In a simple way, we can say using the pseudo-code:
```
StateFlow = SharedFlow
            .withInitialValue(initialValue)
            .replay(count=1)
            .distinctUntilChanged()
```
In fact, we can get the StateFlow behavior using SharedFlow as below:
```
val sharedFlow = MutableSharedFlow<Int>(
    replay = 1,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)
sharedFlow.emit(0) // initial value
val stateFlow = sharedFlow.distinctUntilChanged()
```
This is how we can get the StateFlow behavior using SharedFlow.

We can customize the SharedFlow as per our requirement like we can do replay = 2 if we want based on our use case.

Now, it's time to learn where we can use StateFlow and ShareFlow in our Android Project.

Assume that we have a use case: Get the list of the users from the network and show them in the UI.

We have a StateFlow in our ViewModel.
```
val usersStateFlow = MutableStateFlow<UiState<List<User>>>(UiState.Loading)
```
And we have a collector in our Activity.
```
usersStateFlow.collect {

}
```
Now, as soon as we open the activity, the Activity will subscribe to collect. The following will be collected:

usersStateFlow: loading state as StateFlow takes the initial value and emits it immediately.

Now, when our viewModel fetches the data from the network. It will set the data to the usersStateFlow.
```
usersStateFlow.value = UiState.Success(usersFromNetwork)
```
Our Activity collector will get the data of the users and show them in the UI.

Now, if orientation changes, the ViewModel gets retained, and our collector present in the Activity will resubscribe to collect. The following will be collected:

usersStateFlow: List of users which was set from the network. (StateFlow keeps the last value).

### Advantage: No need for a new network call.

Now, let's try to use the SharedFlow in place of the StateFlow.

We have a SharedFlow in our ViewModel.
```
val usersSharedFlow = MutableSharedFlow<UiState<List<User>>>()
```
And we have a collector in our Activity.
```
usersSharedFlow.collect {

}
```
Now, as soon as we open the activity, the Activity will subscribe to collect. Nothing will get collected here as SharedFlow is used.

Now, when our viewModel fetches the data from the network. It will set the data to the usersSharedFlow.
```
usersSharedFlow.emit(UiState.Success(usersFromNetwork))
```
Our Activity collector will get the data of the users and show them in the UI.

Now, if orientation changes, the ViewModel gets retained, and our collector present in the Activity will resubscribe to collect. Nothing will get collected here as SharedFlow is used which does not store any data. We will have to make a new network call.

### Disadvantage: Unnecessary network call as we were already having the data.

So, in this case, we should use StateFlow instead of SharedFlow. However, we can modify SharedFlow to store data as we have seen above about getting the StateFlow behavior using the SharedFlow.

Now, we will take another example to learn where to use SharedFlow instead of StateFlow.

Suppose, suppose we are doing a task, if that task gets failed, we have to show Snackbar.

We have a SharedFlow in our ViewModel.
```
val showSnackbarSharedFlow = MutableSharedFlow<Boolean>()
```
And we have a collector in our Activity.
```
showSnackbarSharedFlow.collect {

}
```
Now, as soon as we open the activity, the Activity will subscribe to collect. Nothing will get collected here as SharedFlow is used.

Then, when our viewModel starts the task and gets failed. It will set the value true to showSnackbarSharedFlow.
```
showSnackbarSharedFlow.emit(true)
```
Our Activity collector will get the value as true and show the Snackbar.

Now, if orientation changes, the ViewModel gets retained, and our collector present in the Activity will resubscribe to collect. Nothing will get collected here as SharedFlow does not keep the last value. And that is fine. We should not show the Snackbar again on orientation changes.

### Advantage: It does not show Snackbar again as intended.

Now, let's try to use the StateFlow in place of the SharedFlow.

We have a StateFlow in our ViewModel.
```
val showSnackbarStateFlow = MutableStateFlow(false)
```
And we have a collector in our Activity.
```
showSnackbarStateFlow.collect {

}
```
Now, as soon as we open the activity, the Activity will subscribe to collect. The following will get collected.

showSnackbarStateFlow: false as StateFlow takes the initial value and emits it immediately.

Now, when our viewModel starts the task and gets failed. It will set the value true to showSnackbarSharedFlow.
```
showSnackbarStateFlow.value = true
```
Our Activity collector will get the value as true and show the Snackbar.

Now, if orientation changes, the ViewModel gets retained, and our collector present in the Activity will resubscribe to collect. The following will be collected:

showSnackbarStateFlow: true will get collected here as StateFlow keeps the last value. It will show the Snackbar again. And that is not fine. We should not show the Snackbar again on orientation changes.

Disadvantage: It shows Snackbar again which is not required.

So, in this case, we should use SharedFlow instead of StateFlow.

# Terminal Operators
    
- Terminal operators are the operators that actually start the flow by connecting the flow builder, operators with the collector.

For example:
```
(1..5).asFlow()
.filter {
    it % 2 == 0
}
.map {
    it * it
}.collect {
    Log.d(TAG, it.toString())
}
```
#### Here, collect is the Terminal Operator.

So, the most basic terminal operator is the collect operator which is the Collector.

So, if you just write the following, the flow will not start:
    
```
(1..5).asFlow()
.filter {
    it % 2 == 0
}
.map {
    it * it
}
```
You must use the terminal operator to start it, in this case, collect.

Let's see another Terminal Operator which is reduce Operator.

- reduce: apply a function to each item emitted and emit the final value
```
 val result = (1..5).asFlow()
    .reduce { a, b -> a + b }

Log.d(TAG, result.toString())
```
Here, the result will be 15.
### Explanation:

At the initial stage, we have 1, 2, 3, 4, 5 which are going to be emitted.

Initially a = 0 and b = 1 which will keep on changing based on the steps.
    
## Step 1:
```
a = 0, b = 1
a = a + b = 0 + 1 = 1
```
## Step 2:
```
a = 1, b = 2
a = a + b = 1 + 2 = 3
```
## Step 3:
    
```
a = 3, b = 3
a = a + b = 3 + 3 = 6
```
## Step 4:
```
a = 6, b = 4
a = a + b = 6 + 4 = 10

```
## Step 5:
    
```
a = 10, b = 5
a = a + b = 10 + 5 = 15
```
This is how the result will be 15.

# Retry / Retrywhen Operator
When we talk about retrying a task using operators in Kotlin Flow, we talk about the following two operators:

## retryWhen
## retry
Both operators can be used interchangeably in most cases, we will learn about them today.

# retryWhen
- Let's look at the source code to understand the definition of the retryWhen operator.
```
fun <T> Flow<T>.retryWhen(predicate: suspend FlowCollector<T>.(cause: Throwable, attempt: Long) -> Boolean): Flow<T>
```
And, we use this operator as below:
```
.retryWhen { cause, attempt ->

}
```
Here, we have two parameters as follows:

cause: This cause is Throwable which is the base class for all errors and exceptions.
attempt: This attempt is the number that represents the current attempt. It starts with zero.
For example, if there is an exception when we started the task, we will receive the cause(exception) and attempt(0).

-  The retryWhen takes a predicate function to decide whether to retry or not.

- If the predicate function returns true, then only it will retry else it will not.

For example, we can do as below:
```
.retryWhen { cause, attempt ->
    if (cause is IOException && attempt < 3) {
        delay(2000)
        return@retryWhen true
    } else {
        return@retryWhen false
    }
}
```
In this case, we are returning true when the cause is IOException, and the attempt count is less than 3.

So, it will only retry if the condition is satisfied.

Note: As the predicate function is suspending function, we can call another suspending function from it.

If we notice in the above code, we have called the delay(2000), so that it retries only after a delay of 2 seconds.

retry
This is the definition of the retry Flow operator.
```
fun <T> Flow<T>.retry(
    retries: Long = Long.MAX_VALUE,
    predicate: suspend (cause: Throwable) -> Boolean = { true }
): Flow<T>
```
The complete block from the source code of Kotlin Flow.
```
fun <T> Flow<T>.retry(
    retries: Long = Long.MAX_VALUE,
    predicate: suspend (cause: Throwable) -> Boolean = { true }
): Flow<T> {
    require(retries > 0) { "Expected positive amount of retries, but had $retries" }
    return retryWhen { cause, attempt -> attempt < retries && predicate(cause) }
}
```
- If we see the retry function, it actually calls the retryWhen internally.
- retry function has default arguments.

If we do not pass the retries, it will use the Long.MAX_VALUE.
If we do not pass the predicate, it will provide true.
For example, we can do as below:
```
.retry()
```                                                      
It will keep retrying until the task gets completed successfully.

For example, we can also do as below:
```
.retry(3)
 ```                                                         
It will only retry 3 times.

For example, we can also do as below:
```
.retry(retries = 3) { cause ->
    if (cause is IOException) {
        delay(2000)
        return@retry true
    } else {
        return@retry false
    }
}
```
Here, it becomes very similar to what we did using the retryWhen above.

Here, we are returning true when the cause is IOException. So, it will only retry when the cause is IOException.

If we notice in the above code, we have called the delay(2000), so that it retries only after a delay of 2 seconds.

- Now, let's see the code examples.

- This is a function to simulate a long-running task with exceptions.
```
private fun doLongRunningTask(): Flow<Int> {
    return flow {
        // your code for doing a long running task
        // Added delay, random number, and exception to simulate
        delay(2000)
        val randomNumber = (0..2).random()
        if (randomNumber == 0) {
            throw IOException()
        } else if (randomNumber == 1) {
            throw IndexOutOfBoundsException()
        }
        delay(2000)
        emit(0)
    }
}
```
- Now, when using the retry operator
```
viewModelScope.launch {
    doLongRunningTask()
        .flowOn(Dispatchers.Default)
        .retry(retries = 3) { cause ->
            if (cause is IOException) {
                delay(2000)
                return@retry true
            } else {
                return@retry false
            }
        }
        .catch {
           // error
        }
        .collect {
            // success
        }
}
```
Similarly, when using the retryWhen operator
```
viewModelScope.launch {
    doLongRunningTask()
        .flowOn(Dispatchers.Default)
        .retryWhen { cause, attempt ->
            if (cause is IOException && attempt < 3) {
                delay(2000)
                return@retryWhen true
            } else {
                return@retryWhen false
            }
        }
        .catch {
            // error
        }
        .collect {
            // success
        }
}
```
- If we see, every time we are adding the delay of 2 seconds, but in real use-cases, we add delay with exponential backoff. Do not worry, we will implement that too.
    

