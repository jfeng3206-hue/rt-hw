# Setting Up Eureka in Spring Boot — Step by Step

## Prerequisites & versions

- used **Java 8+** and **Spring Boot 1.5.x** in this video
---

## PART A — The Eureka Server (the registry)

Enable the server on the main application class:
```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

Configure `application.yml`:**
```yaml
server:
  port: 8761            # default Eureka port

eureka:
  client:
    register-with-eureka: false   # the server shouldn't register with itself
    fetch-registry: false         # nor pull a registry from anyone
  instance:
    hostname: localhost
```

Run and  open the dashboard at **http://localhost:8761** to confirm it's up

---

## PART B — The Eureka Clients (Product, Order, Payment, + Shopping)

Add the client dependency

Add the client annotation

Configure each service's `application.yml`:
```yaml
spring:
  application:
    name: product-service      # THIS is the ID it registers under (used by callers)

server:
  port: 9090                   # unique per service (e.g. 8081/8082/8083)

eureka:
  client:
    register-with-eureka: true   
    fetch-registry: true        
    service-url:
      defaultZone: http://localhost:8761/eureka/   # where to send heartbeats
```
Do the same for  `payment-service` , and `shopping-service`

---

## PART C — Service-to-service calls 

Add @RestController and autowire the RestTemplate

```
@RestController
public class ShoppingController {

    @Autowired                       
    private RestTemplate restTemplate;

    @GetMapping("/amazon-payment")
    public String invokePaymentService(@PathVariable int price) {
        String url = "https://PAYMENT-SERVICE/payment-provider/payNow/"+price;
        return restTemplate.getForObject(
            url, String.class);
    }
}
```