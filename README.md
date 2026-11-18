# Java Interview Questions: 100 Tricky Questions with Answers

A study guide of tricky Java interview questions, grouped by the pattern each one tests.

> **Source note:** This is inspired by the topic of the video *"82 Tricky Java Interview Questions | Most Candidates Get These Wrong"* (GenZ Career, https://youtu.be/glsIlGLglzQ). It is **not** a transcript of that video. The video's transcript was not accessible, so the questions and answers below are my own compilation of the classic tricky Java topics. Verify against the video if you need an exact match.

## Contents

1. [Strings](#1-strings-q1-q10)
2. [Primitives, Wrappers and Numbers](#2-primitives-wrappers-and-numbers-q11-q25)
3. [OOP, Initialization and Overloading](#3-oop-initialization-and-overloading-q26-q42)
4. [Exceptions and finally](#4-exceptions-and-finally-q43-q50)
5. [Collections](#5-collections-q51-q66)
6. [Generics](#6-generics-q67-q70)
7. [Concurrency](#7-concurrency-q71-q84)
8. [JVM, Memory and Class Loading](#8-jvm-memory-and-class-loading-q85-q90)
9. [Modern Java (8 to 21)](#9-modern-java-8-to-21-q91-q100)

---

## 1. Strings (Q1-Q10)

### Q1. What does this print?
```java
String s1 = "hello";
String s2 = "hello";
String s3 = new String("hello");
System.out.println(s1 == s2);        // ?
System.out.println(s1 == s3);        // ?
System.out.println(s1.equals(s3));   // ?
System.out.println(s1 == s3.intern());// ?
```
**Answer:** `true`, `false`, `true`, `true`. Literals share one object in the string pool. `new String(...)` always creates a separate heap object. `equals` compares content. `intern()` returns the pooled instance.

### Q2. How many objects does `new String("abc")` create?
**Answer:** Up to two: the literal `"abc"` in the string pool (if not already there) and a new `String` object on the heap.

### Q3. What does this print, and why?
```java
String a = "a";
String c1 = "a" + "b";
String c2 = a + "b";
final String f = "a";
String c3 = f + "b";
System.out.println(c1 == "ab"); // true
System.out.println(c2 == "ab"); // false
System.out.println(c3 == "ab"); // true
```
**Answer:** Concatenation of compile-time constants is folded by the compiler into the literal `"ab"`. `a` is not final, so `a + "b"` is computed at runtime and creates a new object. A `final` variable initialized with a constant is itself a constant, so `c3` is folded.

### Q4. Why is `String` immutable?
**Answer:** String pool sharing is only safe if strings cannot change. Immutability also makes `hashCode` cacheable (good for `HashMap` keys), makes strings thread-safe, and protects security-sensitive values such as class names, file paths and URLs.

### Q5. `String` vs `StringBuilder` vs `StringBuffer`?
**Answer:** `String` is immutable. `StringBuilder` is mutable and not thread-safe (fast). `StringBuffer` is mutable and synchronized (slower, rarely needed). Use `StringBuilder` for building strings in loops.

### Q6. What does this print?
```java
String s = "java";
s.concat(" rocks");
s.toUpperCase();
System.out.println(s);
```
**Answer:** `java`. String methods return new strings. The results were discarded.

### Q7. What happens when you `switch` on a `null` String?
**Answer:** `NullPointerException`. The switch calls `hashCode()`/`equals()` on the value. Check for null first.

### Q8. Can two different strings have the same `hashCode`?
**Answer:** Yes. `"Aa".hashCode() == "BB".hashCode()` (both 2112). Equal hash codes do not imply equal objects, which is why `equals` is still needed.

### Q9. What do these print?
```java
System.out.println('a' + 'b');          // 195
System.out.println("" + 'a' + 'b');     // ab
System.out.println('a' + 'b' + "");     // 195
System.out.println(1 + 2 + "3" + 4 + 5);// 3345
System.out.println((char) ('a' + 1));   // b
```
**Answer:** `char + char` is numeric addition (int). Once a `String` is involved, `+` becomes concatenation, evaluated left to right.

### Q10. Why is `char[]` preferred over `String` for passwords?
**Answer:** A `String` stays in memory (possibly in the pool) until garbage collected and cannot be wiped. A `char[]` can be overwritten with zeros right after use.

---

## 2. Primitives, Wrappers and Numbers (Q11-Q25)

### Q11. What does this print?
```java
Integer a = 127, b = 127;
Integer c = 128, d = 128;
System.out.println(a == b); // true
System.out.println(c == d); // false
```
**Answer:** `Integer.valueOf` caches values from -128 to 127. Outside that range, boxing creates new objects, so `==` compares different references. Always use `equals` for wrappers.

### Q12. Is `0.1 + 0.2 == 0.3`?
**Answer:** No. `0.1 + 0.2` is `0.30000000000000004` because binary floating point cannot represent these decimals exactly. Use `BigDecimal` (constructed from strings) for money, or compare with a tolerance.

### Q13. What does `Integer.MAX_VALUE + 1` give? What about `Math.abs(Integer.MIN_VALUE)`?
**Answer:** `Integer.MIN_VALUE` (overflow wraps silently). `Math.abs(Integer.MIN_VALUE)` is still `-2147483648` because +2147483648 does not fit in an int. Use `Math.addExact` to detect overflow.

### Q14. What is the result of `5 / 2`, `5 / 2.0`, `(double) (5 / 2)`?
**Answer:** `2`, `2.5`, `2.0`. Integer division truncates before the cast happens.

### Q15. Will this compile?
```java
byte b = 10;
b = b + 1;   // ?
b += 1;      // ?
```
**Answer:** `b = b + 1` fails: `b + 1` is an `int`. `b += 1` compiles because compound assignment includes an implicit narrowing cast.

### Q16. What do these division results print?
```java
System.out.println(1.0 / 0);   // Infinity
System.out.println(0.0 / 0);   // NaN
System.out.println(1 / 0);     // ArithmeticException
```
**Answer:** Floating-point division by zero yields `Infinity` or `NaN`. Integer division by zero throws `ArithmeticException`.

### Q17. Is `Double.NaN == Double.NaN`?
**Answer:** `false` for the primitive comparison. `Double.valueOf(Double.NaN).equals(Double.NaN)` is `true`. Use `Double.isNaN(x)` to test.

### Q18. What is the bug here?
```java
long micros = 24 * 60 * 60 * 1000 * 1000;
System.out.println(micros); // 500654080
```
**Answer:** The multiplication happens in `int` and overflows before widening to `long`. Fix: `24L * 60 * 60 * 1000 * 1000`.

### Q19. What does `i = i++` do?
```java
int i = 5;
i = i++;
System.out.println(i); // 5
int j = 5;
j = j++ + ++j;
System.out.println(j); // 12
```
**Answer:** `i++` returns the old value (5), increments, then the assignment overwrites `i` with 5. For `j`: `j++` gives 5 (j becomes 6), `++j` gives 7, sum is 12.

### Q20. What does this print?
```java
Object o = true ? 1 : 2.0;
System.out.println(o); // 1.0
```
**Answer:** `1.0`. In a ternary with numeric operands, binary numeric promotion converts both branches to `double`.

### Q21. Why can this throw `NullPointerException`?
```java
Integer a = null;
int b = true ? a : 0; // NPE
```
**Answer:** With one `Integer` and one `int` operand, the ternary's type is `int`, so `a` is auto-unboxed, and unboxing `null` throws NPE.

### Q22. What does this print?
```java
System.out.println(Long.valueOf(1).equals(1));   // false
System.out.println(Long.valueOf(1).equals(1L));  // true
```
**Answer:** `1` is boxed to `Integer`, and `Long.equals` returns false for a non-`Long`.

### Q23. `short s = 1; s = s + 1;` vs `s += 1;`?
**Answer:** Same as `byte`: the first doesn't compile (int result), the second does (implicit cast).

### Q24. Which overload is called?
```java
static void m(long x)    { System.out.println("long"); }
static void m(Integer x) { System.out.println("Integer"); }
m(5); // long
```
**Answer:** `long`. Overload resolution order: (1) exact/widening, (2) boxing, (3) varargs. Widening `int` to `long` wins over boxing.

### Q25. Which overload is called for `null`?
```java
static void m(Object o) { System.out.println("Object"); }
static void m(String s) { System.out.println("String"); }
m(null); // String
```
**Answer:** `String`, the most specific type. If the overloads were `m(String)` and `m(StringBuilder)`, the call would be ambiguous and fail to compile.

---

## 3. OOP, Initialization and Overloading (Q26-Q42)

### Q26. Overloading vs overriding?
**Answer:** Overloading: same name, different parameters, resolved at compile time by the declared types. Overriding: subclass replaces an instance method with the same signature, resolved at runtime by the actual object type.

### Q27. Can you override a static method? What about fields?
**Answer:** No. Static methods are hidden, not overridden, and fields are never polymorphic.
```java
class P { String name = "P"; static String who() { return "P"; } }
class C extends P { String name = "C"; static String who() { return "C"; } }
P p = new C();
System.out.println(p.name);  // P
System.out.println(p.who()); // P
```

### Q28. What is the initialization order with inheritance?
```java
class Parent {
    static { System.out.println("Parent static"); }
    { System.out.println("Parent instance"); }
    Parent() { System.out.println("Parent ctor"); }
}
class Child extends Parent {
    static { System.out.println("Child static"); }
    { System.out.println("Child instance"); }
    Child() { System.out.println("Child ctor"); }
}
new Child();
```
**Answer:** Parent static, Child static, Parent instance, Parent ctor, Child instance, Child ctor. Static blocks run once at class initialization (parent first). Then for each object: parent init blocks and constructor, then child's.

### Q29. What is the danger of calling an overridable method from a constructor?
```java
class A { A() { show(); } void show() {} }
class B extends A {
    String s = "hi";
    void show() { System.out.println(s); } // prints null
}
new B();
```
**Answer:** The parent constructor runs before the child's fields are initialized, so the overridden method sees default values (`null`/`0`).

### Q30. Can a constructor be private? Why?
**Answer:** Yes. Used for singletons, utility classes, static factory methods and preventing subclassing.

### Q31. Can you override a private method?
**Answer:** No. Private methods are not inherited. A subclass method with the same name is a brand new method.

### Q32. Abstract class vs interface?
**Answer:** Abstract classes can hold state and constructors and allow single inheritance. Interfaces define a contract, allow multiple inheritance of type, and since Java 8 can have `default`/`static` methods (and `private` ones since Java 9). Prefer interfaces for capabilities and abstract classes for shared state/behavior.

### Q33. What is the diamond problem and how does Java handle default methods?
```java
interface A { default void hi() { System.out.println("A"); } }
interface B { default void hi() { System.out.println("B"); } }
class C implements A, B {
    public void hi() { A.super.hi(); } // must resolve explicitly
}
```
**Answer:** If two interfaces provide the same default method, the class must override it and may delegate with `Interface.super.method()`. A concrete class method always wins over an interface default.

### Q34. Can you instantiate an abstract class?
**Answer:** No, but you can create an anonymous subclass: `new AbstractFoo() { ... }`.

### Q35. What rules apply when overriding a method?
**Answer:** Same signature; return type must be the same or a covariant subtype; visibility cannot be reduced; it cannot throw broader checked exceptions; `final`/`static`/`private` methods cannot be overridden.

### Q36. What is a covariant return type?
**Answer:** An overriding method may return a subtype of the parent's return type, e.g. parent returns `Object`, child returns `String`.

### Q37. What does `null instanceof Object` return?
**Answer:** `false`. `instanceof` is always false for `null`.

### Q38. What is the `equals`/`hashCode` contract?
**Answer:** Equal objects must have equal hash codes. `equals` must be reflexive, symmetric, transitive, consistent, and `x.equals(null)` is false. Override both together or hash-based collections break.

### Q39. Shallow vs deep copy with `clone()`?
**Answer:** `Object.clone()` copies fields; references are shared (shallow). A deep copy also duplicates the referenced objects. Prefer copy constructors or factory methods over `Cloneable`.

### Q40. Static nested vs inner class? Where is the memory-leak risk?
**Answer:** A static nested class has no reference to the outer instance. An inner (non-static) class holds an implicit reference to the enclosing object, which can keep it alive (a classic leak with long-lived listeners or threads).

### Q41. `final` vs `finally` vs `finalize`?
**Answer:** `final`: constant/non-overridable/non-extendable. `finally`: block that runs after try/catch. `finalize()`: deprecated for removal (Java 18); use try-with-resources or `Cleaner` instead.

### Q42. Is a `final` reference variable immutable? How do you make an immutable class?
**Answer:** `final` only prevents reassigning the reference. The object can still change. For an immutable class: make the class `final`, fields `private final`, no setters, defensively copy mutable inputs and outputs. Or use a `record`.

---

## 4. Exceptions and finally (Q43-Q50)

### Q43. What does this return?
```java
static int test() {
    try { return 1; }
    finally { return 2; }
}
```
**Answer:** `2`. A `return` in `finally` overrides the earlier return and even swallows exceptions. Never return from `finally`.

### Q44. When does `finally` not run?
**Answer:** When the JVM exits (`System.exit`, `Runtime.halt`), the process is killed, or the thread is stuck in an infinite loop or deadlock before reaching it.

### Q45. How does try-with-resources work?
**Answer:** Resources implementing `AutoCloseable` are closed automatically in **reverse order** of declaration. If both the body and `close()` throw, the body's exception is primary and the other is attached as a *suppressed* exception (`getSuppressed()`).

### Q46. Checked vs unchecked exceptions?
**Answer:** Checked (extend `Exception` but not `RuntimeException`) must be caught or declared. Unchecked (`RuntimeException`, `Error`) need not be.

### Q47. Does catch order matter?
```java
try { /* ... */ }
catch (Exception e) { }
catch (IOException e) { } // compile error: already caught
```
**Answer:** Yes. Catch the more specific subclass first, otherwise the compiler reports it as unreachable.

### Q48. What happens if a static initializer throws?
**Answer:** The first access throws `ExceptionInInitializerError`. Later accesses to the class throw `NoClassDefFoundError`.

### Q49. Can you catch `StackOverflowError` or `OutOfMemoryError`?
**Answer:** Technically yes (they are `Throwable`s), but recovery is unreliable. Fix the cause (runaway recursion, leaks) rather than catching.

### Q50. What are the rules of multi-catch?
```java
try { /* ... */ }
catch (IOException | SQLException e) { }
```
**Answer:** The alternatives cannot be subclasses of each other, and the catch parameter is implicitly `final`.

---

## 5. Collections (Q51-Q66)

### Q51. How does `HashMap` work internally?
**Answer:** An array of buckets. The index is derived from the key's (spread) hash, `(n - 1) & hash`. Collisions form a linked list, converted to a red-black tree when a bucket reaches 8 entries and the table has at least 64 buckets (otherwise the table resizes). Reverts to a list at 6. Resizes (doubles) when size exceeds capacity x 0.75.

### Q52. Why is table capacity a power of two?
**Answer:** So the index can be computed with a fast bitmask `(n - 1) & hash` instead of modulo, and resizing splits buckets cleanly.

### Q53. What happens if you mutate a key after putting it in a `HashMap`/`HashSet`?
**Answer:** Its hash changes, so lookups probe the wrong bucket and the entry becomes effectively unreachable. Use immutable keys.

### Q54. Which maps allow `null`?
**Answer:** `HashMap`: one null key and null values. `LinkedHashMap`: same. `TreeMap`: no null key (natural ordering). `Hashtable` and `ConcurrentHashMap`: no null keys or values.

### Q55. `HashMap` vs `Hashtable` vs `ConcurrentHashMap` vs `synchronizedMap`?
**Answer:** `HashMap`: not thread-safe. `Hashtable`: legacy, locks the whole table. `synchronizedMap`: wraps with one lock. `ConcurrentHashMap`: fine-grained, high-concurrency, atomic `compute`/`merge`/`putIfAbsent`. Prefer `ConcurrentHashMap`.

### Q56. Why does this throw, and when doesn't it?
```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4));
for (Integer x : list) {
    if (x == 2) list.remove(x); // ConcurrentModificationException
}
```
**Answer:** Structural modification during iteration triggers the fail-fast `ConcurrentModificationException`. Quirk: removing the *second-to-last* element does not throw, because `hasNext()` returns false. Use `Iterator.remove()` or `list.removeIf(...)`.

### Q57. What is special about `Arrays.asList` and `List.of`?
```java
List<String> a = Arrays.asList("x", "y");
a.set(0, "z");  // OK, writes through to the array
a.add("w");     // UnsupportedOperationException
List<String> b = List.of("x", "y");
b.set(0, "z");  // UnsupportedOperationException
List.of("x", null); // NullPointerException
```
**Answer:** `Arrays.asList` is fixed-size and backed by the array. `List.of` is fully immutable and disallows nulls.

### Q58. What does `Arrays.asList(new int[]{1,2,3}).size()` return?
**Answer:** `1`. Generics don't work with primitives, so the whole `int[]` is a single element of a `List<int[]>`. Use `Integer[]` or `IntStream`.

### Q59. `ArrayList` vs `LinkedList`?
**Answer:** `ArrayList`: O(1) random access, amortized O(1) append, O(n) middle insert. `LinkedList`: O(1) insert at known node, but O(n) access and poor cache locality. `ArrayList` is usually faster in practice.

### Q60. `list.remove(1)` on a `List<Integer>`?
```java
List<Integer> l = new ArrayList<>(List.of(10, 1, 20));
l.remove(1);                    // removes index 1 -> [10, 20]
l.remove(Integer.valueOf(1));   // removes the value 1
```
**Answer:** `remove(int)` takes an index, `remove(Object)` takes a value. The primitive overload is chosen for an `int` argument.

### Q61. `Comparable` vs `Comparator`?
**Answer:** `Comparable` defines a class's natural order (`compareTo`, one per class). `Comparator` defines external, multiple orderings (`Comparator.comparing(...).thenComparing(...)`).

### Q62. How does `HashSet` ensure uniqueness? And `TreeSet`?
**Answer:** `HashSet` is backed by a `HashMap` (uses `hashCode`/`equals`). `TreeSet` uses `compareTo`/`Comparator` only: two elements comparing as 0 are considered duplicates even if `equals` says otherwise.

### Q63. How do you build an LRU cache with the JDK?
```java
class LRU<K, V> extends LinkedHashMap<K, V> {
    private final int cap;
    LRU(int cap) { super(16, 0.75f, true); this.cap = cap; }
    @Override protected boolean removeEldestEntry(Map.Entry<K, V> e) {
        return size() > cap;
    }
}
```
**Answer:** `LinkedHashMap` with `accessOrder = true` plus `removeEldestEntry`.

### Q64. Is `PriorityQueue` iteration sorted?
**Answer:** No. It's a binary heap; only `poll()`/`peek()` give the smallest element. Printing or iterating shows heap order.

### Q65. `Iterator` vs `ListIterator` vs `Enumeration`?
**Answer:** `Iterator`: forward, supports remove. `ListIterator`: bidirectional, supports add/set. `Enumeration`: legacy, read-only.

### Q66. `Collections.unmodifiableList` vs `List.copyOf`?
**Answer:** `unmodifiableList` is a read-only *view*: changes to the underlying list show through. `List.copyOf` makes an independent immutable copy.

---

## 6. Generics (Q67-Q70)

### Q67. What is type erasure? What does this print?
```java
System.out.println(new ArrayList<String>().getClass()
                == new ArrayList<Integer>().getClass()); // true
```
**Answer:** Generic type info is erased at compile time (replaced by bounds or `Object`). At runtime both are plain `ArrayList`. It's why `m(List<String>)` and `m(List<Integer>)` clash (same erasure), and why you can't do `new T()` or `instanceof List<String>`.

### Q68. Is `List<Object>` a supertype of `List<String>`?
**Answer:** No. Generics are invariant. Use wildcards: `List<? extends Number>` to read (producer), `List<? super Integer>` to write (consumer). Rule: **PECS** (Producer Extends, Consumer Super).

### Q69. Why can't you create `new T[10]` or `new List<String>[10]`?
**Answer:** Arrays are reified and covariant while generics are erased, so the runtime couldn't enforce type safety. Use `List<List<String>>` or `(T[]) new Object[10]` with care.

### Q70. What is heap pollution?
**Answer:** A variable of a parameterized type refers to an object of a different type, typically through raw types or unchecked casts. It surfaces later as `ClassCastException` far from the cause.

---

## 7. Concurrency (Q71-Q84)

### Q71. `start()` vs `run()`? What if you call `start()` twice?
**Answer:** `start()` creates a new thread that executes `run()`. Calling `run()` directly just executes on the current thread. A second `start()` throws `IllegalThreadStateException`.

### Q72. What does `synchronized` lock on?
**Answer:** Instance method: `this`. Static method: the `Class` object. Block: the given object. Static and instance synchronized methods use different locks and do not block each other.

### Q73. What does `volatile` guarantee? Is `count++` safe on a volatile int?
**Answer:** It guarantees visibility and ordering (happens-before) but **not atomicity**. `count++` is read-modify-write, so use `AtomicInteger` or `synchronized`.

### Q74. Why must `wait()`/`notify()` be called inside `synchronized`? Why use a `while` loop?
**Answer:** They operate on the object's monitor, so the thread must own it (else `IllegalMonitorStateException`). Waiting must be in a `while (!condition)` loop to handle spurious wakeups and missed conditions.

### Q75. `sleep()` vs `wait()`?
**Answer:** `sleep` pauses the thread and keeps any locks it holds. `wait` releases the monitor and waits for `notify`/`notifyAll` (or timeout).

### Q76. What are the four conditions for deadlock? How do you prevent it?
**Answer:** Mutual exclusion, hold and wait, no preemption, circular wait. Prevent by acquiring locks in a consistent global order, using `tryLock` with timeouts, or reducing lock scope.

### Q77. Why use `ExecutorService` instead of `new Thread()`? Pitfalls?
**Answer:** Thread reuse, bounded concurrency, queueing, lifecycle control. Pitfalls: `newFixedThreadPool` uses an unbounded queue (memory risk), `newCachedThreadPool` can create unbounded threads. Prefer a configured `ThreadPoolExecutor`. Always `shutdown()`.

### Q78. `shutdown()` vs `shutdownNow()`?
**Answer:** `shutdown()` stops accepting tasks but finishes queued ones. `shutdownNow()` attempts to interrupt running tasks and returns the list of tasks that never started.

### Q79. Is `map.get` then `map.put` safe on a `ConcurrentHashMap`?
```java
if (!map.containsKey(k)) map.put(k, v);   // race condition
map.putIfAbsent(k, v);                    // atomic
map.merge(k, 1, Integer::sum);            // atomic counter
```
**Answer:** No, the two calls are not atomic together. Use the atomic compound methods.

### Q80. What is `ThreadLocal` and what is its pitfall?
**Answer:** Per-thread variable storage. In thread pools, threads live on, so values leak or leak between requests unless you call `remove()` in a `finally`.

### Q81. `Runnable` vs `Callable`? What does `Future.get()` do?
**Answer:** `Callable` returns a value and can throw checked exceptions. `Future.get()` blocks until the result is ready. Prefer `CompletableFuture` for non-blocking composition.

### Q82. How do atomic classes work? What is the ABA problem?
**Answer:** They use hardware compare-and-swap (CAS) in a retry loop (lock-free). ABA: a value changes A to B back to A, so CAS thinks nothing changed. `AtomicStampedReference` adds a version stamp.

### Q83. Why does double-checked locking need `volatile`?
```java
class Singleton {
    private static volatile Singleton inst;
    static Singleton get() {
        if (inst == null) {
            synchronized (Singleton.class) {
                if (inst == null) inst = new Singleton();
            }
        }
        return inst;
    }
}
```
**Answer:** Without `volatile`, instruction reordering can publish a reference to a partially constructed object. (Alternatives: enum singleton or holder class idiom.)

### Q84. When does the JVM exit? What are virtual threads?
**Answer:** The JVM exits when all **non-daemon** threads finish (daemon threads are killed). Virtual threads (final in Java 21) are lightweight JVM-managed threads, ideal for large numbers of blocking I/O tasks. Avoid pooling them and be careful with `synchronized` pinning in older versions.

---

## 8. JVM, Memory and Class Loading (Q85-Q90)

### Q85. Is Java pass-by-value or pass-by-reference?
```java
static void change(StringBuilder sb, int n) { sb.append("!"); n = 99; sb = null; }
StringBuilder s = new StringBuilder("hi"); int x = 1;
change(s, x);
System.out.println(s + " " + x); // hi! 1
```
**Answer:** Always pass-by-value. For objects, the *reference* is copied: you can mutate the object through it, but reassigning the parameter doesn't affect the caller. A `swap(a, b)` method therefore cannot work.

### Q86. Explain JVM memory areas.
**Answer:** Heap (objects, shared, GC-managed), thread stacks (frames, locals), Metaspace (class metadata, native memory, replaced PermGen in Java 8), PC registers, native method stacks, code cache (JIT output).

### Q87. `StackOverflowError` vs `OutOfMemoryError`?
**Answer:** `StackOverflowError`: a thread's stack is exhausted (usually infinite recursion). `OutOfMemoryError`: heap (or Metaspace/native/direct memory) is exhausted.

### Q88. How does garbage collection decide what to free? Does `System.gc()` force it?
**Answer:** Objects unreachable from GC roots (thread stacks, static fields, JNI refs) are collectable. `System.gc()` is only a hint. Reference types: strong, soft (cleared under memory pressure), weak (cleared at next GC), phantom (post-mortem cleanup).

### Q89. Can Java have memory leaks?
**Answer:** Yes: objects that are reachable but no longer used, such as growing static collections, unremoved listeners, unclosed resources, `ThreadLocal`s in pools, and inner classes holding outer references.

### Q90. Explain class loading and parent delegation.
**Answer:** Loaders: Bootstrap, Platform, Application (plus custom). A loader first delegates to its parent and only loads the class itself if the parent can't. This prevents user code from replacing core classes like `java.lang.String`. Classes are initialized lazily on first active use (new, static access, etc.).

---

## 9. Modern Java (8 to 21) (Q91-Q100)

### Q91. Why must lambda-captured locals be effectively final?
**Answer:** Lambdas capture the *value* of locals (which may outlive the stack frame/thread), so the variable must not change. Instance and static fields can be modified.

### Q92. Why does reusing a stream fail?
```java
Stream<String> s = Stream.of("a", "b");
s.forEach(System.out::println);
s.count(); // IllegalStateException: stream has already been operated upon or closed
```
**Answer:** A stream is single-use. Streams are also lazy: nothing runs until a terminal operation.

### Q93. `map` vs `flatMap`?
**Answer:** `map` transforms each element 1:1. `flatMap` transforms each element into a stream and flattens the results into one stream (e.g. `List<List<X>>` to `Stream<X>`).

### Q94. `Optional.orElse` vs `orElseGet`?
```java
opt.orElse(expensive());     // expensive() ALWAYS runs
opt.orElseGet(() -> expensive()); // runs only if empty
```
**Answer:** `orElse` evaluates its argument eagerly. Also: `Optional.of(null)` throws NPE (use `ofNullable`), and don't use `Optional` for fields or parameters.

### Q95. What does `Collectors.toMap` do on duplicate keys?
**Answer:** Throws `IllegalStateException` (Duplicate key). Pass a merge function: `toMap(k, v, (a, b) -> a)`.

### Q96. `Stream.toList()` vs `Collectors.toList()`?
**Answer:** `Stream.toList()` (Java 16) returns an unmodifiable list and allows nulls. `Collectors.toList()` makes no guarantee about mutability.

### Q97. What are the pitfalls of parallel streams?
**Answer:** They use the shared `ForkJoinPool.commonPool()`, so blocking tasks starve other work. They need stateless, non-interfering, associative operations. They are slower for small data or I/O-bound work. Never mutate shared state from them.

### Q98. What is a functional interface?
**Answer:** An interface with exactly one abstract method (default/static methods don't count), optionally annotated `@FunctionalInterface`. Lambdas and method references target it (`Function`, `Predicate`, `Supplier`, `Consumer`, `Runnable`).

### Q99. What are records and sealed classes?
```java
record Point(int x, int y) {}                      // Java 16
sealed interface Shape permits Circle, Square {}   // Java 17
```
**Answer:** A record is a concise immutable data carrier with auto-generated constructor, accessors, `equals`, `hashCode`, `toString`. A sealed type restricts which classes may extend it, enabling exhaustive pattern-matching `switch`.

### Q100. What does pattern matching for `switch` look like (Java 21)?
```java
static String describe(Object o) {
    return switch (o) {
        case Integer i when i > 10 -> "big int " + i;
        case Integer i             -> "int " + i;
        case String s              -> "string " + s.length();
        case null                  -> "null";
        default                    -> "other";
    };
}
```
**Answer:** `switch` can match on types with guards (`when`), handle `null` explicitly, and is checked for exhaustiveness (especially with sealed types).

---

## Using this as a repo

```bash
git init
git add README.md
git commit -m "Add 100 tricky Java interview questions"
git branch -M main
git remote add origin https://github.com/<your-username>/java-interview-questions.git
git push -u origin main
```
