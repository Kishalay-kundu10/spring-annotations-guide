# Bean Annotations

Spring beans are the fundamental building blocks of a Spring application. These annotations help Spring identify, create, and manage beans in the IoC (Inversion of Control) container.

---

## Table of Contents
- [@Component](#component)
- [@Service](#service)
- [@Repository](#repository)
- [@Controller](#controller)
- [@Bean](#bean)
- [@Autowired](#autowired)
- [@Qualifier](#qualifier)
- [@Primary](#primary)
- [@Lazy](#lazy)
- [@Scope](#scope)
- [@DependsOn](#dependson)

---

## @Component

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks a class as a Spring-managed component |
| **Used On** | Class |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What is @Component and how does it work?" |

### What It Does
`@Component` is a generic stereotype annotation that tells Spring to create and manage an instance of this class as a bean in the application context. During component scanning, Spring automatically detects classes annotated with `@Component` and registers them as beans.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Component is a class-level annotation that marks a Java class as a Spring bean. It's a generic stereotype for any Spring-managed component. When component scanning is enabled (via @ComponentScan or @SpringBootApplication), Spring automatically detects @Component classes and creates bean instances in the IoC container. It's the parent stereotype annotation for @Service, @Repository, and @Controller."

### Code Example
```java
@Component
public class EmailService {
    
    public void sendEmail(String to, String message) {
        System.out.println("Sending email to: " + to);
    }
}

// Usage - Spring will auto-inject this
@Component
public class NotificationManager {
    
    @Autowired
    private EmailService emailService;
    
    public void notify(String user) {
        emailService.sendEmail(user, "Notification");
    }
}
```

### Key Points to Remember
- Generic stereotype annotation for any Spring-managed component
- Enables auto-detection through classpath scanning
- Default bean name is the uncapitalized class name (e.g., `EmailService` → `emailService`)
- Can specify custom bean name: `@Component("myCustomName")`
- Parent annotation for specialized stereotypes (@Service, @Repository, @Controller)

### Common Pitfalls
- Forgetting to enable component scanning (@ComponentScan)
- Using @Component for configuration classes (use @Configuration instead)
- Creating multiple beans of the same type without @Qualifier
- Not understanding bean scope (singleton by default)

---

## @Service

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks a class as a service layer component |
| **Used On** | Class |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What's the difference between @Component and @Service?" |

### What It Does
`@Service` is a specialized form of `@Component` that indicates the class holds business logic. While functionally identical to `@Component`, it provides better semantic meaning and allows for potential AOP (Aspect-Oriented Programming) enhancements specific to service layers.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Service is a specialization of @Component that indicates a class contains business logic and belongs to the service layer. While it functions the same as @Component technically, it provides better code readability and semantic meaning. It also allows Spring or AOP frameworks to apply service-layer-specific processing in the future. It's a best practice to use @Service for service layer classes rather than generic @Component."

### Code Example
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EmailService emailService;
    
    public User registerUser(UserDto userDto) {
        // Business logic
        User user = new User();
        user.setEmail(userDto.getEmail());
        user.setName(userDto.getName());
        
        // Save user
        User savedUser = userRepository.save(user);
        
        // Send welcome email
        emailService.sendEmail(savedUser.getEmail(), "Welcome!");
        
        return savedUser;
    }
    
    public List<User> getAllActiveUsers() {
        return userRepository.findByStatus("ACTIVE");
    }
}
```

### Key Points to Remember
- Specifically designed for service layer (business logic)
- Functionally equivalent to @Component
- Improves code readability and maintainability
- Makes code self-documenting
- Can be used for AOP pointcut expressions targeting service layer
- Best practice for separation of concerns

### Common Pitfalls
- Using @Service for repository or controller classes
- Thinking it provides additional functionality (it's just semantic)
- Putting data access logic in service layer (belongs in @Repository)
- Making service classes too fat with multiple responsibilities

---

## @Repository

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks a class as a Data Access Object (DAO) |
| **Used On** | Class |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What's special about @Repository compared to @Component?" |

### What It Does
`@Repository` is a specialized `@Component` for the persistence layer. It not only marks the class as a bean but also enables automatic exception translation - converting database-related exceptions into Spring's DataAccessException hierarchy.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Repository is a specialization of @Component that marks a class as a DAO (Data Access Object) in the persistence layer. Its key feature is automatic exception translation - it catches platform-specific database exceptions (like SQLException) and translates them into Spring's unified DataAccessException hierarchy. This makes exception handling consistent across different database vendors and provides better abstraction. It's the recommended annotation for classes that directly interact with the database."

### Code Example
```java
@Repository
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        try {
            return jdbcTemplate.queryForObject(sql, 
                new Object[]{id}, 
                new UserRowMapper());
        } catch (EmptyResultDataAccessException e) {
            // Spring's exception - translated from SQLException
            return null;
        }
    }
    
    public void save(User user) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        jdbcTemplate.update(sql, user.getName(), user.getEmail());
    }
    
    public List<User> findByStatus(String status) {
        String sql = "SELECT * FROM users WHERE status = ?";
        return jdbcTemplate.query(sql, 
            new Object[]{status}, 
            new UserRowMapper());
    }
}

// With Spring Data JPA, you just extend interfaces
@Repository
public interface UserJpaRepository extends JpaRepository<User, Long> {
    List<User> findByStatus(String status);
    Optional<User> findByEmail(String email);
}
```

### Key Points to Remember
- Designed for data access/persistence layer
- Enables automatic exception translation
- Converts vendor-specific exceptions to Spring's DataAccessException
- Works with JdbcTemplate, JPA, Hibernate, etc.
- With Spring Data JPA, just extend repository interfaces
- Makes persistence layer exception handling consistent

### Common Pitfalls
- Using @Repository for service layer classes
- Not leveraging Spring Data JPA interfaces (unnecessary boilerplate)
- Mixing business logic with data access logic
- Not handling translated exceptions properly

---

## @Controller

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks a class as a Spring MVC controller |
| **Used On** | Class |
| **Spring Module** | Spring Web MVC |
| **Common Interview Question** | "What's the difference between @Controller and @RestController?" |

### What It Does
`@Controller` is a specialized `@Component` that marks a class as a web controller in Spring MVC. It handles HTTP requests and returns view names or model data. Typically used for traditional web applications that return HTML views.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Controller is a specialization of @Component used in Spring MVC to handle web requests. It marks a class as a controller where methods handle HTTP requests using @RequestMapping or similar annotations. Unlike @RestController, methods in @Controller return view names (like JSP or Thymeleaf templates) by default. To return data directly (like JSON), you need to add @ResponseBody. It's used for traditional MVC applications where you render server-side views."

### Code Example
```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    // Returns a view name
    @GetMapping("/list")
    public String listUsers(Model model) {
        List<User> users = userService.getAllActiveUsers();
        model.addAttribute("users", users);
        return "user-list"; // Returns view name (user-list.html/jsp)
    }
    
    // Returns a view with path variable
    @GetMapping("/{id}")
    public String getUserDetails(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", user);
        return "user-details";
    }
    
    // Returns JSON data (needs @ResponseBody)
    @GetMapping("/api/users")
    @ResponseBody
    public List<User> getUsersAsJson() {
        return userService.getAllActiveUsers();
    }
    
    // Form submission
    @PostMapping("/register")
    public String registerUser(@ModelAttribute UserDto userDto, 
                              RedirectAttributes redirectAttributes) {
        userService.registerUser(userDto);
        redirectAttributes.addFlashAttribute("message", "User registered!");
        return "redirect:/users/list";
    }
}
```

### Key Points to Remember
- Used for traditional MVC web applications
- Methods return view names by default
- Works with view resolvers (JSP, Thymeleaf, etc.)
- Can return data with @ResponseBody on individual methods
- Handles form submissions and redirects
- Use @RestController for REST APIs instead

### Common Pitfalls
- Using @Controller for REST APIs (use @RestController)
- Forgetting @ResponseBody when returning JSON
- Not configuring view resolver properly
- Mixing REST and view-based endpoints in same controller

---

## @Bean

| Aspect | Details |
|--------|---------|
| **Purpose** | Declares a method as a bean producer |
| **Used On** | Method (inside @Configuration class) |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "When do you use @Bean vs @Component?" |

### What It Does
`@Bean` is a method-level annotation used inside `@Configuration` classes. It tells Spring that the method will return an object that should be registered as a bean in the application context. Used when you need more control over bean creation or when working with third-party classes.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Bean is a method-level annotation used in @Configuration classes to explicitly declare a bean. Unlike @Component which is class-level, @Bean gives you full control over object instantiation. It's essential when you need to create beans from third-party libraries (where you can't add @Component), configure beans with complex initialization logic, or create multiple beans of the same type with different configurations. The method name becomes the bean name by default."

### Code Example
```java
@Configuration
public class AppConfig {
    
    // Basic bean creation
    @Bean
    public EmailService emailService() {
        return new EmailService("smtp.gmail.com", 587);
    }
    
    // Bean with dependencies
    @Bean
    public UserService userService(UserRepository userRepository, 
                                   EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
    
    // Third-party library bean
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return mapper;
    }
    
    // Multiple beans of same type with custom names
    @Bean(name = "primaryDataSource")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/primary")
            .username("root")
            .password("password")
            .build();
    }
    
    @Bean(name = "secondaryDataSource")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/secondary")
            .username("root")
            .password("password")
            .build();
    }
    
    // Bean with initialization and destruction callbacks
    @Bean(initMethod = "init", destroyMethod = "cleanup")
    public ConnectionPool connectionPool() {
        return new ConnectionPool();
    }
}
```

### Key Points to Remember
- Used in @Configuration classes only
- Method name becomes bean name (can override with `name` attribute)
- Parameters are auto-injected by Spring
- Can specify init and destroy methods
- Essential for third-party library beans
- Allows complex bean initialization logic
- Can create multiple beans of same type with different names

### Common Pitfalls
- Using @Bean outside @Configuration class
- Not understanding the difference between @Bean and @Component
- Creating circular dependencies
- Forgetting that method name is the bean name
- Not closing resources properly (use destroyMethod)

---

## @Autowired

| Aspect | Details |
|--------|---------|
| **Purpose** | Automatically injects dependencies |
| **Used On** | Constructor, Field, Setter Method |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What is dependency injection and how does @Autowired work?" |

### What It Does
`@Autowired` enables automatic dependency injection. Spring automatically resolves and injects collaborating beans into your bean. It can be applied to constructors, fields, or setter methods. Since Spring 4.3, constructor injection for single-constructor classes doesn't require @Autowired.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Autowired enables automatic dependency injection in Spring. When Spring creates a bean, it looks at @Autowired annotations and automatically injects the required dependencies from the application context. It works by type matching - Spring finds a bean of the matching type and injects it. There are three types: constructor injection (recommended), field injection, and setter injection. Constructor injection is preferred because it makes dependencies explicit, allows immutability, and makes testing easier. If multiple beans of the same type exist, you need @Qualifier to specify which one to inject."

### Code Example
```java
// 1. CONSTRUCTOR INJECTION (RECOMMENDED - Best Practice)
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;
    
    // @Autowired optional for single constructor since Spring 4.3
    @Autowired
    public OrderService(OrderRepository orderRepository,
                       PaymentService paymentService,
                       EmailService emailService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }
    
    public Order placeOrder(OrderDto orderDto) {
        // Use dependencies
        Order order = new Order();
        paymentService.processPayment(order);
        orderRepository.save(order);
        emailService.sendOrderConfirmation(order);
        return order;
    }
}

// 2. FIELD INJECTION (Easy but not recommended for production)
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private InventoryService inventoryService;
    
    public Product addProduct(Product product) {
        return productRepository.save(product);
    }
}

// 3. SETTER INJECTION
@Service
public class NotificationService {
    
    private EmailService emailService;
    private SmsService smsService;
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    @Autowired
    public void setSmsService(SmsService smsService) {
        this.smsService = smsService;
    }
}

// 4. OPTIONAL DEPENDENCY (required = false)
@Service
public class ReportService {
    
    @Autowired(required = false)
    private PdfGenerator pdfGenerator; // May not be available
    
    public void generateReport() {
        if (pdfGenerator != null) {
            pdfGenerator.generate();
        } else {
            // Fallback logic
        }
    }
}

// 5. COLLECTION INJECTION (All beans of a type)
@Service
public class NotificationDispatcher {
    
    @Autowired
    private List<NotificationChannel> channels; // All NotificationChannel beans
    
    public void sendNotification(String message) {
        channels.forEach(channel -> channel.send(message));
    }
}
```

### Key Points to Remember
- Three types: constructor, field, setter injection
- Constructor injection is the recommended approach
- Injects by type by default
- Use `required = false` for optional dependencies
- Can inject collections of beans (List, Set, Map)
- Single constructor doesn't need @Autowired (Spring 4.3+)
- Use @Qualifier when multiple beans of same type exist

### Common Pitfalls
- Overusing field injection (makes testing harder)
- Creating circular dependencies
- Not handling optional dependencies properly
- Injecting beans of same type without @Qualifier
- Using @Autowired on static fields (doesn't work)
- Not understanding the difference between byType and byName

---

## @Qualifier

| Aspect | Details |
|--------|---------|
| **Purpose** | Specifies which bean to inject when multiple candidates exist |
| **Used On** | Field, Parameter, Method |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "How do you handle multiple beans of the same type?" |

### What It Does
`@Qualifier` is used along with `@Autowired` to resolve ambiguity when multiple beans of the same type exist in the application context. It specifies the exact bean name to inject.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Qualifier is used with @Autowired to resolve ambiguity when multiple beans of the same type exist. By default, @Autowired injects by type, but if Spring finds multiple matching beans, it throws a NoUniqueBeanDefinitionException. @Qualifier lets you specify the exact bean name to inject. You can use it on fields, constructor parameters, or setter methods. An alternative approach is using @Primary to mark one bean as the default choice."

### Code Example
```java
// Multiple beans of same type
@Configuration
public class DataSourceConfig {
    
    @Bean(name = "mysqlDataSource")
    public DataSource mysqlDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/mydb")
            .build();
    }
    
    @Bean(name = "postgresDataSource")
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://localhost:5432/mydb")
            .build();
    }
}

