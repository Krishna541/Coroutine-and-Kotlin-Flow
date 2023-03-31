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
