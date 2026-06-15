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


**Mock Questions 0611**

Link:https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0611.mov

- what annotations you used in spring / How does Spiring IoC work (all annotations and injection and bean types)

  Start with two features: IoC and AOP

  - IoC: @SpringBootApplication enables auto-configuration, Spring configuration and component scan. Register beans with @Controller/@Restcontroller, @Service, @Repository for 3-tier layers. @Component for anything outside those layers; @Bean + @Configuration for third-party classes.  Different bean scopes using @Scope: Singleton, prototype, session, request, application

    -IoC implemented via DI using @Autowire, which has three types (field, construcotr, setter)

  - AOP: AOP has two styles. The first is @RestControllerAdvice with @ExceptionHandler for handling exceptions at the controller layer. The second is the aspect style: I define an @Aspect with a @Pointcut for where, and the advice annotations — @Around, @Before, @After, @AfterThrowing, @AfterReturning — for when the logic runs.

  - RESTful endpoints: Since development is annotation-driven, I design endpoints to avoid boilerplate using @RestController, then @RequestMapping for the parent URL, and @GetMapping / @PostMapping / @PutMapping / @DeleteMapping for each operation. For the request side I handle inputs with @RequestParam, @PathVariable, @RequestHeader, and @RequestBody. For the response, I either return through @RestController directly or use @ResponseBody, always with the appropriate HTTP status code.

- completablefuture / multithreading in spring framework

  First, CompletableFuture is a java 8 feature as a replacement for the older FutureTask. FutureTask blocks the main thread when using get() and waiting for the result, whereas CompletableFuture is non-blocking and gives chaining APIs like thenApply and thenCompose. So in my service layer, if i have a payload i can slice into chunks, i can use completableefuture to process each chunk in paralle and join the results and return one integrated response to the user. 

  Second, in Spring, I put @Async on a service method — say it returns CompletableFuture<User> — so when multiple requests hit that method they execute in parallel instead of in series. For that to work I need @EnableAsync, and I configure a thread pool by implementing AsyncConfigurer and overriding the executor with a ThreadPoolTaskExecutor. After Java 21 there are also virtual threads, where I don't even need to manage a pool — I just wrap the task in a virtual thread.

  When I customize the thread pool, there are seven parameters — core pool size, max pool size, keep-alive time, the work queue, and so on. And an important design point: I segregate pools by task type
    - Core tasks — like order processing or anything financial, where the company is actually making money — get a larger, more powerful primary pool with more threads and resources. 
    - Auxiliary tasks — email or SMS notifications, monthly report generation — go to a smaller pool, and there I can use a lenient rejection policy like discard-oldest, because some latency or even dropping a task there is acceptable. 
    - If I have multiple pools I use @Qualifier to pick which one a method uses.
  
  Third, the thread-safety layer underneath all of this. For shared state I use atomic types like AtomicReference with compare-and-swap as a non-locking mechanism, ConcurrentHashMap for concurrent collections, the synchronized keyword for critical sections, and the Lock interface with ReentrantLock when I need more control. So altogether it's CompletableFuture and @Async for the async execution, thread pools for managing it, and CAS plus locks for thread safety.

- what annotations we use to configure customized actuator

  Actuator expose endpoints to monitor application status.

  Customization annotations exist — @Endpoint, @ReadOperation (GET), @WriteOperation (POST), @DeleteOperation — but rarely customize it manually. 

- Could AOP apply to the private method?

  No. AOP works through proxies, which can't override private methods — and even nested public method calls aren't intercepted 

  Solutions: (1) extract the logic into a separate Spring bean and inject it, or (2) self-injection (breaking the circular dependency with @Lazy). (3) AspctJ

- how can you use/define profile

  - Profiles define environment-specific configuration, such as dev, test, prod, and activate the right set at runtime

  - Profiles are defined in profile-specific property files: application-dev.properties etc. Annotated beans with @Profile(\"dev"\) so they load under that profile. 
  
  - activated via spring.profiles.active=dev in properties. an environment variable SPRING_PROFILES_ACTIVE; or a JVM arg -Dspring.profiles.active=prod.

- why AOP.

  - AOP separate cross-cutting concerns, such as logging and security, from core business logic, enhances serial exectuion with extra logic as fucntionality, and avoids repetitive codes. 

- what java version can we use with spring boot 3 

  Java 17.

- Dependency injection types

  Three types: constructor, field, setter. 
  
  - Constructor injection is officially recommended because it ensures all beans are instantiated at boot-up, preventing NullPointerExceptions, and eases unit testing (you can pass mock/Mockito objects directly). 

  - Setter injection helps when you have optional dependencies

  - Field injection is not recommended because it will be difficult to test without Spring Boot.

- how you implement exception in springboot

  - In java, there are 2 types of exception: runtime vs compile time. Compile time checked exceptions force me to wrap in try-catch or declare with throws. Run time exceptions need me to extend RunTimeException to handle. On top of the built-in ones, I define custom exceptions for my business logic. For example, resourcenotfound exception. For each custom exception, carry an error code and a clear message.

  - In spring, exception handling is implemented via @RestControllerAdvice with @ExceptionHandler by having a global exception handler class.  Global exception handler helps customized runtime exceptions. For example, you can have resourcenotfound exception with It returns a consistent response — the HTTP status code plus a structured error body with the message and a timestamp. For example, my ResourceNotFoundException maps to a 404, a validation failure maps to a 400, and anything unexpected falls back to a 500.

  - This is really one of the two AOP styles in Spring — @RestControllerAdvice is the controller-layer way of intercepting cross-cutting concerns, the other being the @Aspect plus pointcut style. So global exception handling keeps my controllers clean and gives the front end a consistent error contract.

- what spring boot version you used

  I've had the experience of working with both Spring Boot 2 and 3.

  Spring Boot 2 is on Java 8+ and Spring Boot 3 is compatible since Java 17. Spring Boot 3 also moved from javax to jakarta namespaces.

- what is @EnableAutoConfiguration

  It loads all the jar packages you imported in the Maven pom.xml into the Spring container — when you add @SpringBootApplication to the boot class, it scans and loads those third-party dependencies so the app runs without manual configuration.

- how to stop auto configuration in spring boot

  use the exclude attribute inside @SpringBootApplication to exclude a specific class or a whole package path. 

  Or the spring.autoconfigure.exclude property in application.properties

- advantages of Spring boot Framework

  - Auto-configuration
  - Embedded Tomcat and acuator
  - Starter dependencies

- How do you customize an annotation

  via meta-annotations

  - @Target — where the annotation can be applied.
  - @Retention — when it stays valid (compile time, load/class time, or runtime).
  - @Documented - Integrates the custom token into generated Javadoc
  - @Inherited - whether subclasses inherit it,  
  - @Repeatable - apply the same annotation multiple times at one location with different values.