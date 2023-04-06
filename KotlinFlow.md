

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



