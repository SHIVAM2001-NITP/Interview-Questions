# 82 Tricky Java Interview Questions ("Can we...?") with Answers and Code

Every question from the video **"82 Tricky Java Interview Questions | Most Candidates Get These Wrong"** by GenZ Career (https://youtu.be/glsIlGLglzQ), in the video's order (Q1 to Q82), grouped by the pattern each one tests.

How this file was made:
- The answers follow the explanations given in the video (taken from its auto-generated transcript, with transcription slips fixed, such as "abset" for abstract and "surprised exception" for suppressed exception).
- The code examples are added here so each answer can be run and checked. Lines marked *(added)* are extra notes that are not in the video.
- Some answers depend on the Java version (Java 7, 8, 9, 25, 26). The version is stated where it matters.

## Contents

1. [main() and source files (Q1-Q5)](#1-main-and-source-files-q1-q5)
2. [Inheritance, interfaces and abstract classes (Q6-Q16)](#2-inheritance-interfaces-and-abstract-classes-q6-q16)
3. [Constructors (Q17-Q21)](#3-constructors-q17-q21)
4. [Enums (Q22-Q24)](#4-enums-q22-q24)
5. [Strings (Q25-Q29)](#5-strings-q25-q29)
6. [Immutability (Q30-Q32)](#6-immutability-q30-q32)
7. [Exceptions and finally (Q33-Q38)](#7-exceptions-and-finally-q33-q38)
8. [Collections (Q39-Q48)](#8-collections-q39-q48)
9. [Generics (Q49-Q51)](#9-generics-q49-q51)
10. [Threads and synchronization (Q52-Q62)](#10-threads-and-synchronization-q52-q62)
11. [JVM, garbage collection and memory (Q63-Q67)](#11-jvm-garbage-collection-and-memory-q63-q67)
12. [Lambdas, streams, Optional, records and var (Q68-Q75)](#12-lambdas-streams-optional-records-and-var-q68-q75)
13. [Singleton, serialization and class loaders (Q76-Q79)](#13-singleton-serialization-and-class-loaders-q76-q79)
14. [Production pitfalls (Q80-Q82)](#14-production-pitfalls-q80-q82)

---

## 1. main() and source files (Q1-Q5)

### Q1. Can we run a Java main program without `public`, `static` and `void`?
**Answer: Yes, from Java 25.** In older versions `public static void main(String[] args)` was mandatory. Java 25 allows an instance `main` with no `public`, no `static` and no `String[]` parameter.
```java
// Java 25+
class Hello {
    void main() {
        System.out.println("Hello");
    }
}
```

### Q2. Can we write a Java file with no class in it?
**Answer: Yes, from Java 25.** These are called *compact source files*. We write methods directly in the file, and the compiler puts everything into an implicitly declared class.
```java
// Hello.java (Java 25+), run with: java Hello.java
void main() {
    System.out.println("No class wrapper needed");
}
```

### Q3. Can we override the `main` method?
**Answer: No.** `main` is static, and static methods are never overridden. Writing the same `main` in a child class is **method hiding**. Which one runs is decided at compile time by the class name, not at runtime by the object.
```java
class Parent { public static void main(String[] a) { System.out.println("Parent main"); } }
class Child extends Parent { public static void main(String[] a) { System.out.println("Child main"); } }
// java Parent -> Parent main
// java Child  -> Child main   (hiding, not overriding)
```

### Q4. Can the JVM call our overloaded `main`?
**Answer: No.** We can overload `main` as many times as we like and it compiles, but the JVM looks for one exact signature, `main(String[])`, and ignores the others. To run an overloaded version, call it yourself from the real `main`.
```java
public class Demo {
    public static void main(String[] args) {
        System.out.println("real main");
        main(5);                      // we must call the overload ourselves
    }
    static void main(int x) { System.out.println("overloaded main " + x); }
}
```

### Q5. Can we print something without writing a `main` method?
**Answer: Only up to Java 6.** Until Java 6, a `System.out.println` in a `static` block ran at class-loading time and printed before the JVM looked for `main`. From Java 7 onwards the JVM checks for `main` first, so we get a `NoSuchMethodError` and nothing prints.
```java
public class Demo {
    static { System.out.println("static block"); }   // Java 6: prints. Java 7+: never reached
}
```

---

## 2. Inheritance, interfaces and abstract classes (Q6-Q16)

### Q6. Can a class extend itself?
**Answer: No.** The compiler reports a *cyclic inheritance* error. If `A extends A`, creating an `A` would need the parent constructor, which is `A` itself, and it would go on forever.
```java
class A extends A { }   // error: cyclic inheritance involving A
```

### Q7. Can a class extend two classes?
**Answer: No.** Java does not allow multiple inheritance of classes because of the **diamond problem**. If both parents have the same method, the child has no way to decide which one to inherit.
```java
class C extends A, B { }   // compile error
```

### Q8. Can an interface extend multiple interfaces?
**Answer: Yes.** An interface can extend as many interfaces as it wants. It is allowed here but not for classes because interfaces traditionally had only method declarations: no body, no state, no fields, no constructors, so there was nothing to conflict.
```java
interface A { void a(); }
interface B { void b(); }
interface C extends A, B { }   // valid
```

### Q9. Can two interfaces have the same default method?
**Answer: Yes, but the implementing class will not compile until we fix it.** Default methods (Java 8) brought the diamond problem back. Java handles it by forcing us to decide: the class must override the method itself. Java did not solve the diamond problem, it made us solve it.
```java
interface A { default void hi() { System.out.println("A"); } }
interface B { default void hi() { System.out.println("B"); } }
class C implements A, B {
    @Override public void hi() { A.super.hi(); }   // we must resolve it
}
```

### Q10. Can an interface have private methods?
**Answer: Yes, from Java 9.** Two default methods often need the same helper code, and before Java 9 there was no way to share it without making it public. Now an interface can have `private` and `private static` methods purely for internal reuse. They are not visible to implementing classes.
```java
interface Logger {
    default void info(String m)  { print("INFO",  m); }
    default void error(String m) { print("ERROR", m); }
    private void print(String level, String m) { System.out.println(level + ": " + m); }
}
```

### Q11. Can an interface have a constructor?
**Answer: No.** A constructor initializes the state of an object. An interface has no state and we can never create an object of an interface directly. Variables declared in an interface are automatically `public static final` constants, initialized once at the interface level.
```java
interface Config {
    int MAX = 10;     // implicitly public static final
    // Config() { }   // compile error
}
```

### Q12. Can an abstract class have a constructor?
**Answer: Yes.** We cannot create an object of an abstract class, but when a child object is created the parent constructor always runs first. The abstract class constructor initializes the fields the child inherits. If we don't write one, Java adds a default constructor anyway.
```java
abstract class Shape {
    final String name;
    Shape(String name) { this.name = name; }
    abstract double area();
}
class Circle extends Shape {
    final double r;
    Circle(double r) { super("circle"); this.r = r; }
    double area() { return Math.PI * r * r; }
}
```

### Q13. Can we create an object of an abstract class?
**Answer: No, not directly.** `new MyAbstract() { ... }` does not create an object of the abstract class. It creates an **anonymous subclass**, and the object belongs to that subclass. So never directly, but always through a subclass.
```java
Shape s = new Shape("anon") {
    double area() { return 0; }
};
```

### Q14. Can we override a static method?
**Answer: No. It is method hiding, not overriding.** Overriding is resolved at runtime by the actual object. Hiding is resolved at compile time by the reference type. With `Parent p = new Child()`, calling the static method runs the parent's version even though the object is a child. That is the opposite of what overriding does.
```java
class Parent { static String who() { return "Parent"; } }
class Child extends Parent { static String who() { return "Child"; } }
Parent p = new Child();
System.out.println(p.who());   // Parent
```

### Q15. Can we override a private method?
**Answer: No.** A private method is not even visible to the child class. A method with the same name in the child is treated as a completely new method with no relationship to the parent's. That is why `@Override` fails there.
```java
class P { private void m() { } }
class C extends P {
    @Override private void m() { }   // error: method does not override a supertype method
}
```

### Q16. Can a top-level class be private or protected?
**Answer: No.** A top-level class can only be `public` or default (package-private). `private` means accessible only within the enclosing class, and a top-level class has no enclosing class, so it would be useless because nobody could ever access it.
```java
private class A { }     // error: modifier private not allowed here
protected class B { }   // error: modifier protected not allowed here
```

---

## 3. Constructors (Q17-Q21)

### Q17. Can a constructor be `final`, `static` or `abstract`?
**Answer: No, none of the three, and each has a different reason.**
- `final` is pointless because a constructor is never inherited, so it can never be overridden anyway.
- `static` is wrong because a constructor initializes an instance and static means no instance.
- `abstract` means no body, and something with no body can never initialize anything.
```java
class A {
    final A() { }      // error
    static A() { }     // error
    abstract A();      // error
}
```

### Q18. Can a constructor be private?
**Answer: Yes, and it is very common.** Nobody outside can create an object directly. That is what we want for a **singleton**, where we control object creation, and for **utility classes** like `Math`, where every method is static and an object makes no sense.
```java
final class MathUtil {
    private MathUtil() { }
    static int square(int x) { return x * x; }
}
```

### Q19. Can a constructor call another constructor?
**Answer: Yes, with `this(...)`.** This is called **constructor chaining**. A no-argument constructor can call the three-argument one with default values, so the same initialization code is not repeated.
```java
class Box {
    int w, h, d;
    Box() { this(1, 1, 1); }
    Box(int w, int h, int d) { this.w = w; this.h = h; this.d = d; }
}
```

### Q20. Can we call `this()` and `super()` in the same constructor?
**Answer: No.** Both want to be the first statement and only one can be first. The logic still works out: if we call `this()`, that other constructor eventually calls `super()`, so the parent is initialized one level down the chain.

**From Java 25, flexible constructor bodies** let us write validation code *before* calling `this()` or `super()`. Earlier that was illegal, so we had to use ugly static helper methods to validate arguments.
```java
class Account extends Base {
    Account(int balance) {
        if (balance < 0) throw new IllegalArgumentException("negative");   // Java 25: allowed here
        super(balance);
    }
}
```

### Q21. Can a constructor throw an exception?
**Answer: Yes, both checked and unchecked.** If someone passes invalid data, we should fail right there rather than create a half-broken object.
```java
class User {
    User(String email) {
        if (email == null || !email.contains("@"))
            throw new IllegalArgumentException("invalid email");
    }
}
```

---

## 4. Enums (Q22-Q24)

### Q22. Can an enum have a constructor?
**Answer: Yes, but it is always private**, whether we write the keyword or not. The constants are fixed and created once when the enum loads, and nobody should be able to create a new one later.
```java
enum Level {
    LOW(1), HIGH(10);
    private final int value;
    Level(int value) { this.value = value; }   // implicitly private
}
```

### Q23. Can an enum extend a class?
**Answer: No.** Every enum already extends `java.lang.Enum` internally, and a class can extend only one class, so that slot is taken. This is also why an enum cannot be extended by another class: the compiler makes it final.
```java
enum Color extends Base { RED }   // compile error
```

### Q24. Can an enum implement an interface?
**Answer: Yes.** Interfaces are free even though the `extends` slot is taken. Each constant can have its own implementation of a method, which gives a clean **strategy pattern** with no if-else chain.
```java
interface Op { int apply(int a, int b); }
enum Operation implements Op {
    PLUS     { public int apply(int a, int b) { return a + b; } },
    MINUS    { public int apply(int a, int b) { return a - b; } },
    MULTIPLY { public int apply(int a, int b) { return a * b; } }
}
// Operation.PLUS.apply(2, 3) -> 5
```

---

## 5. Strings (Q25-Q29)

### Q25. Can we extend the `String` class?
**Answer: No, because `String` is final.** If we could extend it, someone could override `equals` or `length` and change its behavior. Imagine that object used as a `HashMap` key or in a security check comparing a file path, and everything breaks. `String` is used everywhere in Java, including inside the JVM itself, and making it final keeps it trustworthy.
```java
class MyString extends String { }   // error: cannot inherit from final String
```

### Q26. Can a `String` ever be made mutable?
**Answer: Yes, using reflection.** A `String` holds a private final byte array, but reflection can reach and modify it directly. The string that other classes share from the pool can then be changed. It shows why reflection is powerful and dangerous at the same time. **From Java 26, the JVM gives a warning when code mutates final fields this way.**
```java
String s = "hello";
Field f = String.class.getDeclaredField("value");
f.setAccessible(true);   // (added) Java 9+ also needs: --add-opens java.base/java.lang=ALL-UNNAMED
byte[] bytes = (byte[]) f.get(s);
bytes[0] = 'J';
System.out.println("hello");   // prints Jello, the pooled literal itself changed
```

### Q27. Can two strings with the same content have different references?
**Answer: Yes.** `String a = "hello"` goes into the string pool. `String b = new String("hello")` creates a brand new object on the heap outside the pool. The content is identical but `a == b` is `false` because they are two different objects. This is why we compare strings with `equals()` and never with `==`.
```java
String a = "hello";
String b = new String("hello");
System.out.println(a == b);        // false
System.out.println(a.equals(b));   // true
```

### Q28. Can `==` ever be true for strings created with `new`?
**Answer: Yes, if we call `intern()`.** `intern()` checks the pool. If the same content already exists there, it returns the pooled reference instead of our heap object, so both point to the same pool object.
```java
System.out.println(new String("hello").intern() == "hello");   // true
```

### Q29. Can `String.hashCode()` return zero?
**Answer: Yes.** The empty string always gives zero, and some non-empty strings also return zero, which depends on the hash calculation.
```java
System.out.println("".hashCode());           // 0
System.out.println("f5a5a608".hashCode());   // 0  (added: a known non-empty example)
```

---

## 6. Immutability (Q30-Q32)

### Q30. Can we create our own immutable class?
**Answer: Yes.** Four steps: (1) make the class `final`, (2) make all fields `private final`, (3) give no setters, only getters, (4) initialize everything through the constructor. There is a fifth step that people usually forget, covered in the next question.
```java
public final class Point {
    private final int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
}
```

### Q31. Can an immutable class have a mutable field?
**Answer: It can, but then the class is no longer truly immutable.** This is the mistake almost everybody makes. `private final List<String>` only means the reference cannot be reassigned, while the list contents can still change. If a getter returns the list directly, the caller can call `add()` on it from outside. The fix (the fifth step) is **defensive copying**: copy the list in the constructor and return a copy or an unmodifiable view from the getter.
```java
public final class Team {
    private final List<String> members;
    public Team(List<String> members) {
        this.members = new ArrayList<>(members);               // copy in
    }
    public List<String> getMembers() {
        return Collections.unmodifiableList(members);          // view out
    }
}
```

### Q32. Can an immutable object still break in multithreading?
**Answer: No.** If the class is truly immutable and all fields are `final`, Java guarantees they are fully visible to other threads once the constructor finishes. This is the **final field guarantee**, and it makes immutable objects thread-safe. *(added: this holds as long as `this` does not escape during construction.)*
```java
final class Money {
    private final long cents;
    Money(long cents) { this.cents = cents; }   // safe to share between threads once constructed
}
```

---

## 7. Exceptions and finally (Q33-Q38)

### Q33. Can we write `try` without `catch`?
**Answer: Yes, in two ways.** (1) `try-finally`: skip the catch and only write `finally`. The exception still propagates up but our cleanup code runs. (2) `try-with-resources` (Java 7): a resource opened in the try parentheses closes automatically, so no catch is needed. The only thing we can never do is write `try` alone. It must have at least a `catch` or a `finally`.
```java
try { work(); } finally { cleanup(); }

try (var in = new FileInputStream("a.txt")) { in.read(); }   // no catch needed (read() still declares IOException, so the method needs throws)
```

### Q34. Can the `finally` block ever not execute?
**Answer: Yes, in a few cases.** If `System.exit()` is called inside `try`, the JVM shuts down immediately. If the JVM crashes or the thread is killed. If there is an infinite loop inside `try`, we never leave `try` so we never reach `finally`.
```java
try {
    System.exit(0);
} finally {
    System.out.println("never printed");
}
```

### Q35. Can `finally` change the value returned by `try`?
**Answer: Yes, if `finally` has its own `return`.** If `try` returns 1 and `finally` returns 2, the answer is 2. The return in `finally` completely overrides the one from `try`.
```java
static int test() {
    try { return 1; }
    finally { return 2; }
}
// test() -> 2
```

### Q36. Can we throw an exception from the `finally` block?
**Answer: Yes, but we should not.** If `try` already threw something, the exception from `finally` replaces it and the real cause is lost. This is why **try-with-resources** was designed better: if both the body and `close()` throw, the original exception wins and the other is attached to it as a **suppressed exception**. Nothing gets lost.
```java
try (Res r = new Res()) {
    throw new RuntimeException("body");
} catch (Exception e) {
    System.out.println(e.getMessage());            // body
    System.out.println(e.getSuppressed().length);  // 1  (the exception from close())
}
```

### Q37. Can we catch an `Error`?
**Answer: Yes, but we should not.** `Error` extends `Throwable`, so `catch (Error e)` compiles. An `Error` means the JVM itself is in trouble (`OutOfMemoryError`, `StackOverflowError`) and application code cannot meaningfully recover from it.
```java
try { recurseForever(); }
catch (StackOverflowError e) { /* compiles and runs, but recovery is unreliable */ }
```

### Q38. Can a static block throw an exception?
**Answer: Unchecked yes, checked never.** A static block has no `throws` clause, so there is nowhere to declare a checked exception. When it throws something, the JVM wraps it in `ExceptionInInitializerError` and the class fails to load. The tricky part: every later attempt to use that class gives `NoClassDefFoundError`, which tells us nothing about the real cause. The real error appeared only once, at the first attempt.
```java
class Config {
    static int value;
    static { value = 1 / 0; }
}
// 1st use of Config -> ExceptionInInitializerError (caused by ArithmeticException)
// later uses        -> NoClassDefFoundError: Could not initialize class Config
```

---

## 8. Collections (Q39-Q48)

### Q39. Can `HashMap` store a null key?
**Answer: Yes. Exactly one null key and any number of null values.** `HashMap` normally calls `hashCode()` on the key to find the bucket, but it cannot do that for null, so there is a special rule: null always goes into **bucket 0**. A map cannot hold duplicate keys, so putting a null key twice just overrides the first.
```java
Map<String, Integer> m = new HashMap<>();
m.put(null, 1);
m.put(null, 2);
System.out.println(m.size());       // 1
System.out.println(m.get(null));    // 2
```

### Q40. Can `ConcurrentHashMap` store a null key or value?
**Answer: No. It throws `NullPointerException`.** In a single-threaded `HashMap`, if `get()` returns null we can call `containsKey()` to tell whether the key is missing or the value was null. In `ConcurrentHashMap` that check is unreliable because between `get()` and `containsKey()` another thread could have changed everything. So Java removed the ambiguity: null is not allowed, and null always means "not present".
```java
Map<String, String> m = new ConcurrentHashMap<>();
m.put("k", null);   // NullPointerException
m.put(null, "v");   // NullPointerException
```

### Q41. Can two keys with the same hash code both be stored in a `HashMap`?
**Answer: Yes.** The same hash code does not mean the same key. Both keys land in the same bucket (a **collision**). `HashMap` then compares them with `equals()`, and if `equals()` says they are different, both are stored in that bucket as a linked list. `hashCode()` decides where to look and `equals()` decides what to return.
```java
Map<String, Integer> m = new HashMap<>();
m.put("Aa", 1);
m.put("BB", 2);                        // "Aa".hashCode() == "BB".hashCode() == 2112
System.out.println(m.size());          // 2
```

### Q42. Can we use a mutable object as a `HashMap` key?
**Answer: We can, but we will lose the data.** The map calculates the key's hash and stores it in, say, bucket 5. If we then change a field inside the key, its hash code changes too. A later `get()` with the same object calculates a new hash, goes to bucket 9 and finds nothing. The entry is still in bucket 5 but unreachable forever. That is why `String` is the most popular key: it is immutable so its hash never changes.
```java
class Key { int id; Key(int id) { this.id = id; }
    public int hashCode() { return id; }
    public boolean equals(Object o) { return o instanceof Key k && k.id == id; } }

Map<Key, String> m = new HashMap<>();
Key k = new Key(5);
m.put(k, "value");
k.id = 9;
System.out.println(m.get(k));   // null, the entry is lost
```

### Q43. Can a `HashSet` contain duplicates?
**Answer: Yes, if we break the contract.** `HashSet` uses `hashCode()` and `equals()` to detect duplicates. If we override `equals()` but not `hashCode()`, two logically equal objects get different hash codes, go to different buckets, and the set never even compares them. Both get stored.
```java
class P { String n; P(String n) { this.n = n; }
    public boolean equals(Object o) { return o instanceof P p && p.n.equals(n); } }   // no hashCode()

Set<P> s = new HashSet<>();
s.add(new P("a"));
s.add(new P("a"));
System.out.println(s.size());   // 2 (almost always)
```

### Q44. Can we override `equals()` without `hashCode()`?
**Answer: The compiler allows it, but it breaks our code in every hash-based collection** (`HashMap`, `HashSet`, `Hashtable`). The contract says that if two objects are equal, their hash codes must be equal. Always override both together.
```java
@Override public boolean equals(Object o) { /* ... */ }
@Override public int hashCode() { return Objects.hash(name, id); }   // always add this
```

### Q45. Can we modify a list while iterating it?
**Answer: Not with a normal for-each loop, or we get `ConcurrentModificationException`.** The iterator keeps a count called `modCount`. If the list changes by any route other than the iterator itself, the count no longer matches and the iterator fails fast. Correct ways: `Iterator.remove()`, `removeIf()` (Java 8), or `CopyOnWriteArrayList`, whose iterator works on a snapshot.
```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4));

for (Integer x : list) { if (x == 2) list.remove(x); }   // ConcurrentModificationException

Iterator<Integer> it = list.iterator();                  // correct
while (it.hasNext()) { if (it.next() == 2) it.remove(); }

list.removeIf(x -> x == 3);                              // correct, Java 8+
```

### Q46. Can a `TreeMap` store null keys?
**Answer: No, for a different reason than `ConcurrentHashMap`.** `TreeMap` keeps everything sorted. To place a key it must compare it with existing keys by calling `compareTo()`, and calling that on null gives `NullPointerException`. So `HashMap` allows a null key but `TreeMap` doesn't, purely because one hashes and the other compares.
```java
new TreeMap<String, Integer>().put(null, 1);   // NullPointerException
```

### Q47. Can a `HashMap` cause an infinite loop?
**Answer: Yes, but only before Java 8.** If two threads resized a `HashMap` at the same time, the linked list inside a bucket could end up with a circular reference: node A pointing to B and B pointing back to A. After that, any `get()` on that bucket loops forever, CPU hits 100% and the application freezes, with no exception and no error message. Java 8 changed the resize logic to preserve order, so this particular bug is gone. *(added: `HashMap` is still not thread-safe. Use `ConcurrentHashMap` for shared maps.)*
```java
// Java 7 and earlier: concurrent put() calls on a plain HashMap could create a cycle in a bucket
Map<Integer, Integer> shared = new HashMap<>();   // do not share across threads
```

### Q48. Can `Collections.sort()` throw an exception on valid data?
**Answer: Yes. The data is fine but our comparator is the problem.** If the comparator is inconsistent (A greater than B, B greater than C, but C greater than A), sorting can fail. Since Java 7, sorting uses **TimSort**, which detects the inconsistency and refuses to continue with `IllegalArgumentException: Comparison method violates its general contract!`. The old algorithm silently produced a wrongly sorted list. The usual cause is a comparator that is not symmetric, or one that **subtracts** and overflows with large numbers.
```java
list.sort((a, b) -> a - b);              // wrong: overflows for large or negative values
list.sort(Integer::compare);             // correct
list.sort(Comparator.naturalOrder());    // correct
```

---

## 9. Generics (Q49-Q51)

### Q49. Can we create an array of generics?
**Answer: No.** Arrays and generics work in opposite ways. An array knows its element type at runtime and checks every element put in it. Generics use **type erasure**, so by that time the type information is gone.
```java
List<String>[] a = new List<String>[10];   // compile error: generic array creation
List<String>[] b = new List[10];           // compiles with an unchecked warning
List<List<String>> c = new ArrayList<>();  // preferred alternative
```

### Q50. Can we use primitives with generics?
**Answer: No.** Again it comes back to erasure. After compilation the generic type becomes `Object`, and a primitive like `int` is not an object. Use the wrapper types.
```java
List<int> bad = new ArrayList<>();        // compile error
List<Integer> ok = new ArrayList<>();     // use the wrapper (autoboxing)
```

### Q51. Can a `List<String>` be assigned to a `List<Object>`?
**Answer: No, even though `String` is an `Object`.** If it were allowed, someone could add an `Integer` through the `List<Object>` reference (a valid object after all), and the original code would later read it expecting a `String` and crash. Generics are **invariant** on purpose. For flexibility, use wildcards.
```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings;            // compile error

List<? extends Object> view = strings;     // OK to read from
```

---

## 10. Threads and synchronization (Q52-Q62)

### Q52. Can we start a thread twice?
**Answer: No. The second `start()` throws `IllegalThreadStateException`.** A thread has a life cycle (new, runnable, running, dead). Once it is dead it can never go back, and `start()` only works on a thread in the *new* state. To run the same task again, create a new `Thread` object, or better, use a thread pool.
```java
Thread t = new Thread(() -> System.out.println("hi"));
t.start();
t.start();   // IllegalThreadStateException
```

### Q53. Can we call `run()` instead of `start()`?
**Answer: Yes, it compiles, but no new thread is created.** Calling `run()` directly is an ordinary method call and the code executes sequentially on the current thread. Only `start()` asks the JVM to create a new thread, which then calls `run()`.
```java
Thread t = new Thread(() -> System.out.println(Thread.currentThread().getName()));
t.run();     // prints "main"
t.start();   // prints "Thread-0"
```

### Q54. Can a dead thread be restarted?
**Answer: No. Dead is permanent.** Once `run()` finishes, the thread's resources are released and the object can never be started again.
```java
Thread t = new Thread(() -> { });
t.start();
t.join();
t.start();   // IllegalThreadStateException, the thread is dead
```

### Q55. Can `volatile` make `count++` atomic?
**Answer: No.** This is probably the most important misconception in Java concurrency. `volatile` gives **visibility**: every thread reads the latest value from main memory instead of its own cached copy. But `count++` is three operations (read, add one, write back) and `volatile` does not lock anything. Two threads can read the same value, both add one, and we lose an increment. For atomicity use `AtomicInteger`, `synchronized` or a lock.
```java
volatile int count;
void unsafe() { count++; }                       // not atomic

AtomicInteger counter = new AtomicInteger();
void safe() { counter.incrementAndGet(); }       // atomic
```

### Q56. Can two threads enter the same `synchronized` method at once?
**Answer: Yes, if they work on different objects.** A synchronized method locks the **object**, not the method. Thread 1 on object A and thread 2 on object B run at the same time. They only block each other if it is the same object, or if the method is `static`, because a static synchronized method locks the **class**.
```java
class Counter {
    synchronized void inc() { /* locks "this" */ }
    static synchronized void sinc() { /* locks Counter.class */ }
}
// a.inc() and b.inc() run in parallel for two different objects a and b
```

### Q57. Can a thread acquire the same lock twice?
**Answer: Yes. Java locks are re-entrant.** If a synchronized method calls another synchronized method on the same object and locks were not re-entrant, the thread would block waiting for a lock it already holds. The JVM keeps a counter: each acquire increases it, each exit decreases it, and the lock is released only when it reaches zero.
```java
class Demo {
    synchronized void a() { b(); }   // holds the lock on "this"
    synchronized void b() { }        // takes the same lock again, no deadlock
}
```

### Q58. Can a constructor be synchronized?
**Answer: No.** The compiler rejects it, and it is also unnecessary: while a constructor is running, no other thread can access that object because the reference has not been published yet.
```java
class A { synchronized A() { } }   // compile error: modifier synchronized not allowed here
```

### Q59. Can `sleep()` release the lock?
**Answer: No.** This is the classic `sleep()` vs `wait()` question. `sleep()` pauses the thread but keeps every lock, so other threads needing that lock keep waiting. `wait()` does the opposite: it releases the lock so another thread can proceed, and waits to be notified. That is why `wait()` must be called inside a synchronized block, whereas `sleep()` can be called anywhere.
```java
synchronized (lock) {
    Thread.sleep(1000);   // still holds "lock" for the whole second
    lock.wait();          // releases "lock" until notify()
}
```

### Q60. Can `Thread.interrupt()` actually stop a thread?
**Answer: No. It only sets a flag.** It is a polite request, not a command. If the thread is inside `sleep()`, `wait()` or `join()`, it gets an `InterruptedException` and can react. If it is busy in a plain loop, nothing happens unless the code itself checks `isInterrupted()`. There is no safe way to forcibly kill a thread in Java. `stop()` existed once and was deprecated because it released locks and left objects in a broken state.
```java
Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // do work
    }
});
t.start();
t.interrupt();   // the loop ends only because it checks the flag
```

### Q61. Can a deadlock happen with only one lock?
**Answer: Yes.** People assume deadlock always needs two locks, but it doesn't. If a thread holds a lock and calls `wait()` expecting a notification that nobody will ever send, it waits forever.
```java
synchronized (lock) {
    lock.wait();   // no other thread ever calls lock.notify(): stuck forever
}
```

### Q62. Can we make a `HashMap` thread-safe without `ConcurrentHashMap`?
**Answer: Yes, with `Collections.synchronizedMap()`.** The wrapper puts one lock around the entire map, so only one thread can touch it at a time, whether reading or writing. `ConcurrentHashMap` locks only the specific bucket being written to, and reads need no lock at all, so multiple threads work in parallel.
```java
Map<String, Integer> m = Collections.synchronizedMap(new HashMap<>());
```

---

## 11. JVM, garbage collection and memory (Q63-Q67)

### Q63. Can a Java app run without the JRE installed?
**Answer: Yes, from Java 9 with `jlink`.** Earlier, the JRE had to be downloaded and installed on every machine. `jlink` bundles a small custom runtime together with our application, so the application carries its own JVM.
```bash
jlink --module-path $JAVA_HOME/jmods --add-modules java.base --output myruntime
myruntime/bin/java -version
```

### Q64. Can we force garbage collection?
**Answer: No.** Most people say yes because of `System.gc()`, but that is only a **request**, like asking a friend to please clean the room. The JVM can completely ignore it. There is no API in Java that guarantees garbage collection runs right now.
```java
System.gc();   // a hint only, the JVM may ignore it
```

### Q65. Can two objects that reference each other be garbage collected?
**Answer: Yes.** Java does not count references, it checks **reachability**. It starts from **GC roots** (the main thread stack, static variables) and asks: can I reach this object from there? If nothing from outside can reach A and B, both become garbage together, even though they hold each other.
```java
class Node { Node other; }
Node a = new Node(), b = new Node();
a.other = b;
b.other = a;
a = null;
b = null;     // both are now unreachable and eligible for GC
```

### Q66. Can an object come back to life after becoming eligible for GC?
**Answer: Yes. This is called object resurrection.** Before collecting an object the JVM calls its `finalize()` method. If inside `finalize()` we assign `this` to a static variable, the object becomes reachable again. But the JVM calls `finalize()` only once per object, so the second time it becomes garbage there is no escape. *(added: `finalize()` is deprecated for removal in modern Java. This is for interview knowledge, not for production use.)*
```java
class Zombie {
    static Zombie saved;
    @Override protected void finalize() { saved = this; }   // resurrects the object
}
```

### Q67. Can a memory leak happen even with automatic GC?
**Answer: Yes.** This is where a lot of production issues come from. GC removes only objects that nobody can reach. If we still hold a reference, GC cannot help, because it thinks we still need the object. Common cases: a static list that keeps growing and is never cleared, listeners registered but never removed, and `ThreadLocal` values in a thread pool where the thread stays alive forever.
```java
class Registry {
    static final List<byte[]> CACHE = new ArrayList<>();
    static void handle() { CACHE.add(new byte[1_000_000]); }   // grows forever, never cleared
}
```

---

## 12. Lambdas, streams, Optional, records and var (Q68-Q75)

### Q68. Can a functional interface have more than one method?
**Answer: Yes, as long as only one of them is abstract.** It can have any number of `default` and `static` methods. It can also declare public methods of `Object` such as `equals()` and `toString()`, and those don't count either. The best example is `Comparator`: `compare()` is its single abstract method, plus many default and static helpers like `reversed()` and `comparing()`.
```java
@FunctionalInterface
interface Validator {
    boolean validate(String s);                                   // the one abstract method
    default Validator and(Validator o) { return s -> validate(s) && o.validate(s); }
    static Validator notEmpty() { return s -> !s.isEmpty(); }
    boolean equals(Object o);                                     // Object method, doesn't count
}
```

### Q69. Can a lambda use a non-final local variable?
**Answer: Only if the variable is effectively final**, meaning we never change it after assigning it (Java 8 onwards, so the `final` keyword is no longer needed). If we modify it anywhere, the lambda won't compile. The reason: a lambda captures the **value** of a local variable, not a reference to it, and the lambda might run later on a different thread after the method has finished and its stack frame is gone. Instance and static variables have no such restriction because they live on the heap.
```java
int x = 10;
Runnable r = () -> System.out.println(x);   // OK, x is effectively final
x++;                                        // now the lambda above fails to compile
```

### Q70. Can we reuse a stream?
**Answer: No.** A stream can be consumed only once. After a terminal operation like `collect()` or `forEach()` the stream is closed, and using it again throws `IllegalStateException: stream has already been operated upon or closed`. If we need it twice, go back to the source and create a fresh stream, or use a `Supplier<Stream<T>>` that builds a new one each time.
```java
Stream<String> s = Stream.of("a", "b");
s.forEach(System.out::println);
s.count();                                   // IllegalStateException

Supplier<Stream<String>> sup = () -> Stream.of("a", "b");
sup.get().forEach(System.out::println);      // fresh stream each time
sup.get().count();
```

### Q71. Can a stream run without a terminal operation?
**Answer: No, and nothing happens at all.** Intermediate operations like `map`, `filter` and `sorted` are **lazy**: they only build a pipeline. With no terminal operation, the code inside `map` never executes, not even once. This surprises people debugging with `peek()`: they add a print statement and see nothing.
```java
Stream.of(1, 2, 3).map(x -> { System.out.println("map " + x); return x * 2; });   // prints nothing

Stream.of(1, 2, 3).map(x -> { System.out.println("map " + x); return x * 2; })
      .toList();                                                                  // now it runs
```

### Q72. Can a parallel stream be slower than a sequential one?
**Answer: Yes, but not always.** A parallel stream splits the data, distributes it across threads, then merges the results. It is bad for small data sets, tiny work per element, sources that are hard to split (like `LinkedList`), and operations that involve I/O or shared state. It works well for a large data set, CPU-heavy work per element, and an easily splittable source like an array or `ArrayList`.
```java
List<Integer> big = IntStream.range(0, 10_000_000).boxed().toList();
big.parallelStream().mapToLong(Integer::longValue).sum();   // good: big, CPU-bound, splittable
```

### Q73. Can an `Optional` itself be null?
**Answer: Unfortunately, yes.** `Optional<String> name = null;` and then `name.isPresent()` gives `NullPointerException`, the very thing `Optional` was invented to prevent. The rule: never return null from a method that returns `Optional`. Return `Optional.empty()` instead.
```java
Optional<String> bad = null;
bad.isPresent();                     // NullPointerException

Optional<String> good = Optional.empty();
```

### Q74. Can a record be mutable?
**Answer: The record itself is immutable, but it can still hold a mutable object.** Record fields are `final`, so they can never be reassigned. But if a component is a list, the contents of that list can still be modified by anyone who holds it. The fix is a **compact constructor** that copies the list when the record is created.
```java
record Team(String name, List<String> members) {
    Team {
        members = List.copyOf(members);   // defensive copy, now truly immutable
    }
}
```

### Q75. Can `var` be used as a field type?
**Answer: No.** `var` works only for **local variables**: inside a method, in a for loop, and in try-with-resources. It cannot be used for fields, method parameters, return types or constructor parameters. A field or method signature is part of the class's public contract and Java wants that contract written explicitly, while a local variable is our own business.
```java
class A {
    var field = 10;                    // error
    void m(var p) { }                  // error
    void ok() { var x = 10; }          // OK
}
```

---

## 13. Singleton, serialization and class loaders (Q76-Q79)

### Q76. Can a singleton have more than one instance?
**Answer: Yes. There are four ways to break it.**
1. **Reflection:** the constructor is private, but `setAccessible(true)` creates a brand new object.
2. **Serialization:** deserializing gives a fresh object. Fix: add a `readResolve()` method that returns the existing instance.
3. **Cloning:** if the class implements `Cloneable`, `clone()` gives a second object.
4. **Multiple class loaders:** each class loader loads its own copy of the class with its own static instance, so one JVM can have two singletons.
```java
public class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() { }
    public static Singleton get() { return INSTANCE; }
    protected Object readResolve() { return INSTANCE; }    // fixes serialization
}

Constructor<Singleton> c = Singleton.class.getDeclaredConstructor();
c.setAccessible(true);
Singleton second = c.newInstance();                        // reflection breaks it
```
*(added: an `enum` singleton is the usual way to defend against reflection and serialization.)*

### Q77. Can a `transient` field still be serialized?
**Answer: No. That is the entire purpose of `transient`.** On deserialization the field comes back as its default value (null or zero). **But** with custom serialization (`writeObject` and `readObject` methods) we can explicitly write the transient field ourselves. This is used in the JDK: `ArrayList` marks its internal array `transient` and writes only the used elements manually, instead of the whole array with its empty capacity.
```java
class User implements Serializable {
    String name;
    transient String password;                          // not serialized by default

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeObject(password);                      // we choose to write it ourselves
    }
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
        password = (String) in.readObject();
    }
}
```

### Q78. Can two classes with the same fully qualified name exist in one JVM?
**Answer: Yes, if they are loaded by different class loaders.** A class's identity in the JVM is not just its name, it is the **name plus the class loader** that loaded it. So `com.app.User` loaded by loader A and `com.app.User` loaded by loader B are two completely different classes. This is how application servers run multiple applications side by side, each with its own version of the same library.
```java
URLClassLoader a = new URLClassLoader(new URL[]{ urlOfJar }, null);
URLClassLoader b = new URLClassLoader(new URL[]{ urlOfJar }, null);
Class<?> c1 = a.loadClass("com.app.User");
Class<?> c2 = b.loadClass("com.app.User");
System.out.println(c1 == c2);   // false: same name, different classes
```

### Q79. Can static fields be shared across class loaders?
**Answer: No.** Each class loader has its own copy of the class and therefore its own static fields. That is why a singleton can exist twice in one JVM, and why a static cache in a shared library does not actually get shared between two applications on the same server. They share only if the same class is loaded by a common **parent** class loader, because of the parent-first delegation model.
```java
// c1 and c2 from the previous question are different classes,
// so c1's static fields and c2's static fields are separate copies.
```

---

## 14. Production pitfalls (Q80-Q82)

### Q80. Can `ThreadLocal` cause a memory leak?
**Answer: Yes, and it is one of the most common leaks in real applications.** The `ThreadLocal` map holds the **key as a weak reference** but the **value as a strong reference**. If the `ThreadLocal` key is garbage collected, we are left with an entry whose key is null and whose value is still strongly held. In a thread pool the threads live forever, so those values are never cleaned. *(added: the fix is to always call `remove()` in a `finally` block.)*
```java
static final ThreadLocal<byte[]> CTX = new ThreadLocal<>();

void handleRequest() {
    CTX.set(new byte[1_000_000]);
    try {
        // work
    } finally {
        CTX.remove();   // always clean up, especially on pooled threads
    }
}
```

### Q81. Can a parallel stream starve our whole application?
**Answer: Yes, and it is a brilliant production issue.** Every parallel stream in the entire JVM uses the same shared `ForkJoinPool.commonPool()` by default, and its size is the **number of cores minus one**. If one slow parallel stream occupies those threads, every other parallel stream anywhere in the application waits behind it. If someone does blocking I/O inside a parallel stream, those few threads sit idle waiting.
```java
list.parallelStream().forEach(x -> callSlowRemoteService(x));   // blocks the shared common pool

ExecutorService pool = Executors.newFixedThreadPool(20);        // (added) better for blocking work
```

### Q82. Can we disable garbage collection completely?
**Answer: Yes, with Epsilon GC.** It is a real collector that allocates memory and **never reclaims any of it**. When the heap fills up, the application crashes with `OutOfMemoryError`. Useful for short-lived programs and performance testing.
```bash
java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC -Xmx512m MyApp
```

---

## Source

- Video: *82 Tricky Java Interview Questions | Most Candidates Get These Wrong*, GenZ Career, https://youtu.be/glsIlGLglzQ
- The channel offers its own PDF of these questions and an interview preparation kit, linked in the video description.
