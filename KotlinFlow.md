

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
