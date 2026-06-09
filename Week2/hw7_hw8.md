## Homework7

- write out the optimized Singleton Version and explain each line of code

```
  class LazySingleton{
        
        //private static volatile instance
        // instance must be volatile.
        //volatie prevents instruction ordering and thread stability
        // Without volatile, 
        // the CPU or compiler can reorder these steps, 
        // causing the assignment (Step 3) to happen before the constructor runs (Step 2).
        // If this happens, another thread checking the singleton will see that the instance is no longer null and will attempt to use it.
        private static volatile LazySingleton instance;
        
        // private constructor
        private LazySingleton(){}
        
        
        //static getInstance method
        public static LazySingleton getInstance(){
            //First check:
            //Let every thread pass through quickly once the singleton is built
            if(instance==null){
                //Need synchronized keyword to ensure thread safety: only 1 thread at a time
                synchronized(LazySingleton.class){
                    //Second check:
                    //catches the few threads if they come at the same moment
                    if(instance==null){
                        instance = new LazySingleton();
                    }
                }
            }
            return instance;
        }
    }

```

- what is reflection

    Reflection in java allows a program to inspect and manipulate classes, methoods, fields and constructors at runtime, even when their details are unknown at compile time.

    Reflection API:

        - Class<?>: 
        ```
           Class<?> c1 = Test.class;
            Class<?> c2 = Class.forName("Test");
            Test obj = new Test();
            Class<?> c3 = obj.getClass();
        ```

        - Field: bypass `private` access restrictions 
            ```
                Test obj = new Test();
                Field f = Test.class.getDeclaredField("message");
                f.setAccessible(true);
                System.out.println(f.get(obj));
            ```

        - Method: method invoked at runtime by their names
            ```
                Test obj = new Test();
                Method m = Test.class.getMethod("show");
                m.invoke(obj);
            ```

        - Constructor: call a constructor at runtime instead of using `new` keyword
            ```
                Constructor<Test> constructor = Test.class.getConstructor();
                constructor.newInstance();
            ```


- what are http status code, 200/ 201/202/ 204/ 307/ 301/ 400/ 401/ 403/ 404/ 500, explain them by your own words

    - 2xx status codes: successfully accepted request
        - 200 OK: standard signal for a successful transaction
        - 201 CREATED: The request succeded and resulted in a new resource built
        - 202 ACCEPTED: The request is accepted for processing, but execution has not been completed
        - 204 NO CONTENT: The request was successful but there is no data payload to return in the response body.

    - 3xx status codes: redirection - the client must take extra steps usually by calling a different URL to finish the request.
        - 301 MOVED PERMANENTLY: the target resource has been assigned to a new permanent web address
        - 307 Temporary Redirect: a redirect that forces the browser to re-send the exact same request method to the temporary URL

    - 4xx status codes: client side error
        - 400 Bad Request: the server cannot process the request because of client error or bad syntax
        - 401 Unauthorized: the request has no valid authentication credentials to the resource
        - 403 Forbidden: the request has no permission to view/access the resource 
        - 404 Not Found: the most common. the server cannot find the requested URL resource

    - 5xx status codes: server side error
        - 500 Internal Server Error: a generic message for unexpected condition or software crash on the server.

- what is http  

    It is a stateless, client-server protocol that defines how clients and servers communicate over the web — typically a client sends a request and the server returns a response. 
    
    It runs on top of TCP (and TLS for HTTPS).

    It is based on client-server model: 
    
    a client sends out a request, which contains a method, an URL,and some headers; 
    
    a server responds with status codes, headers and requested data. 

    It is stateless, which means each request is independent and self-contained.

- what is get, post, put, delete, patch method

    - GET: 

        Read/retrive the resource without modifying it. 
        Idempotent.
        The only safe method, which means it doesn't modify data on the server side. 

    - POST: 

        Create a new resource by submitting the data. The server assigns the ID/URI.
        not idempotent or safe. Each call creates a new resource.

    - PUT: 
    
        Create or replace fully. Update an existing resource by replacing it entierly with the payload.
        idempotent. 

    - DELETE: 
    
        Remove the source at the URI.
        Idempotent

    - PATCH: 

        Partial Update. Modify only the sent fields, not the entire resource.
        not guaranteed idempotent. 

- post vs patch

    - POST: NOT IDEMPOTENT. create a new resource. The server will assign an URI/ID. 
    - PATCH: NOT guaranteed idempotent. Replace partial values for a KNOWN URI.

- post vs put

    - POST: NOT IDEMPOTENT. create a new resource. The server will assign an URI/ID. 
    - PUT: IDEMPOTENT. can replace a KNOWN URI.


- What is idempotent, which http method is idempotent?

    Idempotent means if a request is sent multiple times the result stays unchanged. 
    
    It is important because it determines whether a client can safely retry after a network error.


    Idempotent methods: GET, PUT, DELETE

## Homework8

- TCP 3 way handshaking

    The process that establishes a TCP connection before any data is sent.

    1. Client sends a SYN packet with its initial sequence number, asking to open a connection
    2. Server replies with SYN-ACK: acknowledges the client's SYN and sends its own SYN
    3. Client acknowledges the server's SYN by ACK. Connection established.

- TCP vs UDP

    They are both transport-layer protocols.

    - TCP: handshake first to establish connection. guaranteed delivery & retransmit lost packets. packets arrive in order. more overhead & slower.

    - UDP: connectionless (send only) best-effort, no guarantee reliability. no order guarantee. faster & lightweight.

- what is Tomcat

    Tomcat is a servlet container that runs Java web applications.

    During runtime, it takes an HTTP request, turns it into a Java Servlet call, run the code and turns the result back into a HTTP response. 

    Spring Boot embeds Tomcat by default. 

- what are the basic components for tomcat

    Tomcat nests components: a Connector handles network I/O on a port and parses HTTP; the Engine (Catalina) processes and routes the request; Host is a virtual host; and Context is an individual web app. All wrapped under a Service and the top-level Server.

    - Server: Tomcat instance (the whole thing)
    - Service: groups one ore more connectors with one engine
    - Connector: handles the actual network I/O; listens on a port and parses incoming protocol (e.g. HTTP/1.1) converts requests into objects the engine can process
    - Engine: the request-processing core; routes requests to the right host/app
    - Host: a virtual host (domain name)
    - Context: a single deployed web application

- what is web server

    A web server handles HTTP requests and returns response. It serves static files directly and passes dynamic requests to the application layer.

    For example, Nginx and Apache

- what is 3 tire architecture

    Three-tier splits an app into presentation (UI), application (business logic), and data (database) layers. 
    
    The benefit is separation of concerns — each tier scales and changes independently, so I can swap the DB or redesign the frontend without touching the logic layer.


- what is OSI Model, what is each layer doing

    OSI Model standardizes network communication into 7 layers.

    1. Physical: cables/signals/hardware
    2. Data Link: MAC addressing, farming
    3. Network: Routing and logical addressing across newtorks (IP & routes)
    4. Transport: end-to-end delivery, reliability, segmentation (TCP vs UDP)
    5. Session: session management
    6. Presentation: translation, encryption, cpmressiong (TLS/SSL, encoding)
    7. Application: interface to the user/app; network services (HTTP)

