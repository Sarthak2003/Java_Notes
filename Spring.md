Spring makes programming Java quicker, easier, and safer for everybody. Spring’s focus on speed, simplicity, and productivity has made it the [world's most popular](https://www.jetbrains.com/lp/devecosystem-2023/java/) Java framework.

Spring manages your objects so you don't have to wire them manually. It reduces coupling and lets you focus on business logic.


**Inversion of Control(IoC) & Dependency Injection(DI)**

[Inversion of Control](https://www.baeldung.com/cs/ioc) is a principle in software engineering which transfers the control of objects or portions of a program to a container or framework. We most often use it in the context of object-oriented programming.

**If we want to add our own behavior, we need to extend the classes of the framework or plugin our own classes.**

The advantages of this architecture are:

- decoupling the execution of a task from its implementation
- making it easier to switch between different implementations
- greater modularity of a program
- greater ease in testing a program by isolating a component or mocking its dependencies, and allowing components to communicate through contracts



**Dependency injection** is a pattern we can use to implement IoC, where the control being inverted is setting an object’s dependencies.

Connecting objects with other objects, or “injecting” objects into other objects, is done by an assembler rather than by the objects themselves.

Instead of your class creating its dependencies (`new`), Spring creates them and injects them. This makes your code testable and flexible.

**Spring vs Spring Boot**

**Simply put, the Spring framework provides comprehensive infrastructure support for developing Java applications**.

It’s packed with some nice features like Dependency Injection, and out of the box modules like:

- Spring JDBC
- Spring MVC
- Spring Security
- Spring AOP
- Spring ORM
- Spring Test


**Spring Boot** is basically an extension of the Spring framework, which eliminates the boilerplate configurations required for setting up a Spring application.

**It takes an opinionated view of the Spring platform, which paves the way for a faster and more efficient development ecosystem**.

Here are just a few of the features in Spring Boot:

- Opinionated ‘starter’ dependencies to simplify the build and application configuration
- **Embedded server** to avoid complexity in application deployment
- Metrics, Health check, and externalized configuration
- Automatic config for Spring functionality – whenever possible

Spring Boot is Spring with auto-configuration, embedded servers, and starter dependencies – it's Spring "convention over configuration".


**Problem Spring Solves**

- Traditional Java EE: heavy XML configuration, manual object creation, lifecycle management.
    
- Spring: IoC container manages objects (beans). You just define beans, Spring wires them.
    
- Result: Loose coupling, easier testing, cleaner code.


**IoC Container & DI**

- **IoC Container** = `ApplicationContext`. It creates and manages beans.
    
- **Dependency Injection (DI)** = Spring injects dependencies into your objects.
    
- **Types of DI**:
    
    - **Constructor Injection** (preferred): dependencies as constructor parameters. Final fields, immutable, easy to test.
        
    - **Setter Injection**: optional dependencies via setter methods.
        
    - **Field Injection** (`@Autowired` on field): simple but hides dependencies, hard to test (avoid).
        
- **Why constructor injection?** Ensures required dependencies are present, supports immutability, and is testable without Spring.


**Spring vs Spring Boot**

- **Spring**: framework with many modules; requires configuration (XML or Java config).
    
- **Spring Boot**: built on Spring; provides auto-configuration (based on classpath), embedded servers (Tomcat), starter dependencies (e.g., `spring-boot-starter-web`), and production-ready features (actuator). Goal: "Just run".

- `pom.xml` includes `spring-boot-starter-web` (transitively includes Tomcat, Spring MVC, Jackson).
    
- Main class annotated with `@SpringBootApplication`. This is a combination of `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`.



## Annotations & Bean Lifecycle

**Stereotype Annotations**:

In most typical applications, we have distinct layers like data access, presentation, service, business, etc.

Additionally, in each layer we have various beans. To detect these beans automatically, **Spring uses classpath scanning annotations**.

Then it registers each bean in the _ApplicationContext_.

- _@Component_ is a generic stereotype for any Spring-managed component.
- _@Service_ annotates classes at the service layer.
- _@Repository_ annotates classes at the persistence layer, which will act as a database repository.

**The major difference between these stereotypes is that they are used for different classifications.** When we annotate a class for auto-detection, we should use the respective stereotype.

### **_@Component_**[](https://www.baeldung.com/spring-component-repository-service#component)

**We can use @Component across the application to mark the beans as Spring’s managed components**. Spring will only pick up and register beans with _@Component,_ and doesn’t look for _@Service_ and _@Repository_ in general.

They are registered in _ApplicationContext_ because they are annotated with _@Component_:

```java
@Component
public @interface Service {
}
```

```java
@Component
public @interface Repository {
}
```

_@Service_ and _@Repository_ are special cases of _@Component_. They are technically the same, but we use them for the different purposes.


### **_@Repository_**
**_@Repository_’s job is to catch persistence-specific exceptions and re-throw them as one of Spring’s unified unchecked exceptions**.

For this, Spring provides _PersistenceExceptionTranslationPostProcessor_, which we are required to add in our application context (already included if we’re using Spring Boot):

```xml
<bean class=
  "org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>
```

This bean post processor adds an advisor to any bean that’s annotated with _@Repository._


### **_@Service_**
**We mark beans with @Service to indicate that they’re holding the business logic**. Besides being used in the service layer, there isn’t any other special use for this annotation.


- **Key takeaway**: They are all specializations of `@Component`. They are used for auto-detection via classpath scanning. `@Repository` adds translation for persistence exceptions; `@Controller` is for web controllers; `@Service` is just a marker for service layer.


## **Bean Scopes**

## **Singleton Scope**

When we define a bean with the _singleton_ scope, the container creates a single instance of that bean; all requests for that bean name will return the same object, which is cached. Any modifications to the object will be reflected in all references to the bean. This scope is the default value if no other scope is specified.


## **Prototype Scope**

A bean with the _prototype_ scope will return a different instance every time it is requested from the container. It is defined by setting the value _prototype_ to the _@Scope_ annotation in the bean definition:

- **Key takeaway**: `singleton` (default) – one instance per Spring container; `prototype` – new instance each time requested.


**SPRING BEAN LIFECYCLE**
series of stages a bean goes through from its initial instantiation to its eventual destruction

- ### Container Initialization
	- The Spring IoC container starts and loads configuration metadata (XML, annotations, or Java config).
	- Bean definitions are registered, and infrastructure components (like processors) are prepared.

- ### Bean Instantiation
	- The container creates the bean object using a constructor or factory method.
	- At this stage, the bean exists in memory but dependencies are not yet injected.

- ### Dependency Injection
	- The container resolves required dependencies from the IoC container.
	- Dependencies are injected via constructor, setter, or field injection.

- ### Custom Init Method
	- The custom init() method is called once all dependencies are injected.
	- It is used to perform additional setup like initializing resources, validating properties, or starting connections.
- **Initialization Callbacks:** Custom initialization logic is executed:
    - Methods annotated with `@PostConstruct` are called first.
    - The `afterPropertiesSet()` method of the `InitializingBean` interface is called next.
    - Any custom `init-method` specified in configuration is called last.
- **BeanPostProcessor (After Initialization):** The `postProcessAfterInitialization()` method of any registered `BeanPostProcessor` is called, a stage where dynamic proxies (like those used in AOP) are often created.
- **Bean Ready for Use:** The bean is fully initialized, stored in the container's cache, and ready for use by the application.
- **Bean Destruction:** When the Spring container is shut down (not for `prototype` scoped beans), cleanup logic is executed:
	- Methods annotated with `@PreDestroy` are called first.
	- The `destroy()` method of the `DisposableBean` interface is called next.
	- Any custom `destroy-method` specified in configuration is called last.