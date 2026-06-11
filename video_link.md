# Video Link
This file lists all recorded videos for interview preparation.

## Week 1

- List/Set/Map video
  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/list_set_map.mov

- Java Basics Video
  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/java-basics.mov

- Web Basics Video
  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/web-basics.mov


## Week 2

- **Mock Practice Video 0605**
  Include: Exception Handling, Optional Class, Race Condition Handling, and Thread lifecycle

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0605.mov

  Scripts:

  1. How to handle exceptions:

      Java exceptions split into checked and unchecked. Checked ones are recoverable conditions the compiler forces me to catch or declare with throws; unchecked ones extend RuntimeException and signal bugs I should fix rather than catch. I raise exceptions with throw and declare them with throws. For handling, I use try-catch-finally — where finally always runs for cleanup — and prefer try-with-resources for anything AutoCloseable, since it closes files and streams automatically and removes the need for a manual finally.

  2. How to handle race condition? 
      - Locking: synchronized keyword and lock api.
      - Non-locking: use CAS principle based classes, like AtomicInteger, AomicLong etc. OR immutable class

  3. Thread lifecycle / Thread states

      - New, Runnable, Waiting, Blocked, Timed Waiting, Terminated
      - `start()` makes it Runnable; the scheduler runs it; it leaves Runnable into Blocked (waiting for a lock), Waiting (`wait`/`join` with no timeout), or Timed Waiting (`sleep`/timed `wait`); each of those returns to Runnable; and when `run()` completes it's Terminated."
      - New → Runnable: `start()`
      - Runnable → Running → Runnable: The scheduler picks a Runnable thread to actually execute.
      - Runnable → Blocked: The thread tries to enter a `synchronized` block/method but another thread holds the lock, so it waits to acquire the monitor lock. Once the lock is free and acquired, it goes back to Runnable.
      - Runnable → Waiting: The thread calls `wait()` (with no timeout), `join()` (no timeout), or `LockSupport.park()`. It waits indefinitely until another thread signals it — e.g. via `notify()`/`notifyAll()` — then returns to Runnable.
      - Runnable → Timed Waiting: Same idea but with a timeout — `sleep(ms)`, `wait(ms)`, `join(ms)`, etc. It returns to Runnable when the time expires or it's notified.
      - Runnable → Terminated: When the `run()` method finishes (returns normally or throws an uncaught exception), the thread dies and can't be restarted.

  4. What is Optional in Java and how can you use it? (Java 8 language/library feature)

      - Java 8 new feature
      - wrapper class to help avoid nullpointerexception, with orElseThrow(), ofNullable(), orElse().`ofNullable` to allow null values, `orElse` to provide default values, and `orElseThrow` to handle exceptions when values are missing

## Week 3

Note: for doc clarity, past scripts can be found at Week3/mock-questions-review-script.md

- **Mock Practice Video 0608**

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0608.mov


- **Mock Practice Video 0609**

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0609.mov


- **Mock Practice Video 0610**

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0610.mov 

  1. Where did you use singleton in your project?

      For Singleton Design Pattern, I use it for things that are expensive to create and meant to be shared once — a database connection pool, a thread pool, or loading a wide, fixed reference table into memory once instead of recreating it on every request. For example, a reference table with hundreds of columns that's fixed at runtime — you load it once as a singleton rather than rebuilding it repeatedly. The reason is the same in all cases: one shared instance, created once, to avoid wasting resources.

  2. How to use stream to filter people younger than 30

      I take the list of people, call .stream(), use .filter() with a predicate — p -> p.getAge() < 30 — and then .collect(Collectors.toList()) as the terminal operation to get the list of people under 30

  3. How do you design RESTful APIs to get/create/update an object

      I design it in four parts. First the HTTP methods for the CRUD operations — GET to read, POST to create, PUT to update, DELETE to remove. Then the URL — and the URL never contains verbs; what it carries is path variables. After that I consider request parameters, headers, and the request body — especially the body for a POST, where I send the new record as JSON. Then the response: the response body carries the payload plus the HTTP status code, and the status codes have five levels — 100 informational, 200 success, 300 redirect, 400 client error, 500 server error.

  4. Write REST api (User and ToDoItem): many to many relationship, create api for CRUD operations

      This is still a RESTful endpoint design question, so I start the same way — clarify the requirement and declare the data before coding. For the many-to-many, I model it with two entities, User and Todo, joined by a @ManyToMany relationship in Spring Data JPA, which creates a join table; I'd expose DTOs rather than the entities directly. Then I design the CRUD endpoints — GET to read, POST to create, PUT to update, DELETE to delete — each going through the controller to the service to the repository layer. So the same design structure as before: HTTP methods, URL with path variables, request body for creates, and response body plus status code.

  5. What is SOLID principle

      SOLID is five principles for object-oriented design — it's basically the detailed implementation of good OOD. 
      Single Responsibility: a class should have one reason to change, one job. 
      Open/Closed: open for extension, closed for modification — in Java that means I extend behavior through inheritance or interfaces rather than editing existing code. 
      Liskov Substitution: a subtype is usable wherever its parent is — implemented through polymorphism, like a List reference holding an ArrayList or LinkedList. 
      Interface Segregation: keep interfaces small and focused rather than one huge interface that forces classes to implement methods they don't need. 
      Dependency Inversion — depend on abstractions, which is exactly the reasoning behind Spring's IoC, where the container injects dependencies instead of the developer wiring them manually.