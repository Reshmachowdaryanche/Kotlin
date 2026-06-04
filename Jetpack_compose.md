# What is jetpack compose

> **Jetpack Compose is Android's modern declarative UI toolkit developed by Google for building native Android applications using Kotlin.**
>
> Unlike the traditional View system where we create XML layouts and manually update views using methods like `findViewById()` or `setText()`, Compose allows us to describe the UI as a function of state using composable functions.
>
> When the application's state changes, Compose automatically updates only the affected parts of the UI through a process called recomposition. This reduces boilerplate code, improves readability, and makes UI development faster and easier to maintain.
>
> It also integrates well with modern Android architecture components such as ViewModel, StateFlow, and Navigation, making it the recommended approach for Android UI development today.

# jetpack Compose vs Android View System.

> **Jetpack Compose and the Android View System are two different approaches to building UI in Android.**
>
> The traditional **View System** is based on an **imperative approach**, where we define UI using XML layouts and then manually update views at runtime using methods like `findViewById()`, `setText()`, or `addView()`. In this approach, developers are responsible for keeping the UI in sync with the data, which often leads to more boilerplate code and higher chances of inconsistencies.
>
> In contrast, **Jetpack Compose** follows a **declarative approach**, where we describe the UI using Kotlin functions called composables. Instead of manually updating UI elements, we provide state to composables, and whenever the state changes, Compose automatically updates the UI through recomposition.
>
> So, in short, the View system is **UI-driven and manually updated**, while Compose is **state-driven and automatically updates the UI based on state changes**.
>
> Compose is more modern, reduces boilerplate, and integrates better with Kotlin and modern architecture patterns like ViewModel and StateFlow.

# Explain the concept of declarative UI in Jetpack Compose.

> Declarative UI in Jetpack Compose means that instead of building and updating UI widgets manually, we describe the UI as a function of state.

> In traditional imperative UI toolkits, we create a tree of widgets (often using XML), and each widget maintains its own internal state with getter and setter methods. We directly manipulate these widgets to update the UI.

> However, in Compose, UI components (composables) are stateless and do not expose getters or setters. They are not treated as objects to be modified. Instead, we simply call composable functions with the current state as input, and they return a UI representation.

> The app logic (often through a ViewModel) provides state to top-level composables. These composables then pass the state down the hierarchy and describe the UI at each level.

> When the user interacts with the UI, events like `onClick` are triggered and sent back to the app logic. The app logic updates the state, and when that state changes, Compose automatically re-executes the composable functions. This process is called recomposition, and it updates only the necessary parts of the UI.

> In summary, Declarative UI means the UI is rebuilt from state whenever it changes, and we focus on describing what the UI should look like rather than how to update it manually.


# Declarative UI vs Imperative UI

> **Declarative UI and Imperative UI are two different approaches to building user interfaces.**
>
> In an **imperative UI approach**, developers focus on *how to update the UI*. We manually create UI components and explicitly modify them whenever data changes.
>
> For example, in the traditional Android View system, we define layouts using XML and update views using methods like `findViewById()`, `setText()`, `setVisibility()`, or `addView()`. Here, the developer is responsible for keeping the UI synchronized with the data.
>
> In contrast, **declarative UI focuses on what the UI should look like for a given state**. Instead of manually updating views, we describe the UI as a function of state.
>
> Jetpack Compose follows this declarative approach. We pass state to composable functions, and whenever the state changes, Compose automatically updates the required parts of the UI through recomposition.
>
> So, rather than writing code to manipulate views step by step, we simply describe the desired UI for the current state, and Compose handles the UI updates internally.
>
> In summary:
>
> * **Imperative UI → Focuses on how to update UI manually**
> * **Declarative UI → Focuses on describing UI based on state**
>
> Declarative UI reduces boilerplate code, improves readability, and makes UI management simpler and more predictable.

# What are Composable Functions?

