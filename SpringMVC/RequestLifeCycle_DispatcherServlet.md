# Spring Boot Request Lifecycle: Step-by-Step

Here’s what happens under the hood, from the moment a user sends a request to your application:

1. **Request Initiation**  
   A browser, mobile app, or any HTTP client sends a request to your Spring Boot application (e.g., `GET /api/products`).

2. **Embedded Server Reception**  
   Spring Boot uses embedded servers (like Tomcat, Jetty, or Netty) to listen for incoming HTTP traffic on a specified port (often `8080`).

3. **DispatcherServlet Activation**  
   Once the server receives the request, it is passed to Spring’s `DispatcherServlet`, the central component of Spring MVC. This acts as the "front controller" for all incoming requests.

4. **Handler Mapping**  
   The `DispatcherServlet` checks with the `HandlerMapping` components to determine which controller method should handle the request based on the URI, HTTP method, headers, etc.

5. **Handler Adapter & Controller Invocation**  
   The selected `HandlerAdapter` calls your controller method — for example, `@GetMapping("/products")`.

6. **Return Value Processing**  
   After the controller finishes execution, it returns a view or response data (like a `ResponseEntity`, JSON, or HTML template).

7. **View Resolver / Message Converter**  
   If the return type is a view (like Thymeleaf or JSP), Spring delegates it to a `ViewResolver`. If it’s raw data (e.g., JSON), the `HttpMessageConverter` converts it into the appropriate response body.

8. **Response Sent Back**  
   The HTTP response is returned to the client, and the lifecycle ends for that request.

---

## Visual Diagram: Request Lifecycle

```text
+--------------+               +-----------------+               +-------------------+
|  HTTP Client | --(Request)-> | Embedded Server | ------------> | DispatcherServlet |
|  (Browser)   |               | (e.g., Tomcat)  |               | (Front Controller)|
+--------------+               +-----------------+               +-------------------+
       ^                                                                   |
       |                                                                   v
       |                                                         +-------------------+
       |                                                         |  HandlerMapping   |
       |                                                         +-------------------+
       |                                                                   |
       |                                                                   v
       |                                                         +-------------------+
       |                                                         |  HandlerAdapter   |
       |                                                         +-------------------+
       |                                                                   |
       |                                                                   v
       |                                                         +-------------------+
       |                                                         |    Controller     |
       |                                                         | (@GetMapping etc.)|
       |                                                         +-------------------+
       |                                                                   |
       |                                                                   v
       |       +-----------------------+                         +-------------------+
       |       | ViewResolver /        | <---------------------- |   Return Value    |
       +------ | HttpMessageConverter  |                         |  Processing       |
   (Response)  +-----------------------+                         +-------------------+
