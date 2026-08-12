# Interceptors vs. Filters in Spring

In Spring, **Interceptors** and **Filters** are used to intercept HTTP requests and responses to perform tasks before the request reaches the controller or after the response is returned. Both serve similar purposes, but they occupy different places in the request/response lifecycle and provide different levels of access.

---

## Interceptors
* **Framework Level:** Part of Spring MVC's `HandlerInterceptor` interface.
* **Scope:** Applied specifically to handler methods (controllers).
* **Lifecycle Hooks:** They allow you to intercept the request:
  1. **`preHandle()`:** Before it reaches the controller.
  2. **`postHandle()`:** After the controller has processed the request.
  3. **`afterCompletion()`:** After the view is rendered (or response generated), but before sending it to the client.

---

## Filters
* **Servlet Level:** Part of the standard Java Servlet specification (`jakarta.servlet.Filter`).
* **Scope:** Applied globally to every request before it even reaches the Spring `DispatcherServlet`.
* **Lifecycle Hooks:** They sit between the client and the servlet container, allowing them to handle and transform requests and responses before and after the entire servlet request processing lifecycle.

---

## Key Differences Summary

| Feature | Filter | Interceptor |
| :--- | :--- | :--- |
| **Origin** | Java Servlet Specification | Spring Framework |
| **Execution Point** | Before `DispatcherServlet` | After `DispatcherServlet`, before Controller |
| **Scope** | All incoming HTTP requests | Specific Spring Controller endpoints/routes |
| **Spring Context Access** | Limited (outside Spring context unless wrapped) | Full access to Spring Beans and application context |
| **Common Use Cases** | Request logging, CORS, Compression, Request Wrapping | Fine-grained Auth checks, Controller Performance Metrics, Model Attributes |