> **Composable functions are the basic building blocks of UI in Jetpack Compose.**
>
> They are Kotlin functions annotated with the `@Composable` annotation and are used to describe how the UI should look for a given state.
>
> Instead of creating XML layouts and manually manipulating View objects, Compose allows us to build UI by calling composable functions directly from Kotlin code.
>
> Example:
>
> ```kotlin
> @Composable
> fun Greeting(name: String) {
>     Text(text = "Hello $name")
> }
> ```
>
> In this example:
>
> * `Greeting()` is a composable function.
> * It accepts data as input through the `name` parameter.
> * It emits UI by calling another composable function `Text()`.
>
> A few important characteristics of composable functions are:
>
> ### 1. Annotated with `@Composable`
>
> Every composable function must use the `@Composable` annotation. This tells the Compose compiler that the function is intended to convert state/data into UI.
>
> ---
>
> ### 2. Accept Data as Input
>
> Composable functions can take parameters. This allows the app logic or ViewModel to pass state into the UI.
>
> Example:
>
> ```kotlin
> Greeting(name = "Reshma")
> ```
>
> ---
>
> ### 3. Emit UI Instead of Returning UI Objects
>
> Composable functions do not return Views or UI objects. Instead, they describe the UI by calling other composable functions.
>
> Example:
>
> ```kotlin
> Text("Hello")
> ```
>
> Compose internally manages the UI hierarchy.
>
> ---
>
> ### 4. Can Be Re-Executed Multiple Times
>
> Composable functions may run many times during recomposition whenever observed state changes.
>
> Because of this, composables should ideally be:
>
> * Fast
> * Idempotent
> * Side-effect free
>
> Meaning:
>
> * Calling the composable multiple times with the same input should produce the same UI.
> * They should not modify global variables or perform operations like network calls directly.
>
> ---
>
> ### 5. Build UI Hierarchies
>
> Composable functions can call other composable functions, which helps create reusable and modular UI components.
>
> Example:
>
> ```kotlin
> @Composable
> fun ProfileScreen() {
>     Column {
>         ProfileImage()
>         UserName()
>         UserBio()
>     }
> }
> ```
>
> In summary, composable functions are declarative Kotlin functions that take state as input and emit UI. They are lightweight, reusable, automatically recomposed when state changes, and form the foundation of Jetpack Compose UI development.

# What is Recomposition?

