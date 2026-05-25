# Service

A `Service` is an Android component that can perform long-running operations in the background.

- Runs on the **main thread (UI thread)** by default  
  → if you do heavy work here, it will block UI and cause **ANR (App Not Responding)**.

- If you need background threading, you must manually create a new thread inside the Service  
  (e.g., using `Thread`, `ExecutorService`, or `Coroutine`).

## Lifecycle Methods

### onCreate()
Called once when the service is created.

### onStartCommand()
Called whenever the service is started.

### onDestroy()
Called when the service is destroyed.

## Use Case

👉 When you want a service that runs indefinitely  
(e.g., music player, tracking location, real-time chat).
