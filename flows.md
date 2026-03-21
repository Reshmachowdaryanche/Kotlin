
## 1. What is Flow?

A stream of data that can be computed asynchronously is referred to as a Flow . It allows you to emit multiple values over time in a sequential and reactive manner. Some key characteristics of Flow:

- Flow is designed to handle asynchronous data streams, where values are emitted one after the other. Each emission is processed sequentially, suspending until the previous emission completes, providing a natural way to handle data flow in a non-blocking way.

- Flow handles backpressure automatically by suspending emissions if the collector (consumer) is slow to process them. This prevents overwhelming the consumer and manages resource usage effectively.

- Flow is "cold," meaning it doesn’t produce or emit any values until it is actively collected. Each time you call collect on a Flow, it starts from scratch, similar to how a function is called and executed anew. This is different from hot streams, like LiveData or RxJava’s Subject, which emit values independently of whether there’s an active observer.

---

## 2. What are the different ways to create a Flow?

Flow builders allow you to create flows in various ways depending on your use case. The most commonly used flow builders include:

The flow builder is the primary way to create a flow. It allows you to emit values asynchronously using the emit() function.

```kotlin
fun simpleFlow(): Flow<Int> = flow {
    for (i in 1..5) {
        delay(100) // Simulate some delay
        emit(i) // Emit values from 1 to 5
    }
}
```

The flowOf builder creates a flow from a fixed set of values.

```kotlin
val values = flowOf(1, 2, 3, 4, 5)
```

The asFlow extension allows you to convert collections or sequences into flows.

```kotlin
val values = listOf(1, 2, 3).asFlow()
```

---

## 3. What are the two different types of Flows?

There are two different types of Flows:

- A Cold Flow in Kotlin is a flow that does not start emitting values until a collector actively starts collecting it. This means that each collector (or subscriber) gets its own instance of the flow, and the flow starts from scratch every time it is collected.
- Cold flows are “lazy,” so no work is done until there is a demand for data.
- Each collector receives its own independent stream of data. Each time a new collector subscribes, the flow starts from the beginning.
Suitable for use cases where you want fresh data for each subscriber, such as database queries, network requests, or other repeatable sources.

``` // Regular Flow example
    val coldFlow = flow {
         emit(0) 
         emit(1)
         emit(2)
   }

    launch { // Calling collect the first time
        coldFlow.collect { value ->
            println("cold flow collector 1 received: $value")
        }

        delay(2500)
       
      // Calling collect a second time
      coldFlow.collect { value ->
            println("cold flow collector 2 received: $value")
        }
    }


// RESULT
// Both the collectors will get all the values from the beginning. 
// For both collectors, the corresponding Flow starts from the beginning.
flow collector 1 received: [0, 1, 2]
flow collector 1 received: [0, 1, 2]
```

- Hot Flow emit values independently of whether there are active collectors or not. Once started, a hot flow continuously produces data that is shared among all collectors. This behavior is similar to broadcasting: new subscribers (collectors) receive only the latest emissions but miss any past values emitted before they started collecting.
- All collectors receive data from the same ongoing flow, starting from the latest value at the time they subscribe.
- Emission does not restart for each new collector; it’s a single, shared source.
- Suitable for scenarios like UI state updates, event broadcasting, or shared state where all subscribers need access to the latest values.
- 
```// SharedFlow example
    val sharedFlow = MutableSharedFlow<Int>()

    sharedFlow.emit(0)
    sharedFlow.emit(1)
    sharedFlow.emit(2)
    sharedFlow.emit(3)
    sharedFlow.emit(4)

    launch {
        sharedFlow.collect { value ->
            println("SharedFlow collector 1 received: $value")
        }

        delay(2500)
       
      // Calling collect a second time
      sharedFlow.collect { value ->
            println("SharedFlow collector 2 received: $value")
        }
    }

// RESULT 
// The collectors will get the values from where they have started collecting. 
// Here the 1st collector gets all the values. But the 2nd collector gets 
// only those values that got emitted after 2500 milliseconds as it started 
// collecting after 2500 milliseconds.
SharedFlow collector 1 received: [0,1,2,3,4]
SharedFlow collector 2 received: [2,3,4]
```
---

## 4. SharedFlow vs StateFlow?

Both SharedFlow and StateFlow are types of hot flows that emit values to multiple subscribers and keep emitting even when no subscribers are actively collecting.

