# Homework6

### Use your own words to describe the following terms

- Recording Link: https://rt-mock-014467817298-us-west-2-an.s3.us-west-2.amazonaws.com/web-basics.mov

- client - server model

  The client-server model is basically a way of splitting an application into two roles that talk to each other over a network. Key idea: the separation of responsibility.

  The client: the side that makes requests - usually the browser, app, or backend service

  The server: the side that listens thoses requests & does the actual work and sends back a response.

  In HTTP protocal, the client sends an HTTP request with a verb and a URL, and the server returns a response with a status code and data. 

- application service

  The service layer in three-tier architecture is responsible for core business logics for the applications. Annotated with `@Service` in SpringBoot. 


- http request/response

  Http request/response is the basic conversation pattern between a client and a server.

  Client sends a request and the server sends back a response. 

  The request: 
    - a verb
      - Verbs include: GET/POST/PUT/PATCH/DELETE. For verbs like POST, PUT, or PATCH, there is a body that carries the actual payload, which is the data the request is sending to be inserted or updated. 

      | Method | Purpose | Safe? | Idempotent? | Cacheable? |
      | --- | --- | --- | --- | --- |
      | GET | read | Yes | Yes | Yes |
      | POST | insert | No | **No** | No |
      | PUT | full update | No | Yes | No |
      | PATCH | partial update | No | No | No |
      | DELETE | remove | No | Yes | No |
    - a URL (the address of the source/door number)
    - a protocol version, like HTTP/1.1 
    - the headers which carry metadata about the request - like what data format accepted (JSON or xml), encoding, authorization info.

  The response:
    - Status code: 2xx if succeeded (200 OK, 201 created, 204 no content),3xx redirection, 4xx client error (bad/invalid user input), 404 resource not found, 5xx if the server broke
    - Header: describing the format and encoding of what's coming back
    - Body: actual data the client asked for, or an error message.

- horizontal scaling vs vertical scaling

  - Horizontal Scaling: more server instances. Preferred because it avoids downtime during maintenance and let you rent/release capacity elastically.

  - Vertical Scaling: bigger single server (more RAM/CPU/disk). Eventually hit a physical ceiling. 
  
- load balancer

  When there are multiple identical instances in parallel, a load balancer is responsible in front to distribute incoming requests evenly to reduce latency and keep things stable under high request volume. 

- microservice, microfronted

  Microservice is an architectural style where you split the business logics into independent apps (e.g., product, payment, inventory as separate services.) Advantages: isolated failures, independent scaling. Downside: communication between separate JVMs becomes a new point of failure and adds data-handling complexity.

  Micor-frontend applies the same idea to the frontend. It breaks the big UI into smaller pieces, like product-listing, order placement, and shopping cart. Advantages: team autonomy and independent deployment. Downside: complexity and consistency. 

- database: relational database (sql database), nonrelational database ( no-sql database)

  - Relational Database: store data in tables: rows and columns with a fixed schema. Tables can reference each other through keys, so relationships can be modeled and joined. Better for data with ACID requirements.

  - Nonrelational Database: flexible schema. Can store JSON-like documents where each one can have different shape; key-value stores like Redis; wide-column stores like Cassandra. Better for data with huge volumes and high throughput. 

- api gateway

  Entry point that sits in front of the backend services and routes incoming requests to the right place. Typical handle: authentication and authorization, rate limiting, load balancing, logging and monitoring, request/response transformation or call aggregations. 

- message queue

  Lets services communicate asynchrnously by passing messages through a buffer in the middle, instead of calling each other directly. The producer drops a message onto the queue, and the consumer picks it up and processes it whenver it's ready. Producer and consumer do not talk to each other directly an don't have to be available at the same time. 


- log, monitor

  Loggin and monitoring are the two pillars of observability.

  Logging is recording discrete events as they happen. Things like DEBUG, INFO, WARN, ERROR. Help capture the status codes to help distinguish a client-side problem from a server-side one. 

  Monitoring is the higher-level, real-time view of system health, such as watching aggregate metrics (request rate, error rate, latency, CPU and memory usage) that can be visualized on dashboards. 

- deployment with AWS/Azure/GCP

  Cloud deployment means running the app on infra rented from the provider instead of buying and maintaining your own physical servers. 

- security (authentication and authorization)

  Need to use HTTPS for both. 

  Authentication: "who are you?" - identify. Since HTTP is stateless, the server doesn't remember between requests. Therefore need to carry token (JWT) in the header of every request. The server then validates the token's signature on each request. Related to 401 Unauthorized code. Usually centralized at the API Gateway. 

  Authorization: "what you are allowed to do?" - permission. Use RBAC (role-based access control) to control what the user is allowed to do. Related to 403 Forbidden code. 
  
- why testing

  - Catch bugs early
  - Prevent regressions: a regression is when a change breaks something that previously worked. 
  - CI/CD: tests are automated if one wants to deploy frequently and automatically.

  Tests include:
    - Unit tests: test one function or class on its own, with dependencies mocked out
    - Integration tests: components can work together? i.e., Service layer actually talking to the database?
    - End-to-End: testing a full user flow through the real system. 
