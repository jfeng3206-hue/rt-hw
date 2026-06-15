# Homework 13: mock questions review + recording 0612 + restful endpoint design from scratch 

**Restful API Design Mock Video Link** 

https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/restful-api-design.mov

Scripts:

@RestController registers this as a controller bean and tells Spring every method returns a response body, not a view name. @RequestMapping at the class level defines the parent URL.

GET: @PathVariable binds the id from the URL path. The resource id belongs in the path; query params are for filtering.

POST: @RequestBody deserializes the JSON into my request DTO and @Valid triggers the validation rules I defined. I return 201 Created, not 200, because a new resource was created.

PUT: Path variable for which product, request body for the new state, 200 OK.
DELETE: Path variable for the id, and 204 No Content because there's no body to return.

**Mock Video Link** 

https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/spring-boot-mock-practice.mov

- introduce what is Spring Framework

  If you mean the core framework, it really comes down to two modules: IoC and AOP. If you mean the ecosystem, I'd describe the evolution from Spring → Spring MVC → Spring Boot

  - Core Modules:
  
    IoC (Inversion of Control) — the container creates and manages objects as beans instead of me instantiating them manually — and AOP (Aspect-Oriented Programming) for cross-cutting logic.

  - Evolution:
    Spring — powerful but everything is manually configured: lots of boilerplate, no embedded Tomcat, no actuator.
    
    Spring MVC — started introducing annotations like @RequestMapping, but still boilerplate-heavy.
    
    Spring Boot — pure annotation-driven with auto-configuration; the boilerplate is extracted away so you focus on the controller layer.

- spring boot version you used

  I've had the experience of working with both Spring Boot 2 and 3.

  Spring Boot 2 is on Java 8+ and Spring Boot 3 is compatible since Java 17. Spring Boot 3 also moved from javax to jakarta namespaces.

- how do you define profile

  Use profile-specific files: application-dev.properties / application-dev.yml, etc.
  Annotate a bean or class with @Profile("dev") so it's only registered for that profile. You can also negate: @Profile("!dev") means register this bean whenever the profile is not dev.
  Set the active profile via JVM args or environment variables.

- what discovery service impl you used before

  I've used Eureka. 
  
  on the @SpringBootApplication boot-up class, add the enable-discovery annotation (e.g. @EnableEurekaClient / @EnableDiscoveryClient) so the application registers itself into Eureka.

  in a microservices architecture, service discovery is the centralized "information board" that monitors each Spring Boot application's status. When an app goes down, it deregisters; so at any time you know all the live, online servers serving the architecture.

- what is aop

  Aspect Oriented Programming.
  Is One of the two important modules in Spring Boot that handles cross-cutting logic without scattering duplicate codes. We usually use for logging and security handling.

  We use @Aspect to mark the class as an aspect, @Pointcut to dfine where the logic applies, and advice annotation such as @Before, @After, @AfterReturning, @AfterThrowing, @Around to define when to apply the logics.

  Two styles in practice:
  1. Global advice on the controller layer — @RestControllerAdvice (e.g. for global exception handling).
  2. @Aspect + @Pointcut + advice for general cross-cutting concerns.

- how to write spring boot to call from frontend to backend and save data to database

  1. Request leaves the browser (front end / UI).
  2. Controller layer — the RESTful endpoint. First thing the request hits. I design the endpoint here (HTTP method + URL + @RequestParam / @PathVariable / @RequestBody).
  3. Service layer — business logic. I use the interface + implementation pattern so I can swap implementations later.
  4. DAO layer — persistence. Nowadays I use an ORM, Spring Data JPA for SQL; for NoSQL there's spring-data-mongo, or the AWS Dynamo libraries.
  5. Wrap the result in a ResponseEntity (payload + HTTP status code), return up through the controller, Tomcat returns it to the browser, and the UI renders.

- Describe Spring MVC

  Spring MVC is the model-view-controller architecture built around the DispatcherServlet, combined with the three-tier layer (controller/service/repository)

  When a request hits the server, DispatcherServlet receives the request and acts as a front controller to hand the request to HandlerMapping, which routes the request to the correct controller. The controller then returns a result, either an HttpMessageConverter serializes an object to JSON for REST, or a ViewResolver renders a view for traditional MVC. The response goes back out through the DispatcherServlet and back to the client.

- how do you validate input data in spring boot

  1. Define the validation rules on the model/dto/entity via annotations: @Notnull, @Email, @NotEmpty, @min, @max, @size
  2. Enable validation via the @valid annotation on the controller