- StateFlow is a specialized hot flow designed to hold and emit state updates. It always has a single current value and emits the latest state to new collectors.
- It holds a single, current value that can be read and updated directly. Changes to the value are updated immediately, and new collectors always receive the latest value upon subscription.
- Only the latest value is replayed to new collectors.
- Exposed as an immutable StateFlow, so external subscribers can read but not modify the value.
- Commonly used in ViewModels to hold the UI state and expose it to the UI layer, such as with Android’s Jetpack Compose or LiveData replacements. Ideal for cases where a single source of truth (the current state) needs to be shared with multiple consumers, ensuring all consumers always have the most recent data.

```class HomeViewModel : ViewModel() {

    private val _textStateFlow = MutableStateFlow("Hello World")
    val stateFlow =_textStateFlow.asStateFlow()

    fun triggerStateFlow(){
        _textStateFlow.value="State flow"
    }
}

// Collecting StateFlow in Activity/Fragment
class HomeFragment : Fragment() {
    private val viewModel: HomeViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

lifecycleScope.launchWhenStarted {

  // Triggers the flow and starts listening for values

  // collectLatest() is a higher-order function in Kotlin's Flow API 
  // that allows you to collect emitted values from a Flow and perform 
  // a transformation on the most recent value only. It is similar to 
  // collect(), which is used to collect all emitted values, 
  // but collectLatest only processes the latest value emitted and 
  // ignores any previous values that have not yet been processed.
    viewModel.stateFlow.collectLatest {
          binding.stateFlowButton.text = it
    }
  }
}

// Collecting StateFlow in Compose
@Compose
fun HomeScreen() {
 // Compose provides the collectAsStateWithLifecycle function, which 
 // collects values from a flow and gives the latest value to be used 
 // wherever needed. When a new flow value is emitted, we get the updated 
 // value, and re-composition takes place to update the state of the value.
 // It uses LifeCycle.State.Started by default to start collecting values 
 // when the lifecycle is in the specified state and stops when it falls 
 // below it.
  val someFlow by viewModel.flow.collectAsStateWithLifecycle()
  
}
```

- SharedFlow is a general-purpose hot flow that can emit events or shared data to multiple subscribers.
- Unlike StateFlow, SharedFlow is highly configurable, allowing you to control the number of past emissions that new subscribers will receive (replay) and set a buffer to handle emissions when there are no active collectors.
- SharedFlow does not hold a single current value. Instead, it broadcasts emissions to all subscribers.
- Allows you to define a buffer for values, which can prevent emissions from being lost if there are no active collectors or if collectors are slow.
- Best for representing events or streams of data that do not represent a continuous state (such as notifications, one-time actions, or events that multiple subscribers might need).

```class HomeViewModel : ViewModel() {
    private val _events = MutableSharedFlow<Event>() // private mutable shared flow
    val events = _events.asSharedFlow() // publicly exposed as read-only shared flow

    suspend fun produceEvent(event: Event) {
        _events.emit(event) // suspends until all subscribers receive it
    }
}

// Collecting StateFlow in Activity/Fragment
class HomeFragment : Fragment() {
    private val viewModel: HomeViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

lifecycleScope.launchWhenStarted {

  // Triggers the flow and starts listening for values

  // collectLatest() is a higher-order function in Kotlin's Flow API 
  // that allows you to collect emitted values from a Flow and perform 
  // a transformation on the most recent value only. It is similar to 
  // collect(), which is used to collect all emitted values, 
  // but collectLatest only processes the latest value emitted and 
  // ignores any previous values that have not yet been processed.
    viewModel.events.collectLatest {
          binding.eventFlowButton.text = it
    }
  }
}

// Collecting StateFlow in Compose
@Compose
fun HomeScreen() {
 // Compose provides the collectAsStateWithLifecycle function, which 
 // collects values from a flow and gives the latest value to be used 
 // wherever needed. When a new flow value is emitted, we get the updated 
 // value, and re-composition takes place to update the state of the value.
 // It uses LifeCycle.State.Started by default to start collecting values 
 // when the lifecycle is in the specified state and stops when it falls 
 // below it.
  val someFlow by viewModel.events.collectAsStateWithLifecycle()
  
}
```

### Replay vs Buffer in SharedFlow

---

### 🧠 Core Difference (One Line)

```text
Replay → for NEW subscribers (past values)

Buffer → for CURRENT flow (handling slow/no collectors)
```

---

### 🔁 1️⃣ Replay (Past Values)

👉 Replay stores previous emissions and gives them to **new collectors**

```kotlin
val flow = MutableSharedFlow<Int>(replay = 2)
```

---

### ✅ What happens:

* Emit: `1, 2, 3`
* New collector joins → gets: `2, 3`

✔️ It **replays past values**