> **Recomposition is the process where Jetpack Compose re-executes composable functions when their input state changes in order to update the UI.**
>
> In traditional imperative UI systems, when data changes, developers manually update the affected widgets using setters like `setText()`, `setVisibility()`, or `notifyDataSetChanged()`.
>
> However, in Jetpack Compose, UI is declarative and state-driven. Instead of manually modifying widgets, we call composable functions again with the updated state. Compose then intelligently updates only the necessary parts of the UI.
>
> This process of re-executing composable functions with new state is called recomposition. 
>
> ---
>
> # Example of Recomposition
>
> ```kotlin
> @Composable
> fun ClickCounter(clicks: Int, onClick: () -> Unit) {
>     Button(onClick = onClick) {
>         Text("I've been clicked $clicks times")
>     }
> }
> ```
>
> In this example:
>
> * `clicks` is the state.
> * Whenever the button is clicked, the value of `clicks` changes.
> * Compose calls the composable again with the updated value.
> * The `Text()` composable gets recomposed and displays the latest count.
>
> So the flow becomes:
>
> ```text
> State Change
>      ↓
> Recomposition
>      ↓
> UI Updated
> ```
>
> Compose automatically updates the UI without developers manually changing views. 
>
> ---
>
> # Intelligent Recomposition
>
> One of the biggest advantages of Compose is that recomposition is intelligent and optimized.
>
> Rebuilding the entire UI tree every time state changes would be computationally expensive and would consume more CPU and battery.
>
> Compose solves this problem by recomposing only the composables whose inputs have changed and skipping the rest. 
>
> Example:
>
> ```kotlin
> Text(header)
> ```
>
> If only the list changes and `header` remains the same:
>
> * Compose skips recomposing `Text(header)`
> * Only the list-related composables are recomposed
>
> This selective recomposition is one of the core reasons why Compose performs efficiently.
>
> ---
>
> # Recomposition Skips Unchanged UI
>
> Compose tries to skip as much work as possible during recomposition.
>
> Every composable function or lambda can independently recompose.
>
> Example:
>
> ```kotlin
> @Composable
> fun NamePicker(
>     header: String,
>     names: List<String>
> )
> ```
>
> If:
>
> * `header` changes → only header-related composables recompose
> * `names` changes → only list-related composables recompose
>
> Compose avoids recomposing unaffected UI sections. 
>
> This is very important for performance because large UI trees can still update efficiently.
>
> ---
>
> # Recomposition is Optimistic
>
> Recomposition in Compose is optimistic.
>
> This means Compose assumes recomposition will complete successfully.
>
> However, while recomposition is happening, state may change again.
>
> In such cases, Compose may:
>
> * Cancel the current recomposition
> * Discard partially built UI
> * Restart recomposition with the latest state
>
> Example:
>
> ```text
> Recomposition Started
>        ↓
> State Changed Again
>        ↓
> Current Recomposition Cancelled
>        ↓
> New Recomposition Started
> ```
>
> Because recompositions can be canceled midway, composables should never depend on side-effects. 
>
> ---
>
> # Why Composables Should Be Side-Effect Free
>
> A side-effect is any operation that changes state outside the composable.
>
> Dangerous side-effects include:
>
> * Updating global variables
> * Writing to SharedPreferences
> * Updating ViewModel state directly
> * Database writes
> * Network calls
>
> Compose may:
>
> * Recompose frequently
> * Skip recompositions
> * Cancel recompositions
> * Execute composables in different orders
>
> Because of this, side-effects inside composables can create inconsistent or unpredictable behavior. 
>
> ---
>
> # Bad Example of Side-Effects
>
> ```kotlin
> var count = 0
>
> @Composable
> fun Counter() {
>     count++
> }
> ```
>
> Since recomposition may happen multiple times:
>
> ```text
> Counter()
> Counter()
> Counter()
> ```
>
> `count` may increase unexpectedly.
>
> This creates unstable UI behavior.
>
> ---
>
> # Good Compose Practice
>
> Instead of performing side-effects inside composables:
>
> * Move business logic to ViewModel
> * Perform expensive work in coroutines
> * Pass state into composables
> * Use callbacks like `onClick`
>
> Example:
>
> ```kotlin
> @Composable
> fun SharedPrefsToggle(
>     text: String,
>     value: Boolean,
>     onValueChanged: (Boolean) -> Unit
> )
> ```
>
> Here:
>
> * Composable only displays UI
> * ViewModel handles SharedPreferences update
> * UI remains side-effect free
>
> This is the recommended Compose architecture. 
>
> ---
>
> # Composable Functions Can Run Frequently
>
> Composable functions may execute very frequently.
>
> During animations, recomposition can happen every frame.
>
> Example:
>
> ```text
> 60 FPS Animation
>        ↓
> Composable may execute 60 times per second
> ```
>
> Because of this:
>
> * Composables should be lightweight
> * Expensive operations should be avoided
> * Heavy work should happen outside composition
>
> Bad:
>
> ```kotlin
> @Composable
> fun Screen() {
>     val prefs = sharedPreferences.getString(...)
> }
> ```
>
> This may repeatedly read storage during recompositions and cause UI lag or jank. 
>
> ---
>
> # Recomposition and State Observation
>
> Compose automatically tracks state reads.
>
> Example:
>
> ```kotlin
> Text("$count")
> ```
>
> Compose internally remembers:
>
> ```text
> This composable depends on count
> ```
>
> When `count` changes:
>
> ```text
> count updated
>       ↓
> Only affected composables recomposed
> ```
>
> This is how Compose achieves efficient UI updates.
>
> ---
>
> # Composable Functions Can Execute in Parallel
>
> Compose is currently mostly single-threaded, but it is designed with future multithreading support in mind.
>
> In the future, Compose may execute composables in parallel on multiple threads for better performance. 
>
> This means:
>
> * Composables should not modify shared mutable variables
> * They should remain thread-safe
> * They should only transform state into UI
>
> ---
>
> # Bad Parallel Execution Example
>
> ```kotlin
> @Composable
> fun ListWithBug(myList: List<String>) {
>     var items = 0
>
>     Column {
>         for (item in myList) {
>             items++
>         }
>     }
>
>     Text("Count: $items")
> }
> ```
>
> This is dangerous because recomposition may happen multiple times or potentially on different threads in the future.
>
> The UI may display incorrect counts. 
>
> ---
>
> # Composable Functions Can Execute in Any Order
>
> Compose does not guarantee execution order between composable functions.
>
> Example:
>
> ```kotlin
> StartScreen()
> MiddleScreen()
> EndScreen()
> ```
>
> Compose may execute them in any order depending on optimization priorities. 
>
> Therefore:
>
> * One composable should never depend on another composable's side-effects
> * Every composable should be self-contained
>
> ---
>
> # Key Characteristics of Recomposition
>
> Recomposition is:
>
> * Automatic
> * State-driven
> * Optimized
> * Selective
> * Cancelable
> * Frequent
> * Potentially parallel in the future
>
> ---
>
> # Interview One-Liner
>
> > Recomposition is the process where Jetpack Compose re-executes composable functions whenever observed state changes and intelligently updates only the affected parts of the UI instead of redrawing the entire screen.

