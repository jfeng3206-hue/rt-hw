# Java Interview Cheat-Sheet: Functional Programming & Version Evolution

A consolidated review covering four connected topics: **version evolution**, **functional interfaces**, **method references**, and the **Stream API**. They form one body of knowledge — Stream operations take functional interfaces, and you pass them lambdas or method references.

---

## Table of Contents

1. [Version Evolution (Java 8 / 17 / 21 / 23)](#1-version-evolution)
2. [Functional Interfaces](#2-functional-interfaces)
3. [Method References](#3-method-references)
4. [Stream API](#4-stream-api)

---

## 1. Version Evolution

### Overview

| Version | Year | LTS | Headline feature |
|---------|------|-----|-----------------|
| Java 8  | 2014 | ✅ | Lambdas + Streams — start of the functional era |
| Java 17 | 2021 | ✅ | Records, Sealed Classes, pattern matching foundations |
| Java 21 | 2023 | ✅ | **Virtual Threads (Loom)**, switch pattern matching |
| Java 23 | 2024 | ❌ | Mostly previews (primitive type patterns, etc.) — not LTS |

The LTS line that matters for production: **8 → 11 → 17 → 21 → 25**. Java 11 (2018) sits between 8 and 17; Java 25 (2025) is the next LTS after 21.

### Java 8 — The Functional Foundation

Everything later builds on it. Tests your fundamentals.

```java
// Lambda + Stream
List<String> names = users.stream()
    .filter(u -> u.getAge() > 18)
    .map(User::getName)
    .collect(Collectors.toList());

// Optional to avoid NPE
Optional.ofNullable(user).map(User::getName).orElse("Unknown");

// default methods on interfaces
interface Greeter { default String hi() { return "hi"; } }
```

Also `java.time` (replacing the mutable, non-thread-safe `Date`/`Calendar`).

### Java 17 — Modern Java Syntax Solidifies

```java
// Record: immutable data class — auto constructor/getters/equals/hashCode/toString
record Point(int x, int y) {}

// Sealed Class: restricts who can extend, pairs with pattern matching
sealed interface Shape permits Circle, Square {}

// instanceof pattern matching — no explicit cast
if (obj instanceof String s) { System.out.println(s.length()); }

// switch expressions
String type = switch (day) {
    case MON, TUE -> "weekday";
    case SAT, SUN -> "weekend";
};

// text blocks
String json = """
    { "name": "Jingya" }
    """;
```

Likely question: *"What does a Record give you, and when would you not use one?"* — It's for immutable data carriers (DTOs, value objects); avoid it when you need mutability or to extend a class.

### Java 21 — The One Backend Engineers Should Care About Most

Jumping from 8/11 straight to 21 gives the biggest payoff. Hottest interview topic right now.

```java
// Virtual threads: millions of concurrent tasks without thread-pool tuning;
// big throughput win for IO-bound workloads
Thread.startVirtualThread(() -> handleRequest());

try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(task);
}

// switch pattern matching + record deconstruction
String describe(Shape s) {
    return switch (s) {
        case Circle c        -> "circle, radius " + c.radius();
        case Square(int len) -> "square, side " + len;   // record pattern
        case null            -> "empty";
    };
}

// Sequenced Collections: unified first/last access
list.getFirst(); list.getLast(); list.reversed();
```

**Virtual threads vs platform threads** (must be able to contrast):

- A platform thread maps 1:1 to an OS thread (~1MB stack, expensive, limited to thousands).
- A virtual thread is JVM-managed, extremely lightweight (millions possible), and when it blocks on IO it *unmounts* from its carrier platform thread instead of blocking the OS thread.
- Payoff: keep writing simple blocking-style code, get async-level scalability — no full reactive rewrite.
- Gotcha: **avoid pinning** — `synchronized` blocks or native calls can pin a virtual thread to its carrier; prefer `ReentrantLock` in hot paths.

Spring Boot 3.2+ enables virtual threads with a single property: `spring.threads.virtual.enabled=true`.

### Java 23 — Non-LTS, Mostly Preview

Know the trends, not the details:

- Primitive types in patterns / `instanceof` / `switch` (preview)
- Markdown in Javadoc
- Generational ZGC becomes the default
- Structured Concurrency and Scoped Values still in preview
- ⚠️ **String Templates** (string interpolation), previewed in 21/22, were **withdrawn in 23 to be redesigned**. So "does Java have string interpolation?" → not officially yet.

### How to Prioritize

1. **Virtual threads (21)** — biggest JVM concurrency shift in a decade. Explain the platform-thread contrast and the pinning gotcha.
2. **Records + Sealed + pattern matching (17 → 21)** — Java's take on algebraic data types (ADTs). Common in domain modeling / DTO design.
3. **Streams / Lambdas / Optional (8)** — old but always tested; the fundamentals.
4. **GC evolution (G1 → ZGC)** — for large-heap, low-latency selection questions.

Strong closing move: group new features as *language ergonomics* (records, sealed, pattern matching, text blocks), *concurrency* (virtual threads, structured concurrency), and *runtime/GC* (ZGC).

---

## 2. Functional Interfaces

An interface with **exactly one abstract method** (Single Abstract Method, SAM). A lambda is the implementation of that one method.

```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);   // the single abstract method
}

Calculator add = (a, b) -> a + b;   // lambda implements it
add.operate(3, 5);                  // 8
```

The `@FunctionalInterface` annotation is **optional**, but it makes the compiler enforce the single-abstract-method rule.

**Gotcha:** `default` methods, `static` methods, and `Object` methods (equals/hashCode/toString) do **not** count as abstract methods.

```java
@FunctionalInterface
interface Foo {
    void doIt();                    // single abstract method ✅
    default void log() {}           // doesn't count
    static void util() {}           // doesn't count
    boolean equals(Object o);       // Object method, doesn't count — still valid ✅
}
```

### The Four Built-in Interfaces (highest frequency)

In `java.util.function`:

| Interface | Abstract method | Meaning | Typical use |
|-----------|----------------|---------|-------------|
| `Function<T,R>` | `R apply(T t)` | input T, return R | `stream.map()` |
| `Predicate<T>` | `boolean test(T t)` | input T, return boolean | `stream.filter()` |
| `Consumer<T>` | `void accept(T t)` | input T, no return | `stream.forEach()` |
| `Supplier<T>` | `T get()` | no input, return T | lazy init, `Optional.orElseGet()` |

```java
Function<String, Integer> len = String::length;
Predicate<Integer> isEven = n -> n % 2 == 0;
Consumer<String> print = System.out::println;
Supplier<List<String>> factory = ArrayList::new;
```

### Variants

```java
BiFunction<T,U,R>   // two args: R apply(T,U)
BinaryOperator<T>   // Function special case: (T,T) -> T
UnaryOperator<T>    // Function special case: T -> T
BiPredicate<T,U>    // boolean test(T,U)
BiConsumer<T,U>     // void accept(T,U)
```

**Primitive specializations** (`IntFunction`, `ToIntFunction`, `IntPredicate`, `IntSupplier`...) exist to avoid boxing/unboxing overhead. `Function<Integer,Integer>` boxes; `IntUnaryOperator` does not.

### High-Frequency Questions

**Q: Function composition?**

```java
Function<Integer,Integer> times2 = x -> x * 2;
Function<Integer,Integer> plus3  = x -> x + 3;

times2.andThen(plus3).apply(5);   // *2 then +3 → 13
times2.compose(plus3).apply(5);   // +3 then *2 → 16
```

`andThen` = current runs first; `compose` = argument runs first.

**Q: Predicate logical combination?**

```java
positive.and(even).test(4);     // true
positive.or(even).test(-2);     // true
positive.negate().test(-1);     // true
```

**Q: Lambda vs anonymous inner class?**

- Lambda has no own `this` — `this` refers to the **enclosing class**; anonymous class `this` refers to itself.
- Lambda generates no separate `.class` file; uses `invokedynamic` + `LambdaMetafactory`. Anonymous class generates `XXX$1.class`.
- Lambda only works for functional interfaces; anonymous class can implement any interface/abstract class.

**Q: Which variables can a lambda capture?**

Only **effectively final** local variables (assigned once, never reassigned — no `final` keyword needed). The lambda may run after the enclosing method returns, so the JVM captures by value.

```java
int count = 0;
Runnable r = () -> System.out.println(count);  // OK
// count = 1;  // reassigning anywhere makes the line above fail to compile
```

---

## 3. Method References

Syntactic sugar for a lambda that only **calls one existing method**. No extra capability over a lambda — just more concise.

```java
list.forEach(s -> System.out.println(s));   // lambda
list.forEach(System.out::println);          // method reference (equivalent)
```

### The Four Forms

| Form | Syntax | Equivalent lambda | Example |
|------|--------|-------------------|---------|
| 1. Static method | `Class::staticMethod` | `(x) -> Class.staticMethod(x)` | `Integer::parseInt` |
| 2. Instance method of a specific object | `instance::method` | `(x) -> instance.method(x)` | `System.out::println` |
| 3. Instance method of an arbitrary object | `Class::instanceMethod` | `(obj, x) -> obj.method(x)` | `String::length` |
| 4. Constructor | `Class::new` | `(x) -> new Class(x)` | `ArrayList::new` |

```java
Function<String,Integer> f   = Integer::parseInt;   // 1
Consumer<String> c           = System.out::println; // 2
Function<String,Integer> len = String::length;      // 3 — equals s -> s.length()
Supplier<List<String>> s     = ArrayList::new;       // 4
Function<Integer,int[]> arr  = int[]::new;           // array constructor
```

### Form 2 vs Form 3 (most confusing — interviewers love this)

Both look like `X::method` but differ in semantics.

```java
String prefix = "Hello";

// Form 2: instance::method — prefix is the fixed receiver
Function<String,Boolean> startsWith = prefix::startsWith;
startsWith.apply("Hel");         // prefix.startsWith("Hel") → true

// Form 3: Class::method — receiver is the first argument passed in
BiFunction<String,String,Boolean> sw = String::startsWith;
sw.apply("Hello", "Hel");        // "Hello".startsWith("Hel") → true
```

**Rule of thumb:**
- `object::method` → that object is the receiver; all args go to the method.
- `Class::method` → if the method is static, it's Form 1; if it's an instance method, it's Form 3 and **the first argument becomes the receiver**.

### High-Frequency Questions

**Q: How does `String::length` know it's an instance method?**
The compiler inspects `String`: `length()` is an instance method, so it's Form 3 and the first argument becomes `this`. With same-named static + instance methods, the compiler matches against the target functional interface signature, erroring only on ambiguity.

**Q: Performance vs lambda?**
Both compile to `invokedynamic` + `LambdaMetafactory`; runtime difference is negligible. One subtlety: `System.out::println` evaluates `System.out` once when the reference is created, whereas `() -> System.out.println(...)` reads `System.out` on each call.

**Q: Constructor reference with arguments?**

```java
Supplier<StringBuilder> s          = StringBuilder::new;   // no-arg
Function<String,StringBuilder> f   = StringBuilder::new;   // String-arg
BiFunction<String,Integer,User> u  = User::new;            // two-arg
```

**Q: When can't you use a method reference?**
When the lambda body does **extra work** on the args — only "pass args straight through to one method" converts.

```java
s -> s.length()              // ✅ → String::length
s -> s.length() + 1          // ❌ extra +1
s -> s.trim().length()       // ❌ two method calls
(a, b) -> a.compareTo(b)     // ✅ → String::compareTo (Form 3)
```

**Q: Most common in practice?**

```java
users.stream().map(User::getName).forEach(System.out::println);
users.sort(Comparator.comparing(User::getAge));
.collect(Collectors.groupingBy(User::getDept));
Optional.ofNullable(user).map(User::getName);
```

Containment to remember: **method reference ⊂ lambda ⊂ functional interface**.

---

## 4. Stream API

A declarative, chained pipeline over collection elements. Three essential traits: **stores no data** (a view over a source), **doesn't modify the source**, and is **single-use** (reusing throws `IllegalStateException`).

```java
List<String> result = users.stream()        // 1. create
    .filter(u -> u.getAge() > 18)            // 2. intermediate (lazy)
    .map(User::getName)                      // 3. intermediate (lazy)
    .collect(Collectors.toList());           // 4. terminal (triggers execution)
```

### Intermediate vs Terminal

Rule: operations returning a `Stream` are intermediate; operations returning something else are terminal.

| Type | Behavior | Methods |
|------|----------|---------|
| Intermediate | Lazy, returns new Stream | `filter` `map` `flatMap` `sorted` `distinct` `limit` `skip` `peek` |
| Terminal | Triggers the pipeline; only one | `collect` `forEach` `reduce` `count` `findFirst` `anyMatch` `min/max` `toArray` |

**Gotcha:** without a terminal operation, intermediates never run.

```java
Stream.of(1,2,3)
    .filter(n -> { System.out.println("filter " + n); return true; });
// Prints nothing — no terminal operation.
```

### Laziness and Short-Circuiting

A stream flows **element by element (vertically)**, not step-by-step across all elements.

```java
Stream.of("a","bb","ccc")
    .filter(s -> { System.out.println("filter:" + s); return s.length() > 1; })
    .map(s -> { System.out.println("map:" + s); return s.toUpperCase(); })
    .findFirst();
// Output: filter:a → filter:bb → map:bb (short-circuits; "ccc" never touched)
```

`findFirst`, `anyMatch`, `limit` stop as soon as satisfied.

### map vs flatMap

`map` is one-to-one; `flatMap` flattens (each element → a stream, then concatenated).

```java
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));
nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [1,2,3,4]
```

### reduce

```java
int sum = nums.stream().reduce(0, Integer::sum);       // with identity
Optional<Integer> max = nums.stream().reduce(Integer::max);  // without → Optional
```

### collect + Collectors

```java
.collect(Collectors.toList());
.collect(Collectors.toSet());
.collect(Collectors.toMap(User::getId, u -> u));   // duplicate keys throw

Map<String, List<User>> byDept =
    users.stream().collect(Collectors.groupingBy(User::getDept));

Map<String, Long> countByDept =
    users.stream().collect(Collectors.groupingBy(User::getDept, Collectors.counting()));

Map<Boolean, List<User>> parts =
    users.stream().collect(Collectors.partitioningBy(u -> u.getAge() > 18));

String names = users.stream().map(User::getName)
    .collect(Collectors.joining(", ", "[", "]"));
```

### High-Frequency Questions

**Q: Can a Stream be reused?**
No. After a terminal operation it's consumed; reusing throws `IllegalStateException: stream has already been operated upon or closed`. Recreate from source or wrap in `Supplier<Stream>`.

**Q: `stream()` vs `parallelStream()`? When to parallelize?**
Parallel uses `ForkJoinPool.commonPool()`. Pitfalls to name:
- Small data / cheap ops → scheduling overhead can make it slower.
- Shared mutable state → thread-safety bugs.
- Shares the common pool → one slow task can starve everything.
- Rule of thumb: large data, heavy per-element work, stateless ops with no side effects.

**Q: `findFirst` vs `findAny`?**
`findFirst` guarantees encounter order; `findAny` doesn't and performs better in parallel. Sequential → usually the same.

**Q: Can `peek` modify data?**
It's for **debugging**, not side effects. It's intermediate (lazy) — may never run if there's no terminal op or it gets short-circuited.

**Q: `Collectors.toMap` pitfall?**
Key collision throws by default; supply a merge function:

```java
.collect(Collectors.toMap(
    User::getName,
    u -> u,
    (existing, replacement) -> existing));   // who wins on conflict
```

**Q: How to create a Stream?**

```java
list.stream();
Stream.of(1, 2, 3);
Arrays.stream(array);
IntStream.range(0, 10);
Stream.iterate(1, n -> n * 2).limit(5);   // infinite + limit
Stream.generate(Math::random).limit(3);
```

**Q: Why primitive streams (IntStream/LongStream/DoubleStream)?**
Avoid boxing/unboxing; dedicated `sum()`, `average()`, `max()`, `summaryStatistics()`. `Stream<Integer>` has no `sum()`:

```java
int total = users.stream().mapToInt(User::getAge).sum();
IntSummaryStatistics stats = users.stream()
    .mapToInt(User::getAge).summaryStatistics();  // count/sum/min/max/avg in one pass
```

---

## How It All Connects

Every Stream operation (`map`/`filter`/`reduce`) takes a **functional interface** (`Function`/`Predicate`/`BinaryOperator`), and what you pass in is usually a **lambda or method reference**. Functional interfaces → method references → Stream API are three layers of the same body of knowledge. Connecting them in an interview is far more convincing than reciting them in isolation.
