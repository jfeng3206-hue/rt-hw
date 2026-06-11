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
  