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

- **Mock Practice Video 0608**
  
  Include: what does restful api mean?

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0608.mov

  Scripts:

  1. What does restful api mean?

    So when I design a RESTful endpoint, I'm building a stateless API over the HTTP protocol that I use for client-server communication. REST stands for Representational State Transfer — it's an architectural style. being stateless means each request is independent and the server doesn't hold session state between calls.

    When designing restful api, the first thing I define is the HTTP method — GET to read, POST to insert, PUT to update, DELETE to remove. Then the URL, which does not contain the verb or method. In the URL I can carry a path variable or request parameters. I can attach request headers — things like content type or an authorization token if I'm doing role-based access control. And for something like a POST, I'll have a request body, usually a JSON payload, which can be nested for more complex objects.
    
    Then on the response side, the server returns a response body plus an HTTP status code. The status codes fall into five families — 100 informational, 200 success, 300 redirect, 400 client-side error, and 500 server-side error.
    
    One thing worth calling out is idempotency — GET is idempotent, so repeating it doesn't change state, whereas POST isn't, which matters for how the API behaves on retries. 

- **Mock Practice Video 0609**

  https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0609.mov

  Scripts:

  1. Difference between recursion and iteration?

      Recursion is when a method calls itself, passing parameters down until it hits a base case. It's natural for things like tree or graph traversal, but it's less readable and it carries a risk for StackOverFlowError because every recursive call pushes a new frame onto the stack and those frames aren't released until the recursion end.
      
      Iteration uses a loop and works from the top down. Because there's no call overhead and no growing stack, it's generally better for performance in production and lighter on JVM memory.
      
      In practice I reach for iteration by default and only use recursion when the problem is naturally recursive and the depth is bounded.

  2. How to group people to key(age), value (list of people)

      Given a List<Person>, the cleanest way is the Stream API. I create a stream and use the terminal operation `collect` with `Collectors.groupingBy`, and I pass the classifier as a method reference on the age field (Person::getAge) using method reference. 

      ```Map<Integer, List<Person>> byAge = people.stream().collect(Collectors.groupingBy(Person::getAge));```
      
  3. How to write restapi in Spring Boot

    When I design a REST endpoint I work in the controller layer and I think in two halves: the request and the response.
    
    First the request. I decide the HTTP method — GET, POST, PUT, or DELETE — and I keep verbs out of the URL because the method already carries the verb. Then I define the user inputs: path variables, request params, request headers, and the request body, depending on what the operation needs.
    
    Then the response. That's the response body — the payload — plus the HTTP status code: 1xx, 2xx, 3xx, 4xx, 5xx, where each class has a defined meaning, like 2xx success, 4xx client error, 5xx server error.
    
    On the implementation side I annotate the class with @RestController and put @RequestMapping at the class level for the base URL — and I usually include a version there, like /api/v1/employees, so I can ship a v2 later without breaking existing clients. 
    
    For each endpoint I use @GetMapping, @PostMapping, etc. — or @RequestMapping with the method specified — and bind inputs with @PathVariable, @RequestParam, @RequestHeader, and @RequestBody.

  4. How did you debug

    My first step when a bug comes in is to reproduce it. Usually what I get from production is just a phenomenon: a user says they clicked a button but the item never shipped, or they expected a confirmation and nothing appeared. I typically don't even have access to run things live in prod, and I wouldn't want to test against the live business there anyway.

    So I start with the logs and the monitoring. I look at the HTTP status codes and any error or exception entries to identify which request is causing the problem. Once I find that request, I take its exact payload into a dev environment and replay it — Postman plus running the service locally in IntelliJ.

    That replay tells me where the bug lives. If the same request reproduces the failure locally, the problem is in my logic. If it doesn't reproduce, then it's likely an environment or configuration issue in prod, not my code. Only after I can reproduce it do I attach breakpoints and step through.

    This is exactly why monitoring matters in real systems — tools like Grafana for dashboards, backed by Prometheus and Spring Boot Actuator for metrics. 
    
  6. New feature in Java 11

    - var in lambda parameters — you can use var on lambda params so you can attach annotations to them.
    - Standard HTTP Client — the new java.net.http.HttpClient became a standard feature in 11 (it was incubating in 9). Supports HTTP/2 and async requests.
    - Single-file source execution — you can run a .java file directly with java Foo.java, no separate compile step, which is handy for scripting.
    - New garbage collectors — ZGC was introduced as an experimental low-latency collector, and Epsilon is a no-op collector that handles allocation but never actually reclaims memory — useful for performance testing.
 
  8. Design the locking schema so that when a thread call method1(), it needs to until some other thread call method2()

    Option 1 — synchronized key word and wait() / notify(). When thread A enters method1(), I have it wait() until method2() has done its work and released its lock. When the other thread runs method2(), it calls notify() (or notifyAll()) to signal A, which then wakes up, reacquires the lock, and continues executing method1(). Both must happen inside a synchronized block on a shared monitor object.

    Option 2 — Lock interface plus Condition. I'd use a ReentrantLock and create a Condition from it. method1() calls condition.await() to release the lock and wait; method2() calls condition.signal() to wake it. This is the more modern, flexible equivalent of wait/notify and lets me have multiple named conditions if needed.