// Using @Qualifier to specify which bean
@Service
public class UserService {
    
    private final DataSource dataSource;
    
    // Constructor injection with @Qualifier
    @Autowired
    public UserService(@Qualifier("mysqlDataSource") DataSource dataSource) {
        this.dataSource = dataSource;
    }
}

// Field injection with @Qualifier
@Service
public class ReportService {
    
    @Autowired
    @Qualifier("postgresDataSource")
    private DataSource dataSource;
}

// Multiple qualifiers in constructor
@Service
public class DataMigrationService {
    
    private final DataSource sourceDs;
    private final DataSource targetDs;
    
    @Autowired
    public DataMigrationService(
            @Qualifier("mysqlDataSource") DataSource sourceDs,
            @Qualifier("postgresDataSource") DataSource targetDs) {
        this.sourceDs = sourceDs;
        this.targetDs = targetDs;
    }
    
    public void migrate() {
        // Migration logic using both data sources
    }
}

// Custom qualifier annotation
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MySQL {
}

@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Postgres {
}

// Using custom qualifiers
@Configuration
public class CustomDataSourceConfig {
    
    @Bean
    @MySQL
    public DataSource mysqlDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/mydb")
            .build();
    }
    
    @Bean
    @Postgres
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://localhost:5432/mydb")
            .build();
    }
}

