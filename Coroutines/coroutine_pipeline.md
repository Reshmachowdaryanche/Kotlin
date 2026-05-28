pipelines are one of the most important coroutine concepts for interviews because they show:

* coroutine communication
* streaming data
* channel chaining
* async transformations
* backpressure handling

Let’s deeply understand this step by step.

---

# What is a Pipeline?

A pipeline means:

```text
Stage 1 output → Stage 2 input → Stage 3 input ...
```

Each stage is usually:

* a coroutine
* connected using channels

---

# Real-World Analogy

Think of a factory assembly line.

```text
Raw Material
    ↓
Cutting Machine
    ↓
Painting Machine
    ↓
Packaging Machine
```

Each machine:

* receives input
* processes it
* sends output forward

That is exactly what Kotlin pipelines do.

---

# Basic Pipeline Structure

```text
Producer → Transformer → Consumer
```

Example:

```text
Numbers → Square → Print
```

---

# Stage 1 — Producer

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1

    while(true){
        send(x++)
    }
}
```

---

# What Happens Here?

This coroutine:

* continuously generates numbers
* sends them into a channel

---

# Internal Flow

```text
send(1)
send(2)
send(3)
send(4)
```

The returned object is:

```kotlin
ReceiveChannel<Int>
```

Meaning:

```text
Other coroutines can receive values from it
```

---

# Visualization

```text
Producer Channel:

[1] [2] [3] [4] ...
```

---

# Stage 2 — Transformer

```kotlin
fun CoroutineScope.square(
    input: ReceiveChannel<Int>
) = produce<Int> {

    for(x in input){
        send(x * x)
    }
}
```

This stage:

1. receives values from input channel
2. transforms them
3. sends transformed values forward

---

# Internal Flow

Input:

```text
1 2 3 4
```

Transformation:

genui{"math_block_widget_always_prefetch_v2":{"content":"y = x^2"}}

Output:

```text
1 4 9 16
```

---

# Important Line

```kotlin
for(x in input)
```

This continuously consumes values from the previous stage.

Equivalent idea:

```text
receive value
process
repeat forever
```

---

# Final Usage

```kotlin
val numbers = produceNumbers()
val squares = square(numbers)

repeat(5){
    println(squares.receive())
}
```

---

# Step-by-Step Execution

---

## Step 1

Producer creates:

```text
1 2 3 4 5 ...
```

---

## Step 2

Square stage receives:

```text
1
```

Calculates:

```text
1 * 1 = 1
```

Sends:

```text
1
```

---

## Step 3

Consumer receives:

```text
1
```

Prints it.

---

# Full Flow Visualization

```text
produceNumbers()
        ↓
   1 2 3 4 5
        ↓
square()
        ↓
   1 4 9 16 25
        ↓
consumer
```

---

# Why Pipelines Are Powerful

Pipelines allow:

| Feature              | Benefit                |
| -------------------- | ---------------------- |
| Separation of stages | Cleaner code           |
| Parallel processing  | Faster execution       |
| Streaming            | Infinite data possible |
| Reusable stages      | Modular architecture   |
| Backpressure         | Controlled flow        |

---

# Streaming Nature

Pipelines process values one-by-one.

They do NOT wait for all data.

Example:

```text
Generate 1 → Square 1 → Print 1
Generate 2 → Square 2 → Print 2
```

This is called:

```text
stream processing
```

---

# Important Interview Point

Pipelines are:

```text
lazy + asynchronous + streaming
```

because data flows continuously.

---

# Now the Prime Number Pipeline

This is a famous Kotlin coroutine example.

It implements a streaming version of:

\text{Sieve of Eratosthenes}

for prime number generation.

---

# Stage 1 — Number Generator

```kotlin
fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start

    while(true){
        send(x++)
    }
}
```

This generates:

```text
2 3 4 5 6 7 8 9 ...
```

---

# Stage 2 — Filter

```kotlin
fun CoroutineScope.filter(
    numbers: ReceiveChannel<Int>,
    prime: Int
) = produce<Int> {

    for(x in numbers){
        if(x % prime != 0){
            send(x)
        }
    }
}
```

This removes multiples of a prime.

---

# Example Walkthrough

---

## Initial Stream

```text
2 3 4 5 6 7 8 9 10 11 ...
```

---

# First Prime

```kotlin
val prime = cur.receive()
```

Receives:

```text
2
```

So:

```text
2 is prime
```

---

# Create Filter for 2

```kotlin
cur = filter(cur, 2)
```

Now multiples of 2 are removed.

Stream becomes:

```text
3 5 7 9 11 13 ...
```

---

# Next Iteration

Receive:

```text
3
```

Now:

```text
3 is prime
```

Create filter for 3.

Removes:

```text
6 9 12 15 ...
```

Remaining:

```text
5 7 11 13 17 ...
```

---

# Pipeline Visualization

```text
numbersFrom(2)
      ↓
filter(2)
      ↓
filter(3)
      ↓
filter(5)
      ↓
filter(7)
```

Each stage removes non-prime numbers.

---

# Amazing Thing Here

Each prime creates:

```text
a NEW coroutine pipeline stage
```

So pipeline grows dynamically.

---

# Why This Example Is Famous

Because it demonstrates:

* channels
* pipelines
* infinite streams
* coroutine communication
* lazy computation
* concurrent transformations

all together.

---

# Important Pipeline Characteristics

---

# 1. Each Stage Runs Concurrently

Every `produce {}` launches a coroutine.

So stages work independently.

---

# 2. Channels Connect Stages

```text
Stage A → Channel → Stage B
```

---

# 3. Backpressure Happens Naturally

If consumer is slow:

```text
Upstream send() suspends
```

So producer slows automatically.

---

# 4. Pipelines Can Be Infinite

Like:

```kotlin
while(true)
```

No need to store everything in memory.

---

# Real Production Use Cases

Pipelines are used in:

* video streaming
* image processing
* event systems
* socket processing
* background workers
* ETL systems
* log processing
* reactive architectures

---

# Simple Mental Model

Think:

```text
Coroutine = Worker
Channel = Conveyor Belt
Pipeline = Factory Line
```

---

# Interview One-Liner

> A coroutine pipeline is a chain of coroutines connected through channels where each stage processes and forwards streamed data asynchronously.
