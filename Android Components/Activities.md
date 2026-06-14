

## 🔹 What is an Activity?

* An **Activity** is a single **screen** in an Android app that the user can interact with.
* Example:

  * A **Login screen** = one activity
  * A **Home screen** = another activity

Think of it like **pages in a book** → each page = one activity.

---

## 🔹 Activity Lifecycle

An Activity goes through multiple states depending on what the user is doing:

1. **onCreate()** → called when the activity is created (initialize UI, data, bindings).
2. **onStart()** → activity is visible but not yet interactive.
3. **onResume()** → activity is visible **and** interactive (in the foreground).
4. **onPause()** → another activity comes in front (partially visible, still alive).
5. **onStop()** → activity completely hidden (but kept in memory).
6. **onDestroy()** → activity is killed (resources released).
7. **onRestart()** → activity comes back after being stopped.

👉 Important for handling memory, releasing resources, and saving/restoring state.

---

## 🔹 Example

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        println("onCreate called")
    }

    override fun onStart() {
        super.onStart()
        println("onStart called")
    }

    override fun onResume() {
        super.onResume()
        println("onResume called")
    }

    override fun onPause() {
        super.onPause()
        println("onPause called")
    }

    override fun onStop() {
        super.onStop()
        println("onStop called")
    }

    override fun onDestroy() {
        super.onDestroy()
        println("onDestroy called")
    }
}
```

---

## 🔹 Launching Activities

* You move from one activity to another using **Intent**:

```kotlin
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)
```

---

## 🔹 Types of Activities

1. **Main / Launcher Activity** → Entry point of the app.
2. **Normal Activity** → Any screen like Home, Settings, Profile.
3. **Dialog Activity** → Looks like a dialog but is still an Activity.
4. **Transparent Activity** → Background visible (like permissions overlay).

---

## 🔹 Why Activities are Important?

* They define **user flow** in Android.
* They manage **UI + user interactions**.
* They handle **navigation** across app screens.

---

# 🔹 Basic Questions

### 1. What is an Activity Lifecycle in Android?

* **Answer**: The sequence of states an Activity goes through, from creation to destruction. Managed by the Android OS to optimize resources.

---

### 2. Can you explain the lifecycle methods of an Activity?

* **Answer**:

  * `onCreate()` → initialize UI and data
  * `onStart()` → activity visible but not yet interactive
  * `onResume()` → activity in foreground, interactive
  * `onPause()` → partially visible, another activity on top
  * `onStop()` → completely hidden
  * `onDestroy()` → cleanup before activity is killed
  * `onRestart()` → activity restarted after being stopped

---

### 3. Difference between onCreate() and onStart()?

* **Answer**:

  * `onCreate()` → called **once**, when Activity is created → setup initial UI, variables.
  * `onStart()` → called every time Activity becomes visible → even after coming back from background.

---

### 4. What is the difference between onPause() and onStop()?

* **Answer**:

  * `onPause()` → Activity is **partially visible**, but not interactive.
  * `onStop()` → Activity is **completely hidden**.

---

### 5. When is onDestroy() called?

* **Answer**:

  * When the activity is finishing (user presses back).
  * Or when system kills it to free memory.

⚠️ Not guaranteed to be called always (e.g., if OS kills app directly).

---

# 🔹 Intermediate Questions

### 6. What happens when the device is rotated?

* **Answer**:

  * Activity is **destroyed** and **recreated** by default.
  * Lifecycle: `onPause()` → `onStop()` → `onDestroy()` → `onCreate()` → `onStart()` → `onResume()`.
  * To handle this, we use `onSaveInstanceState()` and `onRestoreInstanceState()`.

---

### 7. How do you save Activity state during configuration changes?

* **Answer**:

  * Use `onSaveInstanceState(Bundle)` to save temporary UI state.
  * Restore in `onCreate()` or `onRestoreInstanceState()`.
  * Or use **ViewModel** (Jetpack way).

---

### 8. What is the difference between finish() and onBackPressed()?

* **Answer**:

  * `finish()` → programmatically ends the Activity.
  * `onBackPressed()` → simulates physical back button, which **by default calls finish()**, but you can override it.

---

### 9. What’s the difference between singleTask, singleTop, and standard launch modes?

* **Answer**:

  * **standard** → new instance created every time.
  * **singleTop** → if already on top, reuses it.
  * **singleTask** → only one instance exists, clears above activities.

---

# 🔹 Advanced Questions

### 10. What happens when you press Home button?

* **Answer**:

  * Lifecycle: `onPause()` → `onStop()`
  * Activity goes to **background**, but not destroyed.

---

### 11. What happens when you press Back button?

* **Answer**:

  * Lifecycle: `onPause()` → `onStop()` → `onDestroy()`
  * Activity is finished and removed from back stack.

---

### 12. Is onSaveInstanceState() always called?

* **Answer**:

  * No. Called only when Activity **may be destroyed and recreated** (like rotation, memory pressure).
  * Not called if user presses Back (because Activity is finished, not recreated).

---

### 13. Difference between onCreate() and onRestoreInstanceState()?

* **Answer**:

  * `onCreate()` → always called when Activity is created.
  * `onRestoreInstanceState()` → called **only if** Activity is recreated after being killed, and Bundle has saved state.

---

### 14. What happens if the system kills your Activity in the background?

* **Answer**:

  * When user comes back, Activity is recreated from scratch.
  * `onCreate()` + `onRestoreInstanceState()` are triggered with saved data.

---

### 15. How does ViewModel help with lifecycle?

* **Answer**:

  * ViewModel survives configuration changes like rotation.
  * Data persists across `onCreate()` calls.
  * Lifecycle-aware → avoids memory leaks.

---


## 🔹 `onSaveInstanceState(Bundle outState)`

* **When called?**

  * Just **before your activity can be destroyed** by the system (e.g., rotation, low memory).
  * **Not called** when user explicitly finishes the activity (e.g., pressing back).

* **Purpose**

  * Save **temporary UI-related state** (scroll position, input text, selection, etc.).
  * Put values into the `Bundle outState`.

* **Example**

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("username", usernameEditText.text.toString())
    outState.putInt("score", currentScore)
}
```

