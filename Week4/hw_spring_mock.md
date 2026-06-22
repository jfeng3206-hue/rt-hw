**Singleton -> Lazy Loading + Eager Loading**

Singleton has only one instance in the global context. Saves memory, avoids conflict.

Singleton Implementation

  - Eager Loading: Thread-safe by nature; final private static instance + static getInstance()
  - Lazy Loading: Needs volatile key word, volatile prevents instruction reordering, so the object is fully instantiated and allocated in memory before the reference is returned. double check ==null, and synchornized. First check for performance, second check for thread safety.

Where did you use Singleton in your project?
 
  - Distinction: the Singleton design pattern (actively implemented) vs the singleton bean scope (Spring's default). 
  - If about the pattern: use it for things expensive to create and meant to be shared once — a DB connection pool, a thread pool, or loading a wide, fixed reference table into memory once instead of rebuilding per request. Same reason in all cases: one shared instance, created once, to avoid wasting resources.


**Strategy Design Pattern**

- What: The Strategy Design Pattern is a behavioral software design pattern that defines a family of algorithms, encapsulates each one inside a separate class, and makes them interchangeable at runtime.

- Why: allows an application to dynamically change its behavior or execution logic without modifying the core context code, adhering strictly to the Open/Closed Principle.

- User Case:

```
// 1. The strategy — the shared contract every payment method implements
public interface PaymentStrategy {
    PaymentResult pay(Order order);
    PaymentMethod getMethod();          // what this strategy handles
}

// 2. One concrete strategy per method, each a Spring bean
@Component
public class CreditCardPaymentStrategy implements PaymentStrategy {
    @Override public PaymentResult pay(Order order) { /* card gateway */ }
    @Override public PaymentMethod getMethod() { return PaymentMethod.CREDIT_CARD; }
}

@Component
public class PayPalPaymentStrategy implements PaymentStrategy {
    @Override public PaymentResult pay(Order order) { /* PayPal SDK */ }
    @Override public PaymentMethod getMethod() { return PaymentMethod.PAYPAL; }
}

// 3. The context — Spring injects all strategy beans; we index them by method
@Service
public class PaymentService {

    private final Map<PaymentMethod, PaymentStrategy> strategies;

    public PaymentService(List<PaymentStrategy> beans) {
        this.strategies = beans.stream()
            .collect(Collectors.toMap(PaymentStrategy::getMethod, Function.identity()));
    }

    public PaymentResult process(Order order) {
        PaymentStrategy strategy = strategies.get(order.getPaymentMethod());
        if (strategy == null) {
            throw new UnsupportedPaymentException(order.getPaymentMethod());
        }
        return strategy.pay(order);
    }
}
```
**Mock Questions Review**

Link: https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/mock-practice-0619.mov

- What is singleton design pattern

 use it for things expensive to create and meant to be shared once — a DB connection pool, a thread pool, or loading a wide, fixed reference table into memory once instead of rebuilding per request. Same reason in all cases: one shared instance, created once, to avoid wasting resources.

- where can we set CORS (backend or frontend or both).

    CORS = Cross-Origin Resource Sharing — by default, apps on a different URL (or same URL, different port) can't call each other. Always handle it on the backend via a whitelist of allowed IP/URL/port. In a microservice setup, only the API gateway is public (everything else sits on a VPC/local network), so each backend service only needs to whitelist the API gateway's IP(s) — not every user browser. This is true for every backend regardless of language (Java, Python, Node, PHP).

- can you write hint in hibernate

    Yes — Query hints in Hibernate are directives passed to the query engine to fine-tune execution, performance, or behavior.
        - **JPA standard:** `query.setHint("jakarta.persistence.query.timeout", 2000)` or declaratively via `@QueryHint` inside a `@NamedQuery`.
        - **Hibernate-specific:** `org.hibernate.readOnly`, `org.hibernate.fetchSize`, `org.hibernate.comment`, plus builder methods like `setReadOnly()`, `setFetchSize()`, `setCacheable()`.
    But they're not commonly used in real projects. For complex SQL, prefer native SQL to tune; adding an index or a cache layer is more direct than the Hibernate hint API.

- monolithic vs microservices
    - Monolith: one giant app, all business logic + a single DB, no communication cost. Good for vertical scaling only; poor flexibility, scalability, and fault tolerance; hard to extend.
    - Microservice: business sliced into modules, each fairly independent with its own DB, supported by API gateway / config server / service discovery / load balancer / message queue / monitoring. Scalable, flexible, fault-tolerant.

- will you choose stored procedures or java hibernate logic

    Prefer Java/Hibernate logic. The workload then runs on your backend server instead of the DB server — if the backend goes down it's recoverable, but if the DB goes down you can lose data/state and may not recover. Java logic is also flexible, portable, and easy to maintain, whereas stored procedures are vendor-locked (PostgreSQL syntax may not run on Oracle), hurting portability and maintainability.

- I have table person, two columns, one is name another is age, I want to give me one records that tell me the name of the person and also the age of the oldest person

    SELECT name, age FROM person ORDER BY age DESC LIMIT 1;

- SQL coding:given order table and customer table, find largest price in 10 years and return price + customer name

     Need to clarify if order holds a foreign key customer_id -> customer. Customer and order are one to many relationships. The many side keeps the FK.
    Then use inner JOIN and WHERE to select order data

- what annotations and configurations you did in eureka in spring boot

    @EnableEurekaServer placed at the same level as @SpringBootApplication on the bootup class. 
    Eureka is the service discovery registry. In application.properties/yml you configure: spring application name, server port, and the Eureka server URL + port — this tells the app where to register.
   
- what is your responsibility in the microservices?

    Develop/maintain features (UI + backend), support the architecture at runtime, debug via monitoring (Grafana, Kibana, Elasticsearch for logs, Splunk for timestamp queries), check the API gateway, integrate Spring Security + OAuth2 and a rate limiter on the gateway.

- if we have microservices calling each other a -> b -> c , and if some of them are returning 500 / errors, what should we do

    1. Stop the bleeding first by redirecting to standby servers and backup DB. Use Kafka with a retention policy to help buffer/persist messages
    2. Diagnose: find the uppermost broekn service. Reboot/restart the broken cluster to fix, then read logs from the topmost broken service via monitoring tools to find the originating requests. Sometimes it's not your code — the infra is down xs

- how do you secure communication in microservice

    1. Use HTTPS instead of HTTP. HTTPS = HTTP + SSL: asymmetric crypto + certificate exchange between client and server to block man-in-the-middle decoding.
    2. Expose only the API gateway publiicly, keep DB, cache and web services on the local network/VPC
    3. Keep HTTPS on internal module-to-module traffic too, in case the gateway is breached
    4. On the gateway handle both authentication and authorization


- SQL: join, group by and count

    JOIN (inner) → records existing in both tables; left/right join → keep records from one side.
    GROUP BY → categorize by a column (e.g. employees per department).
    COUNT → aggregation, used with GROUP BY (e.g. head count per department: COUNT(*) ... GROUP BY department).