@Service
public class CustomQualifierService {
    
    @Autowired
    @MySQL
    private DataSource mysqlDs;
    
    @Autowired
    @Postgres
    private DataSource postgresDs;
}
```

### Key Points to Remember
- Resolves ambiguity when multiple beans of same type exist
- Works with @Autowired
- Bean name is case-sensitive
- Can create custom qualifier annotations
- Can be used on fields, parameters, and methods
- Alternative to @Primary

### Common Pitfalls
- Misspelling bean names (case-sensitive)
- Not defining the qualified bean
- Using @Qualifier without @Autowired
- Confusing bean name with bean class name
- Overusing qualifiers instead of proper design

---

## @Primary

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks a bean as the primary choice when multiple candidates exist |
| **Used On** | Class, Method |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What's the difference between @Primary and @Qualifier?" |

### What It Does
`@Primary` indicates that a bean should be given preference when multiple candidates are qualified to autowire. When multiple beans of the same type exist, the one marked with `@Primary` will be injected by default unless explicitly specified with `@Qualifier`.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Primary marks a bean as the default choice when multiple beans of the same type exist. Unlike @Qualifier which you use at the injection point, @Primary is declared at the bean definition. When Spring encounters multiple matching beans and one is marked @Primary, it will automatically choose that one. It's useful for setting a default implementation while still allowing other implementations to exist. @Qualifier always takes precedence over @Primary if both are present."

### Code Example
```java
// Multiple implementations of same interface
public interface PaymentGateway {
    void processPayment(double amount);
}