---

## 🔹 `onRestoreInstanceState(Bundle savedInstanceState)`

* **When called?**

  * After `onStart()` if the Activity is **being recreated** and a saved state exists.
  * **Only called if `onSaveInstanceState()` was called earlier.**

* **Purpose**

  * Restore values you previously saved in the bundle.

* **Example**

```kotlin
override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    val username = savedInstanceState.getString("username")
    val score = savedInstanceState.getInt("score")
    usernameEditText.setText(username)
    currentScore = score
}
```

---

## 🔹 Difference between `onCreate()` and `onRestoreInstanceState()`

* You can **also restore state inside `onCreate()`**, because it receives the same `Bundle`.
* Difference:

  * `onCreate(savedInstanceState)` → called every time Activity starts (first time or recreation).
  * `onRestoreInstanceState(savedInstanceState)` → called **only if** state is being restored (i.e., Activity was killed & recreated).

👉 Best practice:

* **Save in `onSaveInstanceState()`**
* **Restore in `onCreate()`** (since it always runs).
* Use `onRestoreInstanceState()` only if you want to **separate initialization vs restoration logic**.

---

## 🔹 Lifecycle Flow Example (Rotation Case)

When you rotate the device:

```
onPause() → onSaveInstanceState() → onStop() → onDestroy()
→ onCreate() → onStart() → onRestoreInstanceState() → onResume()
```

---

✅ **Interview Tip**: If asked in interview
👉 Say: *“`onSaveInstanceState` is used to store temporary UI data in a bundle before the Activity is destroyed (like on rotation). This data is later restored in either `onCreate()` or `onRestoreInstanceState()` when the Activity is recreated. Unlike permanent storage (SharedPreferences, DB), this is only for short-term state persistence.”*

---

Got it 👍 You want this behavior:

👉 You opened activities in order **A → B → C → D**.
👉 On **single back press** from `D`, instead of going to `C`, the back stack should collapse to **A → B** (skipping `C`).

---

## 🔹 Solution Approaches

### **1. Using `FLAG_ACTIVITY_CLEAR_TOP`**

When you open `D` from `C`, instead of the normal `startActivity()`, you can control the back stack.

```kotlin
val intent = Intent(this, B::class.java)
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
startActivity(intent)
```

* This clears all activities **above B** (so C is removed).
* Back stack becomes `A → B`.
* If you press back now, you’ll go back to `A`.

---

### **2. Finishing activities manually**

When navigating from `C` to `D`, you can just `finish()` `C`.

```kotlin
// From C when opening D
val intent = Intent(this, D::class.java)
startActivity(intent)
finish() // removes C from stack
```

Now the stack is: `A → B → D`
On back press → `A → B`.

---

### **3. Custom Back Navigation in D**

If you always want to go directly back to `B` from `D`:

```kotlin
override fun onBackPressed() {
    val intent = Intent(this, B::class.java)
    intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
    startActivity(intent)
    finish()
}
```

This way, pressing **Back in D** → kills `C` and returns to `B`.

