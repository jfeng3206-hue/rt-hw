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

- Mock Practice Video 0605
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

- Mock Practice Video 0608
  
  Include: what does restful api mean?

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0608.mov

  Scripts:

  1. What does restful api mean?

    So when I design a RESTful endpoint, I'm building a stateless API over the HTTP protocol that I use for client-server communication. REST stands for Representational State Transfer — it's an architectural style. being stateless means each request is independent and the server doesn't hold session state between calls.

    When designing restful api, the first thing I define is the HTTP method — GET to read, POST to insert, PUT to update, DELETE to remove. Then the URL, which does not contain the verb or method. In the URL I can carry a path variable or request parameters. I can attach request headers — things like content type or an authorization token if I'm doing role-based access control. And for something like a POST, I'll have a request body, usually a JSON payload, which can be nested for more complex objects.
    
    Then on the response side, the server returns a response body plus an HTTP status code. The status codes fall into five families — 100 informational, 200 success, 300 redirect, 400 client-side error, and 500 server-side error.
    
    One thing worth calling out is idempotency — GET is idempotent, so repeating it doesn't change state, whereas POST isn't, which matters for how the API behaves on retries. 