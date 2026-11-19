# Java Multithreading Interview Questions (41 Q&A with Code)

Real, tricky multithreading questions for experienced developers, taken from a video on multithreading interview questions (supplied as a transcript). The questions are in the video's order (Q1 to Q41), grouped by topic.

How this file was made:
- The answers follow the explanations in the video (from its auto-generated transcript, with transcription slips fixed, such as "arr condition" for race condition and "CS" for CAS).
- The code examples are added here so each answer can be run and checked. Lines marked *(added)* are extra notes that are not in the video.
- Code snippets assume `import java.util.concurrent.*;`, `java.util.concurrent.atomic.*` and `java.util.concurrent.locks.*` where needed.

## Contents

1. [Threads and processes (Q1-Q2)](#1-threads-and-processes-q1-q2)
2. [Creating threads (Q3-Q9)](#2-creating-threads-q3-q9)
3. [Thread life cycle (Q10-Q11)](#3-thread-life-cycle-q10-q11)
4. [Race conditions and `synchronized` (Q12-Q17)](#4-race-conditions-and-synchronized-q12-q17)
5. [ReentrantLock (Q18-Q22)](#5-reentrantlock-q18-q22)
6. [volatile (Q23-Q25)](#6-volatile-q23-q25)
7. [Atomic classes (Q26-Q27)](#7-atomic-classes-q26-q27)
8. [Executor framework and thread pools (Q28-Q35)](#8-executor-framework-and-thread-pools-q28-q35)
9. [CompletableFuture (Q36-Q40)](#9-completablefuture-q36-q40)
10. [ForkJoinPool (Q41)](#10-forkjoinpool-q41)

---

## 1. Threads and processes (Q1-Q2)

### Q1. Is a thread lighter than a process, and can it exist without one?
**Answer: Yes, a thread is lighter than a process, and it cannot exist without a process.** A *process* is an instance of a program being executed. When a program runs, the operating system creates a process to manage its execution (for example, Microsoft Word is a process). A *thread* is the smallest unit of execution within a process. A process can have multiple threads which share the same resources but run independently (a web browser like Chrome may use multiple threads for different tabs). A thread lives inside a process. Think of a process as a house and a thread as a room.

| | Process | Thread |
|---|---|---|
| What it is | A running program | Smallest unit of execution inside a process |
| Weight | Heavier | Lighter |
| Resources | Own resources | Shares the process's resources |
| Can exist alone | Yes | No, it lives inside a process |

### Q2. Multiple users hit the same REST API at the same time. Are they served by one thread or multiple?
**Answer: Multiple threads.** The server assigns a separate thread to each request, so requests run in parallel and not one by one. *(added: web servers such as Tomcat do this with a thread pool.)*
```java
ExecutorService server = Executors.newFixedThreadPool(3);
for (int i = 1; i <= 5; i++) {
    int req = i;
    server.submit(() ->
        System.out.println("request " + req + " served by " + Thread.currentThread().getName()));
}
server.shutdown();
```

---

## 2. Creating threads (Q3-Q9)

### Q3. What are the different ways to create a thread?
**Answer:** In older Java there were two ways: extend the `Thread` class, or implement the `Runnable` interface. From **Java 5**, threads can also be run using the **Executor Service framework** along with `Callable`. From **Java 8**, threads can be created with **lambda expressions**, which makes the code more concise and readable.
```java
// 1. Extend Thread
class MyThread extends Thread {
    public void run() { System.out.println("extended Thread"); }
}
new MyThread().start();

// 2. Implement Runnable
class MyTask implements Runnable {
    public void run() { System.out.println("Runnable"); }
}
new Thread(new MyTask()).start();

// 3. Executor service with Callable (Java 5)
ExecutorService ex = Executors.newSingleThreadExecutor();
Future<String> f = ex.submit(() -> "Callable result");
ex.shutdown();

// 4. Lambda (Java 8)
new Thread(() -> System.out.println("lambda")).start();
```
*(added: Java 21 also has virtual threads: `Thread.ofVirtual().start(() -> { })`.)*

### Q4. What is the difference between extending `Thread` and implementing `Runnable`, and when should each be used?
**Answer:**
- **Extending `Thread`:** our class *becomes* a thread. We put the code in `run()` and start it with `start()`. The limitation is that the class cannot extend any other class, because Java allows only one class to be extended. Use it for simple demos, learning, or very small programs where inheritance is not needed.
- **Implementing `Runnable`:** our class represents a *task*, not the thread itself. We put the logic in `run()` and pass it to a `Thread` object. The class can still extend another class. Use it in real-world applications, especially when the class already extends another class.
```java
class A extends Thread {                       // class is the thread, cannot extend anything else
    public void run() { System.out.println("A running"); }
}

class Report extends BaseReport implements Runnable {   // still free to extend another class
    public void run() { System.out.println("Report running"); }
}
new A().start();
new Thread(new Report()).start();
```

### Q5. Can a class extend `Thread` and implement `Runnable` together?
**Answer: Yes, but it makes no real sense.** `Thread` already implements `Runnable`, so a class that extends `Thread` is automatically a `Runnable`. Implementing `Runnable` again adds nothing.
```java
class Worker extends Thread implements Runnable {   // compiles, but the "implements Runnable" is redundant
    public void run() { System.out.println("working"); }
}
```

### Q6. Why is `Runnable` preferred in real-world applications?
**Answer: It gives better design and flexibility.** With `Runnable`, our class defines only *what work to do*, not *how to run the thread*. This keeps things clean and reusable, and the class can still extend another class, which is very common in real projects.
```java
Runnable task = () -> System.out.println("what to do");   // the work
new Thread(task).start();                                  // how to run it: plain thread
Executors.newFixedThreadPool(2).submit(task);              // how to run it: thread pool
```

### Q7. What happens internally when we call `run()` instead of `start()`?
**Answer: No new thread is created.** `run()` runs like a normal method call on the same thread that called it, usually the main thread. Java does not create a separate thread or do any scheduling. When we call `start()`, Java creates a new thread, and that new thread then calls `run()` internally.
```java
Thread t = new Thread(() -> System.out.println(Thread.currentThread().getName()));
t.run();     // prints: main
t.start();   // prints: Thread-0
```

### Q8. Can one `Runnable` instance be used by multiple threads?
**Answer: Yes.** We can create one object that has the `run()` code and give that same object to many `Thread` objects. *(added: if that object holds shared mutable state, it needs synchronization.)*
```java
Runnable task = () -> System.out.println("run by " + Thread.currentThread().getName());
new Thread(task, "T1").start();
new Thread(task, "T2").start();
new Thread(task, "T3").start();
```

### Q9. What happens if `start()` is called twice on the same thread?
**Answer: Java throws a runtime error.** The first `start()` works and creates a new thread. The second `start()` throws `IllegalThreadStateException`. A thread can be started only once: after it is started or finished, it cannot be restarted.
```java
Thread t = new Thread(() -> System.out.println("hi"));
t.start();
t.start();   // IllegalThreadStateException
```

---

## 3. Thread life cycle (Q10-Q11)

### Q10. Explain the life cycle of a thread.
**Answer:** A thread goes through these states from start to end:
1. **New:** the thread is created but not started (`Thread t = new Thread(...)`).
2. **Runnable:** after `start()`, the thread is ready to run and waiting for CPU time.
3. **Running:** the thread is actually executing its `run()` method. This state is managed by the CPU scheduler.
4. **Blocked / Waiting:** the thread is temporarily not running. This happens when it waits for a lock, calls `sleep()` or `wait()`, or waits for another thread to finish.
5. **Terminated:** the thread finishes its work or stops due to an error. It cannot be restarted.

*(added: in code, `Thread.State` has `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING` and `TERMINATED`. There is no separate `RUNNING` constant: Java reports a running thread as `RUNNABLE`.)*
```java
Thread t = new Thread(() -> {
    try { Thread.sleep(200); } catch (InterruptedException e) { }
});
System.out.println(t.getState());   // NEW
t.start();
System.out.println(t.getState());   // RUNNABLE
Thread.sleep(50);
System.out.println(t.getState());   // TIMED_WAITING (inside sleep)
t.join();
System.out.println(t.getState());   // TERMINATED
```

### Q11. Can a thread re-enter `RUNNABLE` after it is terminated?
**Answer: No.** Once a thread finishes its `run()` method or ends due to an error, it goes into the terminated state and its life cycle is completely over. Java does not allow restarting a dead thread. Calling `start()` again throws an exception.
```java
Thread t = new Thread(() -> { });
t.start();
t.join();
t.start();   // IllegalThreadStateException, the thread is terminated
```

---

## 4. Race conditions and `synchronized` (Q12-Q17)

### Q12. Two threads update a counter at the same time. What happens and how do we solve it?
**Answer: A race condition.** Both threads may read the same old value, increase it and write it back. Because this happens at the same time, one update can override the other, and the counter value becomes wrong. Solutions: use `synchronized` so only one thread updates at a time, use `AtomicInteger` which updates safely without locking, or use locks.
```java
class Unsafe {
    int count;
    void inc() { count++; }                          // race condition
}

class WithSynchronized {
    int count;
    synchronized void inc() { count++; }             // fix 1
}

class WithAtomic {
    AtomicInteger count = new AtomicInteger();
    void inc() { count.incrementAndGet(); }          // fix 2
}

class WithLock {
    int count;
    final ReentrantLock lock = new ReentrantLock();
    void inc() {                                      // fix 3
        lock.lock();
        try { count++; } finally { lock.unlock(); }
    }
}
```

### Q13. How can we use the `synchronized` keyword?
**Answer: Three ways.**
1. **Synchronized method:** add `synchronized` before the method name, which locks the entire method.
2. **Synchronized block:** use a block, and only the code in that block is locked.
3. **Static synchronized:** the lock is on the **class**, not the object. The keyword goes after `static` and before the method name.
```java
class Counter {
    private int count;
    static int total;

    synchronized void inc() { count++; }                 // 1. method: locks "this"

    void inc2() {
        synchronized (this) { count++; }                 // 2. block: locks only this section
    }

    static synchronized void incTotal() { total++; }     // 3. static: locks Counter.class
}
```

### Q14. Is `synchronized` applied to code or to objects?
**Answer: To objects, not directly to code.** When a method or block is marked `synchronized`, the code gets a lock on an object before it runs. Only one thread can hold that object's lock at a time. For an instance synchronized method the lock is on the current object. For a static synchronized method the lock is on the class object.
```java
class Demo {
    private final Object lockObj = new Object();

    synchronized void a() { }                    // lock = this
    static synchronized void b() { }             // lock = Demo.class
    void c() { synchronized (lockObj) { } }      // lock = lockObj
}
```

### Q15. What happens if an exception occurs inside a `synchronized` block?
**Answer: The lock is released automatically.** Step by step: a thread enters the block and gets the lock, an exception occurs, the thread exits the block because of the exception, and Java releases the lock automatically.
```java
Object lock = new Object();
Thread t = new Thread(() -> {
    synchronized (lock) {
        throw new RuntimeException("boom");   // lock is released when this propagates out
    }
});
t.start();
t.join();
synchronized (lock) {
    System.out.println("main got the lock");  // works, the lock was released
}
```

### Q16. Can synchronization guarantee thread ordering?
**Answer: No.** Synchronization makes sure only one thread at a time enters the **critical section**, which is a part of code that accesses shared data and must not be executed by more than one thread at the same time (like a counter). It protects shared data from corruption. It does **not** decide which thread goes first and does not control execution order. Thread scheduling is decided by the CPU and the JVM, not by the `synchronized` keyword.
```java
Object lock = new Object();
for (int i = 1; i <= 3; i++) {
    int id = i;
    new Thread(() -> {
        synchronized (lock) { System.out.println("thread " + id); }
    }).start();
}
// Always one at a time, but the print order (1,2,3 / 2,1,3 / ...) is not guaranteed
```

### Q17. What are the limitations of synchronization?
**Answer:**
1. **Performance issue:** only one thread can enter at a time, so the others wait. This can slow the application when many threads are used.
2. **No fairness guarantee:** it does not guarantee which waiting thread gets the lock first. Some threads may wait longer.
3. **Risk of deadlock:** if locks are taken in the wrong order, threads can wait forever for each other.
4. **Blocking nature:** waiting threads are blocked and cannot do any other work.
5. **Hard to control:** we cannot try to get a lock, set timeouts, or interrupt a thread waiting for a synchronized lock.

Because of these limitations, `ReentrantLock` was introduced in Java 5.
```java
// Deadlock risk: opposite lock order in two threads
// Thread 1: synchronized (a) { synchronized (b) { } }
// Thread 2: synchronized (b) { synchronized (a) { } }
```

---

## 5. ReentrantLock (Q18-Q22)

### Q18. What is `ReentrantLock` and why do we need it?
**Answer:** A `ReentrantLock` is a more powerful locking mechanism that controls access to shared code or data. **Reentrant** means the same thread can lock the same lock again and again without getting stuck, and unlock it the same number of times. It was introduced to solve the problems of `synchronized` and gives more control over locking and unlocking:
- try to get a lock without waiting forever, with `tryLock()`
- fairness, so threads get the lock in order
- time limits
- multiple condition variables
```java
ReentrantLock fair = new ReentrantLock(true);       // fair: longest-waiting thread goes first
Condition notEmpty = fair.newCondition();            // multiple conditions per lock
boolean got = fair.tryLock(1, TimeUnit.SECONDS);     // wait at most 1 second
```

### Q19. How does `ReentrantLock` work?
**Answer:** Use `lock()` to lock the critical section and `unlock()` to release it once the work is done. We can also use `tryLock()`: it does not block forever. If the lock is already held by another thread, it returns `false` immediately, so the thread can do something else instead of waiting.
```java
ReentrantLock lock = new ReentrantLock();

lock.lock();
try {
    // critical section
} finally {
    lock.unlock();            // always unlock in finally
}

if (lock.tryLock()) {
    try { /* critical section */ } finally { lock.unlock(); }
} else {
    // lock is busy: do something else instead of waiting
}
```

### Q20. How does `tryLock()` prevent deadlocks?
**Answer:** It does not force a thread to wait forever for a lock. A deadlock normally happens when two threads hold different locks and wait for each other's lock. With `tryLock()`, a thread tries to get the lock, and if it is not available it fails immediately or after a short time. The thread can then release its other locks, retry later or do something else. Because the thread does not block forever, circular waiting is avoided.
```java
ReentrantLock lockA = new ReentrantLock();
ReentrantLock lockB = new ReentrantLock();

boolean transfer() throws InterruptedException {
    if (lockA.tryLock(100, TimeUnit.MILLISECONDS)) {
        try {
            if (lockB.tryLock(100, TimeUnit.MILLISECONDS)) {
                try {
                    // work with both resources
                    return true;
                } finally { lockB.unlock(); }
            }
        } finally { lockA.unlock(); }
    }
    return false;    // could not get both locks: back off and retry later
}
```

### Q21. What happens if you forget to unlock?
**Answer:** The lock is never released. Other threads wait forever to get it, and the program may appear stuck or frozen. This can lead to deadlock or starvation, because the only thread that could release the lock has already finished or moved on.
```java
lock.lock();
doWork();                // if this throws, or we forget unlock(), everyone else waits forever
lock.unlock();           // wrong: not guaranteed to run

lock.lock();
try { doWork(); }
finally { lock.unlock(); }   // correct
```

### Q22. When should you prefer `ReentrantLock` over `synchronized`?
**Answer:**

| Use `ReentrantLock` when | Use `synchronized` when |
|---|---|
| We don't want threads to wait forever (`tryLock`, timeouts) | The logic is simple |
| We need fairness so threads get the lock in order | We want automatic lock handling (released for us, even on exceptions) |
| We want to lock and unlock at different places in the code | Readability and safety matter more than flexibility |
| We need better handling in complex multi-threading scenarios | |

---

## 6. volatile (Q23-Q25)

### Q23. What is the `volatile` keyword?
**Answer:** `volatile` makes sure that when one thread changes a variable, all other threads see the latest value immediately. It avoids the problem where threads use old or cached values. Each thread keeps its own local copy of variables in CPU cache or registers, but a `volatile` variable is read directly from main memory.

Example from the video: a music player has a play thread that keeps playing songs in a loop, and a stop button handled by the main thread. Both share a variable `stopPlaying`. If it is not `volatile`, the play thread may keep using an old cached value and play forever. If it is `volatile`, the play thread sees the update immediately and stops.
```java
class Player {
    private volatile boolean stopPlaying = false;

    void playLoop() {                    // runs on the play thread
        while (!stopPlaying) {
            // play next song
        }
    }
    void stop() { stopPlaying = true; }  // called from the main thread
}
```

### Q24. Why doesn't `volatile` guarantee atomicity?
**Answer:** Atomicity means an operation happens in one single step. Many operations like `count++` are actually multiple steps: read the value, increase it, write it back. `volatile` only guarantees **visibility**, not that these steps happen together. So two threads can still read the same value and update it at the same time, causing wrong results.
```java
class Counter { volatile int count; }
Counter c = new Counter();
Runnable r = () -> { for (int i = 0; i < 100_000; i++) c.count++; };
Thread a = new Thread(r), b = new Thread(r);
a.start(); b.start();
a.join(); b.join();
System.out.println(c.count);    // usually less than 200000: increments were lost
```

### Q25. Can `volatile` fix race conditions?
**Answer: No.** What `volatile` does is make sure all threads see the latest value. That is *visibility*. What a race condition needs is *atomicity*: only one thread updates at a time. Use `synchronized`, a lock, or an atomic class instead.
```java
volatile int count;                       // visibility only: count++ is still a race
AtomicInteger safe = new AtomicInteger(); // atomicity: safe.incrementAndGet()
```

---

## 7. Atomic classes (Q26-Q27)

### Q26. What are atomic classes and how do they work internally?
**Answer:** Atomic classes like `AtomicInteger` and `AtomicLong` let multiple threads update shared values safely **without using locks**. They are used when many threads change a value at the same time, like counters. Internally they use **CAS (Compare And Swap)**: read the current value, check whether it is still the same as expected, and if yes update it; if no, try again. If another thread changes the value in between, the update fails and CAS retries.
```java
AtomicInteger counter = new AtomicInteger();
counter.incrementAndGet();

// What it does internally, written by hand:
int prev, next;
do {
    prev = counter.get();                          // read current value
    next = prev + 1;
} while (!counter.compareAndSet(prev, next));      // update only if still equal to prev, else retry
```

### Q27. Why are atomics faster than `synchronized`?
**Answer:** Because they avoid blocking threads. With `synchronized`, threads wait and block if the lock is taken, and blocking and waking threads is slow and costly. With atomic classes threads do not block: they use CAS to retry instead of waiting. If an update fails, the thread just tries again. That means less waiting, no lock management and fewer context switches. *(added: under very heavy contention the retries themselves cost CPU, so the advantage is largest at low to moderate contention.)*
```java
// synchronized: a losing thread is blocked and parked by the OS
synchronized void inc() { count++; }

// atomic: a losing thread just retries the CAS loop
void inc2() { atomicCount.incrementAndGet(); }
```

---

## 8. Executor framework and thread pools (Q28-Q35)

### Q28. What is the Executor framework and why do we need it?
**Answer:** It is a Java feature that manages threads for us. Instead of creating and starting threads manually, we submit tasks and Java decides which thread runs them and when. We need it because creating threads again and again is slow and hard to manage: we would have to handle how many threads to create, when to reuse them and when to stop them. The executor framework reuses threads, controls how many threads run at once, makes code cleaner and safer, and improves performance and scalability.
```java
// Manual: new thread for every task
new Thread(task).start();

// Executor: we submit tasks, Java manages threads
ExecutorService ex = Executors.newFixedThreadPool(4);
ex.submit(task);
ex.shutdown();
```

### Q29. Explain the concept of a thread pool.
**Answer:** A thread pool is a group of ready-made threads used to do multiple tasks. Instead of creating a new thread each time a task comes, the pool gives the task to an existing idle thread. When the task is done, the thread goes back to the pool and waits for the next task.

Restaurant kitchen example: the threads are the **chefs**, the thread pool is the set of chefs already hired, and the orders are the **tasks**. When a customer places an order, the restaurant does not hire a new chef: an available chef takes the order, cooks it, becomes free and takes the next one. If all chefs are busy, new orders wait in a **queue** until a chef is free.
```java
ExecutorService chefs = Executors.newFixedThreadPool(2);   // 2 chefs already hired
for (int order = 1; order <= 5; order++) {
    int o = order;
    chefs.submit(() ->
        System.out.println("order " + o + " cooked by " + Thread.currentThread().getName()));
}
chefs.shutdown();
```

### Q30. Who manages the thread life cycle in executors?
**Answer:** The executor framework manages the entire thread life cycle. It creates threads, assigns tasks to them, reuses them, and shuts them down when needed. We don't manage threads manually.
```java
ExecutorService ex = Executors.newFixedThreadPool(4);   // executor creates the threads
ex.submit(task);                                         // executor assigns the task
ex.shutdown();                                           // stop accepting, finish queued tasks
if (!ex.awaitTermination(10, TimeUnit.SECONDS)) {
    ex.shutdownNow();                                    // force stop if still running
}
```

### Q31. Can executors reuse the same threads for different tasks?
**Answer: Yes.** After a task finishes, the thread goes back to the pool and waits for the next task.
```java
ExecutorService ex = Executors.newFixedThreadPool(2);
for (int i = 1; i <= 6; i++) {
    int id = i;
    ex.submit(() -> System.out.println("task " + id + " on " + Thread.currentThread().getName()));
}
ex.shutdown();
// 6 tasks, but only pool-1-thread-1 and pool-1-thread-2 appear: threads are reused
```

### Q32. What happens if the executor queue is full?
**Answer:** If the thread pool is busy and the task queue is full, the executor **rejects** new tasks. This usually throws a `RejectedExecutionException`, or follows a **rejection policy** like running the task in the caller thread or discarding it. *(added: the four built-in policies are `AbortPolicy` (default, throws), `CallerRunsPolicy`, `DiscardPolicy` and `DiscardOldestPolicy`.)*
```java
ExecutorService ex = new ThreadPoolExecutor(
        1, 1, 0, TimeUnit.SECONDS, new ArrayBlockingQueue<>(1));   // 1 thread, queue of 1

Runnable slow = () -> { try { Thread.sleep(1000); } catch (InterruptedException e) { } };
ex.execute(slow);    // runs
ex.execute(slow);    // waits in the queue
ex.execute(slow);    // RejectedExecutionException: pool busy and queue full
```

### Q33. What is the difference between `submit()` and `execute()`?
**Answer:**
- **`execute()`**: use it when we just want to run a task and don't care about a result. It takes a `Runnable` and returns nothing. If something goes wrong inside the task, the exception is thrown directly on the worker thread and usually ends up in the logs via the thread's exception handler.
- **`submit()`**: use it when we want more control. It returns a `Future` that lets us track whether the task is finished, cancel it, or get a result. If the task throws an exception, it does not crash the thread immediately: the exception is **stored** and thrown when we call `future.get()`. Because of this `submit()` is safer and more flexible, and is generally preferred in real applications.
```java
ExecutorService ex = Executors.newFixedThreadPool(1);

ex.execute(() -> { throw new RuntimeException("printed by the thread's handler"); });

Future<?> f = ex.submit(() -> { throw new RuntimeException("stored in the Future"); });
try {
    f.get();                                       // exception is thrown here
} catch (ExecutionException e) {
    System.out.println(e.getCause().getMessage()); // stored in the Future
}
ex.shutdown();
```

### Q34. What is the difference between `Runnable` and `Callable` in executors?
**Answer:** Threads and executors manage threads. `Runnable` and `Callable` represent the **task** that a thread will execute, meaning the code that runs inside a thread.
- **`Runnable`**: does not return a result, cannot throw a checked exception, used for tasks that just perform work.
- **`Callable`**: returns a result, can throw checked exceptions, used when we need an outcome from the task.
```java
Runnable r = () -> System.out.println("does work, returns nothing");

Callable<Integer> c = () -> {
    Thread.sleep(100);            // checked exception allowed
    return 42;                    // returns a result
};

ExecutorService ex = Executors.newSingleThreadExecutor();
Future<Integer> result = ex.submit(c);
System.out.println(result.get()); // 42
ex.shutdown();
```

### Q35. Explain the types of thread executors.
**Answer: Five types.**
1. **Fixed thread pool:** we define how many threads work. With `newFixedThreadPool(4)` at most four threads run in parallel, and extra tasks wait in a queue until a thread is free.
2. **Single thread executor:** exactly one thread. Only one task runs at a time and the rest wait in a queue and execute one by one (sequential execution).
3. **Cached thread pool:** no fixed number of threads. New threads are created when needed, idle threads are reused, and threads idle for some time are destroyed. It is a dynamic pool, but if too many tasks come it may create too many threads and cause performance or memory issues.
4. **Scheduled thread pool:** runs tasks after a delay or repeatedly, such as background jobs or periodic checks.
5. **Fork/Join pool:** used for parallel processing. A big task is divided into smaller tasks that run in parallel, and idle threads can steal work from busy threads. Useful for CPU-intensive tasks and divide-and-conquer problems.
```java
ExecutorService fixed  = Executors.newFixedThreadPool(4);
ExecutorService single = Executors.newSingleThreadExecutor();
ExecutorService cached = Executors.newCachedThreadPool();

ScheduledExecutorService sched = Executors.newScheduledThreadPool(2);
sched.schedule(() -> System.out.println("after 5 s"), 5, TimeUnit.SECONDS);
sched.scheduleAtFixedRate(() -> System.out.println("every 10 s"), 0, 10, TimeUnit.SECONDS);

ForkJoinPool fj = new ForkJoinPool();
```
*(added: `newFixedThreadPool` uses an unbounded queue, which can use a lot of memory if tasks pile up, and `newCachedThreadPool` has no upper limit on threads. For production, a configured `ThreadPoolExecutor` is often safer.)*

---

## 9. CompletableFuture (Q36-Q40)

### Q36. What is `CompletableFuture` and how does it work?
**Answer:** It is used to run tasks **asynchronously** and chain multiple steps **without blocking**. We start a task, and while it runs our main thread is free. When the task finishes, we can say "then do this, then do that". Example: order food, the food gets prepared, then delivered, then the customer is notified. Each step runs automatically after the previous one finishes.
```java
CompletableFuture.supplyAsync(() -> "order placed")
        .thenApply(s -> s + " -> prepared")
        .thenApply(s -> s + " -> delivered")
        .thenAccept(s -> System.out.println(s + " -> customer notified"))
        .join();    // only here so the demo waits for the result
```

### Q37. How do you handle exceptions in `CompletableFuture`?
**Answer:** It has built-in exception handling methods:
- **`exceptionally()`** gives a fallback value.
- **`handle()`** handles success or failure together.
- **`whenComplete()`** just logs or observes errors.

The idea: if something fails, we decide what to do next instead of crashing the program.
```java
CompletableFuture<Integer> f = CompletableFuture.supplyAsync(() -> 10 / 0);

int a = f.exceptionally(ex -> -1).join();                              // -1 (fallback value)
int b = f.handle((value, ex) -> ex == null ? value : -1).join();       // -1 (success or failure)
f.whenComplete((value, ex) -> {                                        // observe only
    if (ex != null) System.out.println("failed: " + ex);
});
```

### Q38. Can `CompletableFuture` cause thread starvation?
**Answer: Yes, if used incorrectly.** By default `CompletableFuture` uses the `ForkJoinPool.commonPool()`. If tasks are blocking, or too many long tasks run, the threads get stuck and new tasks wait forever. This is called starvation. The solution is to use a **custom executor** for blocking tasks.
```java
// Default: runs on ForkJoinPool.commonPool(), risky for blocking calls
CompletableFuture.supplyAsync(() -> blockingCall());

// Fix: dedicated executor for blocking work
ExecutorService io = Executors.newFixedThreadPool(20);
CompletableFuture.supplyAsync(() -> blockingCall(), io);
```

### Q39. Why is `CompletableFuture` better than `Future`?
**Answer:**

| `Future` | `CompletableFuture` |
|---|---|
| We must call `get()` (blocks) | Non-blocking task chaining |
| No chaining | Chain steps with `thenApply`, `thenAccept`, `thenCompose` |
| Poor exception handling | Better exception handling (`exceptionally`, `handle`, `whenComplete`) |
| Hard to combine tasks | Combine multiple tasks easily |

```java
Future<Integer> f = executor.submit(() -> 10);
int v = f.get();                                   // blocks the calling thread

CompletableFuture.supplyAsync(() -> 10)
        .thenApply(x -> x * 2)                     // no blocking, chained
        .thenAccept(System.out::println);
```

### Q40. What are the common methods in `CompletableFuture`?
**Answer:**
- **Create async tasks:** `supplyAsync()` when we need a result, `runAsync()` when no result is needed.
- **Chain next actions:** `thenApply()` transforms the result, `thenAccept()` uses the result, `thenRun()` runs another task.
- **Work with multiple tasks:** `thenCombine()`, `thenCompose()`, `allOf()`, `anyOf()`.
- **Exception handling:** `exceptionally()`, `handle()`, `whenComplete()`.
- **Block and get the result:** `get()` or `join()`.
- **Control methods:** `complete()`, `cancel()`, `isDone()` to manually manage or check the task state.
```java
CompletableFuture<Integer> a = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> b = CompletableFuture.supplyAsync(() -> 20);

a.thenCombine(b, Integer::sum).thenAccept(System.out::println);        // 30
a.thenCompose(x -> CompletableFuture.supplyAsync(() -> x * 2));        // dependent async step
CompletableFuture.allOf(a, b).join();                                  // wait for all
CompletableFuture.anyOf(a, b).join();                                  // wait for the first
CompletableFuture.runAsync(() -> System.out.println("no result"));
```

---

## 10. ForkJoinPool (Q41)

### Q41. What is `ForkJoinPool` and what is a real example?
**Answer:** It is a special thread pool designed for **divide and conquer** tasks. A big task is **split** into smaller tasks (fork), and the results are **combined** (join). Idle threads can **steal work** from busy threads. Real examples: searching a big array, parallel sorting, processing large data in chunks.
```java
class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10_000;
    private final int[] arr;
    private final int lo, hi;

    SumTask(int[] arr, int lo, int hi) { this.arr = arr; this.lo = lo; this.hi = hi; }

    @Override
    protected Long compute() {
        if (hi - lo <= THRESHOLD) {                 // small enough: compute directly
            long sum = 0;
            for (int i = lo; i < hi; i++) sum += arr[i];
            return sum;
        }
        int mid = (lo + hi) >>> 1;
        SumTask left  = new SumTask(arr, lo, mid);
        SumTask right = new SumTask(arr, mid, hi);
        left.fork();                                // fork: run the left half asynchronously
        return right.compute() + left.join();       // compute right here, then join the left result
    }
}

long total = new ForkJoinPool().invoke(new SumTask(arr, 0, arr.length));
```

---

## Source

- A video transcript on Java multithreading interview questions. The transcript does not name the video or channel.
- The 82 "Can we...?" questions from the other video are in [README.md](README.md).