---

### **4. Jetpack Navigation Component (Modern Way)**

If you’re using **Fragments + Navigation Component**, you can pop multiple destinations in one go:

```kotlin
findNavController().popBackStack(R.id.BFragment, false)
```

This clears everything on top of `BFragment` in the navigation stack.

---

✅ **Best Interview Answer**

> *“To control the activity back stack, we can use intent flags like `FLAG_ACTIVITY_CLEAR_TOP` to remove activities above a target activity. For example, from D I can launch B with `CLEAR_TOP`, which clears C and keeps the stack as A → B. Alternatively, I can finish activities programmatically or use the Navigation Component for fragment-based navigation.”*

---
👉 *“If you go from Activity A → Activity B, what lifecycle methods are called on both?”*

Let’s break it step by step.

---

## 🔹 Case 1: Open **B** from **A**

```kotlin
val intent = Intent(this, B::class.java)
startActivity(intent)
```

### Lifecycle sequence:

* **Activity A**

  * `onPause()` (A is partially visible while B is launching)
* **Activity B**

  * `onCreate()` (first time creation)
  * `onStart()`
  * `onResume()`
* **Activity A**

  * `onStop()` (now fully hidden)

✅ Final State:

* B → Resumed (foreground, interactive)
* A → Stopped (in background)

---

## 🔹 Case 2: Press Back (B → A)

When user presses back on B:

### Lifecycle sequence:

* **Activity B**

  * `onPause()`
* **Activity A**

  * `onRestart()` (because it was stopped)
  * `onStart()`
  * `onResume()`
* **Activity B**

  * `onStop()`
  * `onDestroy()`

✅ Final State:

* A → Resumed
* B → Destroyed

---

## 🔹 Case 3: If Activity B is **Transparent / Dialog Theme**

* A’s `onPause()` will be called (partially visible).
* But **A won’t go to onStop()** (still visible behind).
* B goes through full `onCreate() → onStart() → onResume()`.

---

✅ **Interview Tip**
When asked this, you can confidently say:

👉 *“When we navigate from A → B, A goes through `onPause()` → `onStop()`. B goes through `onCreate()` → `onStart()` → `onResume()`. When we press back from B, B goes through `onPause()` → `onStop()` → `onDestroy()`, and A goes through `onRestart()` → `onStart()` → `onResume()`.”*

---

Perfect! Let’s create a **complete Android Activity Lifecycle interview sheet** you can use to practice. I’ll include **questions, answers, and lifecycle flow explanations**.

---

# **📄 Android Activity Lifecycle Interview Sheet**

---

## **1. What is an Activity Lifecycle?**

**Answer:**

* The sequence of states an Activity goes through from creation to destruction.
* Managed by Android OS to optimize memory and user experience.

**Key Methods:** `onCreate()`, `onStart()`, `onResume()`, `onPause()`, `onStop()`, `onDestroy()`, `onRestart()`.

---

## **2. Lifecycle Methods Called When A → B**

**Scenario:** Activity A starts Activity B.

**Sequence:**

* **A:** `onPause()` → `onStop()`
* **B:** `onCreate()` → `onStart()` → `onResume()`

**Back Press from B → A:**

* **B:** `onPause()` → `onStop()` → `onDestroy()`
* **A:** `onRestart()` → `onStart()` → `onResume()`

---

## **3. Difference Between onPause() and onStop()**

* `onPause()` → partially visible, user can’t interact.
* `onStop()` → fully hidden, Activity in background.

**Tip:** Save lightweight data in `onPause()`, release heavy resources in `onStop()`.

---

## **4. When is onRestart() called?**

* Called when Activity moves from **stopped → started** (after background to foreground).

---

## **5. onSaveInstanceState() vs onRestoreInstanceState()**

* `onSaveInstanceState()` → Save UI state before Activity is destroyed (rotation, low memory).
* `onRestoreInstanceState()` → Restore saved UI state after Activity is recreated.

**Flow Example (Rotation):**

```
onPause() → onSaveInstanceState() → onStop() → onDestroy()
→ onCreate() → onStart() → onRestoreInstanceState() → onResume()
```

---

## **6. What happens when Home is pressed?**

* **Lifecycle:** `onPause()` → `onStop()`
* Activity goes to background, not destroyed.

---

## **7. What happens when Back is pressed?**

* **Lifecycle:** `onPause()` → `onStop()` → `onDestroy()`
* Activity is removed from back stack.

---

## **8. What happens on Device Rotation?**

* By default, Activity is **destroyed and recreated**.
* **Lifecycle Calls:**