@Component
public class StripePaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing via Stripe: $" + amount);
    }
}

@Component
@Primary  // This will be the default choice
public class PayPalPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing via PayPal: $" + amount);
    }
}

@Component
public class RazorpayPaymentGateway implements PaymentGateway {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing via Razorpay: $" + amount);
    }
}

// Default injection - gets PayPal (marked as @Primary)
@Service
public class OrderService {
    
    @Autowired
    private PaymentGateway paymentGateway; // Injects PayPalPaymentGateway
    
    public void checkout(double amount) {
        paymentGateway.processPayment(amount);
    }
}

// Override with @Qualifier
@Service
public class InternationalOrderService {
    
    @Autowired
    @Qualifier("stripePaymentGateway")
    private PaymentGateway paymentGateway; // Explicitly uses Stripe
    
    public void checkout(double amount) {
        paymentGateway.processPayment(amount);
    }
}

// Using @Primary with @Bean
@Configuration
public class PaymentConfig {
    
    @Bean
    @Primary
    public PaymentGateway defaultPaymentGateway() {
        return new PayPalPaymentGateway();
    }
    
    @Bean
    public PaymentGateway backupPaymentGateway() {
        return new StripePaymentGateway();
    }
}

// Real-world example: Data sources
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    @ConfigurationProperties("spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}

@Service
public class UserService {
    
    @Autowired
    private DataSource dataSource; // Gets primaryDataSource automatically
}
```

### Key Points to Remember
- Sets a default bean when multiple exist
- Declared at bean definition, not at injection point
- @Qualifier overrides @Primary
- Only one @Primary allowed per type
- Makes code cleaner when you have a clear default
- Useful for strategy pattern implementations

### Common Pitfalls
- Having multiple @Primary beans of same type (causes error)
- Not understanding that @Qualifier overrides @Primary
- Using @Primary when there should be no default
- Forgetting that @Primary only works for autowiring by type

---

## @Lazy

| Aspect | Details |
|--------|---------|
| **Purpose** | Delays bean initialization until first use |
| **Used On** | Class, Method, Field, Parameter |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What is lazy initialization and when should you use it?" |

### What It Does
`@Lazy` delays the initialization of a bean until it's actually needed. By default, Spring creates all singleton beans at application startup (eager initialization). With `@Lazy`, the bean is only created when first requested.

### Interview-Ready Answer
> "When asked in an interview, you can say: @Lazy enables lazy initialization, meaning the bean is created only when first used rather than at application startup. By default, Spring eagerly creates all singleton beans during startup for fail-fast behavior. @Lazy is useful for beans that are expensive to create or rarely used. It can improve startup time but may cause runtime delays on first access. You can use it on @Component classes, @Bean methods, or at injection points. It's also useful for breaking circular dependencies."

### Code Example
```java
// Lazy bean - created only when first used
@Component
@Lazy
public class ExpensiveService {
    
    public ExpensiveService() {
        System.out.println("ExpensiveService created - this takes time!");
        // Simulate expensive initialization
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    public void doWork() {
        System.out.println("Doing expensive work...");
    }
}

// Eager bean (default)
@Component
public class QuickService {
    
    public QuickService() {
        System.out.println("QuickService created immediately at startup");
    }
}

// Lazy injection
@Service
public class ApplicationService {
    
    @Autowired
    @Lazy  // ExpensiveService won't be created until getExpensiveService() is called
    private ExpensiveService expensiveService;
    
    @Autowired
    private QuickService quickService; // Created at startup
    
    public void useExpensiveService() {
        // Bean is created here on first call
        expensiveService.doWork();
    }
}

// Lazy with @Bean
@Configuration
public class AppConfig {
    
    @Bean
    @Lazy
    public HeavyResourceManager heavyResourceManager() {
        System.out.println("Creating heavy resource manager...");
        return new HeavyResourceManager();
    }
    
    @Bean
    public LightweightService lightweightService() {
        return new LightweightService();
    }
}

// Breaking circular dependency with @Lazy
@Component
public class ServiceA {
    
    private final ServiceB serviceB;
    
    @Autowired
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Component
public class ServiceB {
    
    private final ServiceA serviceA;
    
    @Autowired
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}

// Global lazy initialization in application.properties
// spring.main.lazy-initialization=true

// Conditional lazy loading
@Component
public class ReportGenerator {
    
    @Autowired
    @Lazy
    private PdfGenerator pdfGenerator;
    
    @Autowired
    @Lazy
    private ExcelGenerator excelGenerator;
    
    public void generateReport(String format) {
        if ("pdf".equals(format)) {
            pdfGenerator.generate(); // Only PDF bean created
        } else if ("excel".equals(format)) {
            excelGenerator.generate(); // Only Excel bean created
        }
    }
}
```

### Key Points to Remember
- Delays bean creation until first use
- Default is eager initialization (fail-fast)
- Improves startup time for heavy beans
- Can be used to resolve circular dependencies
- Can be applied at class, method, or injection point
- Global lazy mode: `spring.main.lazy-initialization=true`
- First access may have performance hit

### Common Pitfalls
- Overusing lazy initialization (lose fail-fast benefit)
- Not expecting first-access delay
- Using lazy when eager would catch configuration errors at startup
- Not understanding impact on application startup time vs runtime
- Forgetting that lazy beans still need dependencies resolved

---

## @Scope

| Aspect | Details |
|--------|---------|
| **Purpose** | Defines the lifecycle and visibility of a bean |
| **Used On** | Class, Method |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "What are the different bean scopes in Spring?" |

### What It Does
`@Scope` defines the lifecycle and number of instances of a bean. Spring supports several scopes: singleton (default), prototype, request, session, application, and websocket (last 4 for web applications only).

### Interview-Ready Answer
> "When asked in an interview, you can say: @Scope defines the lifecycle of a Spring bean. The main scopes are: 
> 1. **Singleton** (default) - One instance per Spring container, shared across the application
> 2. **Prototype** - New instance created every time the bean is requested
> 3. **Request** - New instance per HTTP request (web apps only)
> 4. **Session** - New instance per HTTP session (web apps only)
> 5. **Application** - One instance per ServletContext (web apps only)
> 6. **Websocket** - One instance per WebSocket session
> Singleton is used for stateless services, while prototype is for stateful objects. For web scopes, beans are tied to the HTTP request/session lifecycle."

### Code Example
```java
// 1. SINGLETON SCOPE (Default)
@Component
@Scope("singleton") // or @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
public class SingletonService {
    private int counter = 0;
    
    public void increment() {
        counter++;
        System.out.println("Counter: " + counter);
    }
}

// 2. PROTOTYPE SCOPE - New instance each time
@Component
@Scope("prototype") // or @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PrototypeService {
    private String id;
    
    public PrototypeService() {
        this.id = UUID.randomUUID().toString();
        System.out.println("Created new PrototypeService: " + id);
    }
}

@Service
public class TestService {
    
    @Autowired
    private ApplicationContext context;
    
    public void demonstrateScopes() {
        // Singleton - same instance
        SingletonService s1 = context.getBean(SingletonService.class);
        SingletonService s2 = context.getBean(SingletonService.class);
        System.out.println("Same instance? " + (s1 == s2)); // true
        
        // Prototype - different instances
        PrototypeService p1 = context.getBean(PrototypeService.class);
        PrototypeService p2 = context.getBean(PrototypeService.class);
        System.out.println("Same instance? " + (p1 == p2)); // false
    }
}

// 3. REQUEST SCOPE - Per HTTP request
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, 
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ShoppingCartRequest {
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
    
    public List<Item> getItems() {
        return items;
    }
}

// 4. SESSION SCOPE - Per HTTP session
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, 
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserSession {
    private String username;
    private LocalDateTime loginTime;
    
    public UserSession() {
        this.loginTime = LocalDateTime.now();
    }
}

// Using scoped beans in singleton beans (requires proxy)
@RestController
public class CartController {
    
    @Autowired
    private ShoppingCartRequest cart; // Proxy injected
    
    @PostMapping("/cart/add")
    public ResponseEntity<?> addToCart(@RequestBody Item item) {
        cart.addItem(item);
        return ResponseEntity.ok("Item added");
    }
    
    @GetMapping("/cart")
    public ResponseEntity<List<Item>> getCart() {
        return ResponseEntity.ok(cart.getItems());
    }
}

// 5. Custom scope with @Scope
@Bean
@Scope(scopeName = "thread", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ThreadScopedBean {
    // Custom thread scope bean
}

// Prototype with ObjectFactory (avoid circular dependency)
@Service
public class SingletonServiceWithPrototype {
    
    @Autowired
    private ObjectFactory<PrototypeService> prototypeFactory;
    
    public void doSomething() {
        PrototypeService prototype = prototypeFactory.getObject();
        // Use fresh instance
    }
}

// Using @Lookup for prototype injection (method injection)
@Component
public abstract class CommandManager {
    
    public void process() {
        Command command = createCommand(); // New instance each time
        command.execute();
    }
    
    @Lookup
    protected abstract Command createCommand();
}

@Component
@Scope("prototype")
public class Command {
    public void execute() {
        System.out.println("Executing command");
    }
}
```

### Key Points to Remember
- **Singleton**: Default scope, one instance per container
- **Prototype**: New instance every time requested
- **Request**: Web scope, one instance per HTTP request
- **Session**: Web scope, one instance per HTTP session
- **Application**: Web scope, one per ServletContext
- Singleton beans are thread-unsafe if they have mutable state
- Use `proxyMode = ScopedProxyMode.TARGET_CLASS` when injecting shorter-lived scopes into longer-lived beans
- Prototype beans are not managed completely (no destroy callbacks)

### Common Pitfalls
- Injecting prototype into singleton without proxy (gets same instance)
- Using singleton scope with mutable state (thread safety issues)
- Not understanding that prototype beans aren't destroyed by container
- Forgetting to set proxyMode for web scopes
- Using session scope without proper proxy configuration
- Expecting prototype beans to have destroy callbacks

---

## @DependsOn

| Aspect | Details |
|--------|---------|
| **Purpose** | Specifies bean initialization order |
| **Used On** | Class, Method |
| **Spring Module** | Spring Core |
| **Common Interview Question** | "How do you control bean initialization order in Spring?" |

### What It Does
`@DependsOn` explicitly defines bean initialization order. It ensures that specified beans are created before the current bean. Useful when beans don't have direct dependencies but one must be initialized before another.

### Interview-Ready Answer
> "When asked in an interview, you can say: @DependsOn controls bean initialization order by specifying which beans must be created before the current bean. While @Autowired creates an implicit dependency, @DependsOn is used when there's no direct dependency but initialization order matters. For example, if Bean A must set up some shared resource before Bean B starts, you use @DependsOn. It's also useful for circular dependency scenarios or when working with static resources. You can specify multiple dependencies as an array of bean names."

### Code Example
```java
// Bean that must be initialized first
@Component("databaseInitializer")
public class DatabaseInitializer {
    
    @PostConstruct
    public void init() {
        System.out.println("1. Initializing database schema...");
        // Create tables, load initial data
    }
}

@Component("cacheWarmer")
public class CacheWarmer {
    
    @PostConstruct
    public void init() {
        System.out.println("2. Warming up cache...");
        // Pre-load cache
    }
}

// Bean that depends on others being initialized first
@Component
@DependsOn({"databaseInitializer", "cacheWarmer"})
public class ApplicationService {
    
    public ApplicationService() {
        System.out.println("3. ApplicationService created - DB and Cache are ready");
    }
}

// With @Bean methods
@Configuration
public class AppConfig {
    
    @Bean
    public DataSourceInitializer dataSourceInitializer() {
        System.out.println("Step 1: DataSource initialized");
        return new DataSourceInitializer();
    }
    
    @Bean
    public SchemaCreator schemaCreator() {
        System.out.println("Step 2: Schema created");
        return new SchemaCreator();
    }
    
    @Bean
    @DependsOn({"dataSourceInitializer", "schemaCreator"})
    public DataMigration dataMigration() {
        System.out.println("Step 3: Running data migration");
        return new DataMigration();
    }
    
    @Bean
    @DependsOn("dataMigration")
    public ApplicationReadyChecker readyChecker() {
        System.out.println("Step 4: Application is ready!");
        return new ApplicationReadyChecker();
    }
}

// Real-world example: Event system initialization
@Component("eventRegistry")
public class EventRegistry {
    
    private static final Map<String, List<EventHandler>> handlers = new HashMap<>();
    
    @PostConstruct
    public void init() {
        System.out.println("Event registry initialized");
    }
    
    public static void register(String event, EventHandler handler) {
        handlers.computeIfAbsent(event, k -> new ArrayList<>()).add(handler);
    }
}

@Component
@DependsOn("eventRegistry")
public class EventHandlerRegistrar {
    
    @PostConstruct
    public void registerHandlers() {
        System.out.println("Registering event handlers...");
        EventRegistry.register("USER_CREATED", new UserCreatedHandler());
        EventRegistry.register("ORDER_PLACED", new OrderPlacedHandler());
    }
}

@Component
@DependsOn("eventHandlerRegistrar")
public class EventPublisher {
    
    public EventPublisher() {
        System.out.println("EventPublisher ready - all handlers registered");
    }
    
    public void publishEvent(String event) {
        // Publish to registered handlers
    }
}

// Breaking circular dependency
@Component
@DependsOn("beanB")
public class BeanA {
    
    @Autowired
    @Lazy
    private BeanB beanB;
}

@Component
public class BeanB {
    
    @Autowired
    private BeanA beanA;
}

// Multiple dependencies
@Component
@DependsOn({
    "configLoader",
    "securityInitializer",
    "databaseConnection",
    "cacheManager"
})
public class MainApplicationService {
    
    public MainApplicationService() {
        System.out.println("All dependencies initialized, starting main service");
    }
}
```

### Key Points to Remember
- Controls explicit bean initialization order
- Accepts single bean name or array of names
- Used when no direct dependency exists
- Bean names are case-sensitive
- Also controls destruction order (reverse order)
- Doesn't create actual dependencies (no injection)
- Useful for setup/initialization scenarios

### Common Pitfalls
- Overusing it instead of proper dependency injection
- Circular @DependsOn (causes error)
- Misspelling bean names
- Using it when @Autowired would be more appropriate
- Not understanding it doesn't inject dependencies
- Creating complex dependency chains that are hard to maintain

---

## Summary Table

| Annotation | Purpose | Where Used | Key Use Case |
|------------|---------|------------|--------------|
| @Component | Generic Spring bean | Class | General purpose components |
| @Service | Business logic layer | Class | Service layer classes |
| @Repository | Data access layer | Class | DAO classes with exception translation |
| @Controller | Web MVC controller | Class | Traditional MVC controllers returning views |
| @Bean | Method-level bean factory | Method | Third-party beans, complex initialization |
| @Autowired | Dependency injection | Constructor/Field/Setter | Auto-inject dependencies |
| @Qualifier | Resolve injection ambiguity | Field/Parameter | Multiple beans of same type |
| @Primary | Default bean choice | Class/Method | Set default when multiple exist |
| @Lazy | Lazy initialization | Class/Method/Field | Expensive beans, circular dependencies |
| @Scope | Bean lifecycle scope | Class/Method | Control instance creation (singleton/prototype) |
| @DependsOn | Initialization order | Class/Method | Control bean creation sequence |

---

## Best Practices

1. **Use Constructor Injection** - Prefer constructor injection over field injection for better testability and immutability
2. **Choose Right Stereotype** - Use @Service, @Repository, @Controller instead of generic @Component for clarity
3. **Singleton is Default** - Be aware that beans are singleton by default; use @Scope for different lifecycles
4. **Avoid Circular Dependencies** - Design properly to avoid circular references; use @Lazy as last resort
5. **Use @Primary Wisely** - Set clear defaults with @Primary, override with @Qualifier when needed
6. **Don't Over-Lazy** - Lazy initialization hides startup errors; use sparingly
7. **Bean Naming** - Use meaningful bean names, especially when using @Qualifier
8. **Thread Safety** - Singleton beans with mutable state need synchronization

---

## Common Interview Questions

1. **What's the difference between @Component and @Bean?**
   - @Component is class-level, @Bean is method-level
   - @Component requires source code access, @Bean works with third-party
   - @Bean gives more control over instantiation

2. **Why is constructor injection preferred?**
   - Makes dependencies explicit and required
   - Enables immutability (final fields)
   - Easier unit testing without Spring
   - Prevents NullPointerException

3. **How does @Autowired work internally?**
   - BeanPostProcessor processes @Autowired
   - Scans for injection points during bean creation
   - Resolves by type from application context
   - Uses @Qualifier for disambiguation

4. **What happens if you have circular dependencies?**
   - Spring throws BeanCurrentlyInCreationException
   - Can be resolved with @Lazy on one dependency
   - Better solution: redesign to remove circular reference

5. **What's the difference between singleton and prototype scope?**
   - Singleton: One instance per container (default)
   - Prototype: New instance every time requested
   - Singleton is cached, prototype is not
   - Prototype beans don't have destroy callbacks

---
```