# What is State in Jetpack Compose?

> **State in Jetpack Compose is any value that can change over time and affect the UI.**
>
> Compose follows a declarative UI model where the UI is generated based on the current state. Whenever the state changes, Compose automatically updates the affected UI through recomposition.
>
> Examples of state include:
>
> * Text entered in a TextField
> * Checkbox selection
> * Loading state
> * Counter value
> * API response data
> * Selected item in a list
>
> ---
>
> ## State Drives UI
>
> In Jetpack Compose:
>
> ```text
> UI = f(State)
> ```
>
> Meaning:
>
> * Same state → Same UI
> * Different state → Different UI
>
> Whenever state changes:
>
> ```text
> State Change
>       ↓
> Recomposition
>       ↓
> UI Updated
> ```
>
> Compose automatically updates the required UI components.
>
> ---
>
> ## What is `remember`?
>
> Composable functions can recompose many times. During recomposition, local variables normally get recreated.
>
> Example:
>
> ```kotlin
> @Composable
> fun Counter() {
>     var count = 0
> }
> ```
>
> Here, during every recomposition:
>
> ```text
> count becomes 0 again
> ```
>
> To retain values across recompositions, Compose provides the `remember` API.
>
> ```kotlin
> val state = remember { ... }
> ```
>
> `remember` stores an object in the Composition memory during the initial composition and returns the stored value during recomposition.
>
> Important point:
>
> > `remember` retains state only as long as the composable remains in the Composition.
>
> If the composable leaves the Composition, the remembered object is forgotten.
>
> ---
>
> ## What is `mutableStateOf`?
>
> `mutableStateOf()` creates an observable `MutableState<T>` object that is integrated with the Compose runtime.
>
> Example:
>
> ```kotlin
> val count = mutableStateOf(0)
> ```
>
> Internally:
>
> ```kotlin
> interface MutableState<T> : State<T> {
>     override var value: T
> }
> ```
>
> Whenever the `value` changes:
>
> ```kotlin
> count.value++
> ```
>
> Compose automatically schedules recomposition for composables reading that state.
>
> ---
>
> ## How Compose Tracks State
>
> Example:
>
> ```kotlin
> Text("$count")
> ```
>
> Compose internally tracks:
>
> ```text
> This composable depends on count
> ```
>
> When `count` changes:
>
> ```text
> State Updated
>       ↓
> Composable Invalidated
>       ↓
> Recomposition Happens
>       ↓
> UI Updated
> ```
>
> This makes Compose highly efficient because only affected composables are recomposed.
>
> ---
>
> ## Different Ways to Declare State in Compose
>
> ### 1. Direct `MutableState` object
>
> ```kotlin
> val mutableState = remember { mutableStateOf(default) }
> ```
>
> Here:
>
> * `mutableState` is a `MutableState<T>` object.
> * To read the value, use:
>
> ```kotlin
> mutableState.value
> ```
>
> * To update the value:
>
> ```kotlin
> mutableState.value = newValue
> ```
>
> #### Example
>
> ```kotlin
> val countState = remember { mutableStateOf(0) }
>
> Text(text = countState.value.toString())
>
> Button(onClick = {
>     countState.value++
> }) {
>     Text("Increment")
> }
> ```
>
> #### What is happening internally?
>
> `mutableStateOf()` creates a state holder object.
>
> Something like:
>
> ```kotlin
> MutableState<Int>
> ```
>
> Compose observes `.value`.
>
> Whenever `.value` changes:
>
> * Compose marks the composable for recomposition.
> * UI updates automatically.
>
> ---
>
> ### 2. Property Delegation (`by`)
>
> ```kotlin
> var value by remember { mutableStateOf(default) }
> ```
>
> This is the **most commonly used Compose style**.
>
> Here:
>
> * `value` is NOT a `MutableState`.
> * `value` directly contains the actual value.
> * Kotlin property delegation (`by`) automatically uses `.value` internally.
>
> So this:
>
> ```kotlin
> var count by remember { mutableStateOf(0) }
> ```
>
> is equivalent to:
>
> ```kotlin
> val countState = remember { mutableStateOf(0) }
>
> var count
>     get() = countState.value
>     set(value) {
>         countState.value = value
>     }
> ```
>
> #### Example
>
> ```kotlin
> var count by remember { mutableStateOf(0) }
>
> Text(text = count.toString())
>
> Button(onClick = {
>     count++
> }) {
>     Text("Increment")
> }
> ```
>
> #### Why preferred?
>
> Cleaner syntax:
>
> * No `.value`
> * Easier to read
> * Less boilerplate
>
> Required imports:
>
> ```kotlin
> import androidx.compose.runtime.getValue
> import androidx.compose.runtime.setValue
> ```
>
> ---
>
> ### 3. Destructuring Declaration
>
> ```kotlin
> val (value, setValue) = remember { mutableStateOf(default) }
> ```
>
> This uses Kotlin destructuring.
>
> `MutableState` provides:
>
> ```kotlin
> component1() -> current value
> component2() -> setter lambda
> ```
>
> So internally:
>
> ```kotlin
> val state = remember { mutableStateOf(default) }
>
> val value = state.value
> val setValue = { newValue ->
>     state.value = newValue
> }
> ```
>
> #### Example
>
> ```kotlin
> val (count, setCount) = remember { mutableStateOf(0) }
>
> Text(text = count.toString())
>
> Button(onClick = {
>     setCount(count + 1)
> }) {
>     Text("Increment")
> }
> ```
>
> ---
>
> ### Comparison Table
>
> | Syntax                                       | Variable Type     | Read          | Update          | Common Usage             |
> | -------------------------------------------- | ----------------- | ------------- | --------------- | ------------------------ |
> | `val state = remember { mutableStateOf() }`  | `MutableState<T>` | `state.value` | `state.value =` | When state object needed |
> | `var value by remember { mutableStateOf() }` | actual value      | `value`       | `value =`       | Most preferred           |
> | `val (value, setValue)`                      | value + setter    | `value`       | `setValue()`    | React-style pattern      |
>
> ---
>
> ### Which one should you use?
>
> ### Most common in Compose
>
> ```kotlin
> var value by remember { mutableStateOf(default) }
> ```
>
> because:
>
> * Cleaner
> * Readable
> * Less boilerplate
> 
> ---
>
### When to use direct `MutableState`
>
> Useful when passing state object itself:
>
> ```kotlin
> fun MyComposable(state: MutableState<Int>)
> ```
>
> or when APIs expect `State<T>`.
>
> ---
>
> ### When destructuring is useful
>
> Less common in Android Compose.
>
> Feels similar to React:
>
> ```javascript
> const [value, setValue] = useState()
> ```
>
> Some developers prefer it for functional style.
>
> ---
>
> ### Important Compose Concept
>
> All 3 ultimately create:
>
> ```kotlin
> MutableState<T>
> ```
>
> and all trigger recomposition when state changes.
>
> The difference is only:
>
> * syntax
> * readability
> * access style
>
> ---
>
> ### Visual Understanding
>
> #### Style 1
>
> ```kotlin
> state.value
> ```
>
> You manually access the box.
>
> ---
>
#### Style 2
?
> ```kotlin
> value
> ```
>
> Kotlin automatically opens the box for you.
>
>---
#### Style 3
> ```kotlin
> value
> setValue()
> ```
>
> Compose gives you:
>
> * current value
> * function to update it
>
> similar to React Hooks.
>
>
> ---
>
> ### Using State in UI
>
> State can control:
>
> * UI values
> * Visibility
> * Conditional rendering
> * Business logic
>
> Example:
>
> ```kotlin
> @Composable
> fun HelloContent() {
>     var name by remember {
>         mutableStateOf("")
>     }
>
>     if (name.isNotEmpty()) {
>         Text("Hello, $name!")
>     }
>
>     OutlinedTextField(
>         value = name,
>         onValueChange = { name = it }
>     )
> }
> ```
>
> Here:
>
> * `name` is state
> * Typing updates state
> * State change triggers recomposition
> * UI updates automatically
>
> ---
>
> ## What is `rememberSaveable`?
>
> `remember` survives recomposition, but it does not survive configuration changes like:
>
> * Screen rotation
> * Process recreation
>
> For this, Compose provides:
>
> ```kotlin
> rememberSaveable
> ```
>
> Example:
>
> ```kotlin
> var text by rememberSaveable {
>     mutableStateOf("")
> }
> ```
>
> `rememberSaveable` automatically saves values that can be stored inside a `Bundle`.
>
> For custom objects, custom savers can be provided.
>
> ---
>
> ## Stateful vs Stateless Composables
>
> ### Stateful Composable
>
> A composable that internally owns and manages state.
>
> Example:
>
> ```kotlin
> @Composable
> fun Counter() {
>     var count by remember {
>         mutableStateOf(0)
>     }
> }
> ```
>
> ---
>
> ### Stateless Composable
>
> A composable that receives state from outside.
>
> Example:
>
> ```kotlin
> @Composable
> fun Counter(
>     count: Int,
>     onIncrement: () -> Unit
> )
> ```
>
> Stateless composables are:
>
> * Reusable
> * Testable
> * Easier to maintain
>
> Modern Compose architecture prefers stateless composables with state hoisting.
>
> ---
>
> ## State Hoisting
>
> State hoisting means moving state outside composables to a higher-level owner like:
>
> * Parent composable
> * ViewModel
>
> Flow:
>
> ```text
> ViewModel → State → UI
> UI Events → Callback → ViewModel
> ```
>
> Benefits:
>
> * Single source of truth
> * Better architecture
> * Easier testing
> * Better reusability
>
> ---
>
> ## Important Warning About Mutable Collections
>
> Using mutable collections directly as state can cause stale or incorrect UI.
>
> Bad Example:
>
> ```kotlin
> val list = mutableListOf<String>()
> ```
>
> Compose cannot observe changes inside normal mutable collections because they are not observable.
>
> Updating the list may not trigger recomposition.
>
> Recommended approach:
>
> ```kotlin
> val listState = mutableStateOf(listOf<String>())
> ```
>
> Use immutable collections with observable state holders.
>
> ---
>
> ## Key Points About Compose State
>
> Good Compose state should be:
>
> * Observable
> * Immutable when possible
> * Single source of truth
> * Hoisted to ViewModel when needed
>
> ---
>
> ## Interview One-Liner
>
> > State in Jetpack Compose is any observable value that can change over time and affect the UI. When state changes, Compose automatically triggers recomposition and updates only the affected UI components.