```
onPause() → onSaveInstanceState() → onStop() → onDestroy()
→ onCreate() → onStart() → onRestoreInstanceState() → onResume()
```

---

## **9. What are Launch Modes & How They Affect Lifecycle?**

| Mode             | Behavior                                          |
| ---------------- | ------------------------------------------------- |
| `standard`       | New instance created every time                   |
| `singleTop`      | Reuses Activity if it’s on top                    |
| `singleTask`     | Only one instance exists; clears above activities |
| `singleInstance` | Only one instance in a separate task              |

---

## **10. How to Clear Back Stack Programmatically?**

```kotlin
val intent = Intent(this, B::class.java)
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
startActivity(intent)
```

* Removes all Activities above target.

---

## **11. Activity Lifecycle for Transparent / Dialog Activities**

* Underlying Activity goes **onPause()** (partially visible).
* Transparent Activity: `onCreate()` → `onStart()` → `onResume()`.

---

## **12. Retaining Data Across Configuration Changes**

* Use `onSaveInstanceState()` / `onRestoreInstanceState()`.
* **Better:** Use **ViewModel**, survives rotation without lifecycle callbacks.

---

## **13. Trick Questions**

* Can `onDestroy()` be skipped? → Yes, if OS kills Activity directly.
* Can you manually call `onDestroy()`? → Not recommended; use `finish()` instead.
* What happens to Activity A → B → C → BackPressed?

  * Explain lifecycle step by step for **all three activities**.

---

## **14. Fragments vs Activities**

* Fragments have **own lifecycle**, but **dependent on host Activity**.
* Useful for single-activity apps with navigation.

---

## **15. Quick Practice Questions**

1. What happens if you open Activity in **singleTop mode**?
2. Difference between `onStart()` and `onResume()`?
3. How to save **scroll position** during rotation?
4. Activity A launches B, B launches C → Press Back → Lifecycle calls?
5. When should you release **heavy resources**?

---

✅ **Tip for Interviews:**

* Always mention **step-by-step lifecycle calls**.
* Use **flow diagrams** or write a **table of A vs B** for navigation questions.
* Show understanding of **temporary vs permanent state** (`onSaveInstanceState` vs database).

---

## **Scenario**

* Activities: `A → B → C → D`
* Requirement: From `D`, pressing back should take you **directly to B**, skipping `C`.

---

## **1️⃣ Understanding the Flags**

### **FLAG_ACTIVITY_CLEAR_TOP**

* If the target activity (`B`) already exists in the back stack:

  * **All activities above it are cleared** (`C` and `D` in our example).
  * `B` is **brought to the front**.

### **FLAG_ACTIVITY_SINGLE_TOP**

* If the target activity (`B`) is already **at the top of the stack**,

  * **No new instance** is created.
* Prevents unnecessary recreation of the activity.

---

## **2️⃣ Full Code Example**

### **Activity B**

```kotlin
class BActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_b)

        val btnToC = findViewById<Button>(R.id.btnToC)
        btnToC.setOnClickListener {
            val intent = Intent(this, CActivity::class.java)
            startActivity(intent)
        }
    }
}
```

---

### **Activity C**

```kotlin
class CActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_c)

        val btnToD = findViewById<Button>(R.id.btnToD)
        btnToD.setOnClickListener {
            val intent = Intent(this, DActivity::class.java)
            startActivity(intent)
        }
    }
}
```

---

### **Activity D**

Here’s where we handle **jump back to B**:

```kotlin
class DActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_d)

        val btnBackToB = findViewById<Button>(R.id.btnBackToB)
        btnBackToB.setOnClickListener {
            val intent = Intent(this, BActivity::class.java)

            // Clear all activities on top of B
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
            startActivity(intent)
            // Optionally finish D so it doesn't stay in the stack
            finish()
        }
    }
}
```

---

## **3️⃣ Lifecycle Explanation**

1. Stack before pressing the button:

```
A → B → C → D
```

2. Press button in D:

* `B` exists in the stack, so **CLEAR_TOP** removes **C and D**.
* Stack after:

```
A → B
```

* `B` receives **`onNewIntent()`** if you override it (because SINGLE_TOP prevents recreation).

---

## **4️⃣ Optional: Handle onNewIntent() in B**

```kotlin
override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent)
    // Handle any data sent from D
    Toast.makeText(this, "Returned from D", Toast.LENGTH_SHORT).show()
}
```

---

✅ **Key Points**

* `CLEAR_TOP` → removes all activities above the target in the stack.
* `SINGLE_TOP` → avoids creating a new instance if target is already on top.
* Use `finish()` in D if you don’t want it to remain in memory.

---