- Spring boot actuator

  Actuator is a production-ready plugin to expose monitoring endpoints and watch the application's status.

  Use actuator:
  1. Import the dependency - spring-boot-starter-actuator.
  2. Configure which endpoints are expose in application.properties/.yml with the best practice of minimum exposure. (Do not include all in prod etc.)
  3. Persist metrics in a time-series databse Prometheus
  4. Configure Promethheus as the data source for visualization tool like Grafana dashboard.

- how does spring mvc work

  1. DispatcherServlet receives the request.
  2. It asks HandlerMapping to find the right controller method, matched by URL + HTTP method.
  3. It calls that controller method.
  4. The controller returns data — a view name, or JSON via @RestController (@ResponseBody).
  5. Through the view resolver → the response (e.g. JSON response body) goes back to the front end.

- what is controller how you use controller how you implement controller

  The controller is the topmost layer of the three-tier architecture, responsible for exposing RESTful endpoints to the UI.

  Two types: @Controller and @RestController.

  How I implement it: use @RestController to register the bean, then design the RESTful endpoints inside it. (Response + Request. Response includes: method, url, reqeust param, request header, path variable; Request includes HttpCode+response body (payload in json))

  Optional add for robustness: layer in exception handling (@RestControllerAdvice)

- what is webflux? Have you used it in your project

  WebFlux is the async, reactive programming style for Spring Boot — a non-blocking alternative to Spring MVC.
  Built on the Reactor library. Requests flow through channels handled by a worker group (like a thread pool), executed in parallel.
  Mono for a single object, Flux for a group of objects — you subscribe to them, similar to Java 8 Stream APIs.
  But since Tomcat added async support and with Java 21 virtual threads boosted thread performance. With virtual threads+ modern Tomcat, reactive is less common.

- how you connect the database in springboot
  1. Add the dependency — spring-data-jpa.
  2. Configure the data source — URL, database name, connection pool size, connection timeout. 
    Two styles:
      Configuration-based — in application.properties / .yml.
      Programmatic — define the configuration as a Spring @Bean and establish the connection pool in code. Useful when you can't afford to reboot the app (hot-fix): expose an endpoint that re-reads the config file at runtime.
  3. @Value lets you pull a field from the properties file into a Java variable — the field name must match exactly.
  4. Multiple data sources in one app — use @Qualifier (inject by name) or @Primary (default).

- how do you handle global exception in spring boot

  I use a global exception handler class annotated with @RestControllerAdvice plus @ExceptionHandler."
  This customizes runtime exceptions — e.g. a ResourceNotFound returning a 404 so the user/dev knows exactly what failed.
  @RestControllerAdvice is part of the AOP style (global advice on the controller layer).

- Spring boot annotation

  - @SpringBootApplication (= @EnableAutoConfiguration + @SpringBootConfiguration + @ComponentScan).
  - Bean declaration by layer: @RestController / @Controller, @Service, @Repository, and @Component for generic beans.
  - Third-party objects: @Configuration on the class + @Bean on the method.
  - Scope: @Scope — singleton (default), prototype, request, session.
  - Dependency injection: @Autowired, @Qualifier / @Primary (when multiple candidates), @Lazy (to break a circular dependency).
  - REST design: @RequestMapping (class level for a parent URL), @GetMapping / @PostMapping / @PutMapping / @DeleteMapping, @RequestParam, @PathVariable, @RequestHeader, @RequestBody.
  - Validation: @Valid, plus field rules @NotNull, @NotEmpty, @NotBlank, @Size, @Email, @Max, @Min.

- how Spring IOC work , all annotations and injection and bean types
  
  IoC means the container instantiates and manages objects as beans instead of me creating them manually.

- how many ways to inject bean in spring and which one we use most

  - Constructor injection — recommended; dependencies are explicit, easier to unit test (pass a mock without Spring), prevents NPE, and fails fast at boot.
  - Field injection.
  - Setter injection.

- by name vs by type

  Spring always starts by injecting by type. If there are multiple beans of the same type, it falls back to by name.

  @Primary — annotated on the bean itself; makes it the default.
  @Qualifier — annotated at the injection point; picks the bean by name.

- why constructor injection

  NPE prevention.
  Easier unit testing — inject mocks directly without bootstrapping Spring.
  Fail-fast vs fail-safe — constructor injection is fail-fast: if a bean can't be created properly, the application fails at boot time, so you find out immediately. With field injection the app boots fine and you only hit the NPE at runtime.

- what java version can we use with spring boot 3
  Java 17 and above.
  
- what is dispatcherservlet
  Front controller that receives the request and hands the requist to HandlerMapping to find the correct controller based on URL and Http Method