---

### 🎯 Purpose:

* Help **late subscribers**
* Useful when new observers should see **recent history**

---

### 🧺 2️⃣ Buffer (`extraBufferCapacity`)

👉 Buffer stores values when:

* No collectors
* Collectors are slow

```kotlin
val flow = MutableSharedFlow<Int>(
    replay = 0,
    extraBufferCapacity = 2
)
```

---

### ✅ What happens:

* Emit values even if no one is collecting
* Values are temporarily stored

✔️ Prevents **data loss during emission**

---

### 🎯 Purpose:

* Handle **backpressure**
* Avoid losing values when system is busy

---

### 🔥 Final Mental Model

```text
Replay → store past for future subscribers

Buffer → store present for slow consumers
```

---

### ⚡ Quick Comparison

| Feature  | Replay           | Buffer                |
| -------- | ---------------- | --------------------- |
| For      | New subscribers  | Current collectors    |
| Stores   | Past values      | Temporary values      |
| Use case | Late subscribers | Backpressure handling |

---

### 🧠 Summary

* `replay` → for **new collectors**
* `buffer` → for **current flow handling**
* Both help in **controlling data flow efficiently**

---

## 5. What are Terminal operators in Flow?

Terminal operators are operators that collect the values emitted by a flow and perform a final action on them. Terminal operators are responsible for starting the flow’s collection process, meaning that until a terminal operator is invoked, the flow remains cold and does not produce any values. Different types of terminal operators:

collect: is used to receive each emitted value from the flow and perform a specified action on it.

```kotlin
flowOf(1, 2, 3).collect { value ->
    println("Received: $value")
}
```

toList collects all emitted values and stores them in a List, returning the list when the flow completes.

```kotlin
val resultList = flowOf(1, 2, 3).toList()
println(resultList)
```

toSet collects all emitted values into a Set.

```kotlin
val resultSet = flowOf(1, 2, 2, 3).toSet()
println(resultSet)
```

first returns the first value emitted.

```kotlin
val firstValue = flowOf(1, 2, 3).first()
println(firstValue)
```

last returns the last emitted value.

```kotlin
val lastValue = flowOf(1, 2, 3).last()
println(lastValue)
```

single expects exactly one value.

```kotlin
val singleValue = flowOf(42).single()
println(singleValue)
```

reduce accumulates values.

```kotlin
val sum = flowOf(1, 2, 3, 4).reduce { accumulator, value ->
    accumulator + value
}
println(sum)
```

fold with initial value.

```kotlin
val sumWithInitial = flowOf(1, 2, 3, 4).fold(10) { accumulator, value ->
    accumulator + value
}
println(sumWithInitial)
```

count items.

```kotlin
val count = flowOf(1, 2, 3, 4).count { it % 2 == 0 }
println(count)
```

---

## 6. What does the launchIn keyword do?

```kotlin
val scope = CoroutineScope(Dispatchers.Default)
flowOf(1, 2, 3)
    .onEach { println("Received: $it") }
    .launchIn(scope)
```

---

## 7. What is the difference between StateIn and ShareIn?

The stateIn operator converts a cold Flow into a StateFlow.

The shareIn operator converts a cold Flow into a SharedFlow.

---

## 8. How can we collect Flows in Jetpack Compose?

Using collectAsState
Using collectAsStateWithLifecycle
Using LaunchedEffect with collect

---

## 9. How can we handle backpressure when using flows?

buffer
conflate
zip and combine

---

## 10. How can we cancel a flow?

Cancel the coroutine collecting the flow

---

## 11. What does the flowOn keyword do?

Changes coroutine context for upstream operations

---

## 12. How can we combine multiple flows?

zip
combine
flattenMerge
merge
flatMapConcat
flatMapMerge
flatMapLatest

---

## 13. What are the different ways to handle exception in flows?

catch
onCompletion
emitCatching

---

## 14. How does the retry operator work with Flow?

Retries flow on exception

---

## 15. How do you implement a debounce mechanism for user input using flows?

---

## 16. Difference between LiveData & Flows.

LiveData is a Hot stream.
Flow is a Cold stream.

LiveData is Lifecycle-aware.
Flow is not lifecycle-aware.

LiveData → UI layer
Flow → Anywhere

Flow supports error handling, threading, backpressure

---

## 17. Difference between Flows & Channels?

Flow = cold stream
Channel = hot stream

Flow = transformations
Channel = communication

---

## 18. Example of unit testing with flows

UserViewModel example
Repository exposes Flow
Use Turbine for testing

---

Thanks for reading!
Hope you find this useful.
