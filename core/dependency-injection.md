# Dependency Injection Annotations

Annotations for managing dependencies and injection in Spring.

---

## Table of Contents
- [@Autowired](#autowired)
- [@Qualifier](#qualifier)
- [@Primary](#primary)
- [@Resource](#resource)
- [@Inject](#inject)
- [@Lazy](#lazy)
- [@Lookup](#lookup)
- [@DependsOn](#dependson)
- [@Order](#order)
- [@Priority](#priority)

---

## @Autowired

| Aspect | Details |
|--------|---------|
| **Purpose** | Automatic dependency injection |
| **Used On** | Constructor, Field, Setter, Method |
| **Module** | Spring Core |

### What It Does
Injects dependencies automatically by type. Spring resolves and injects matching beans from the application context.

### Interview Answer
> "@Autowired enables automatic dependency injection by type. Spring searches for a matching bean and injects it. Can be used on constructors (recommended), fields, or setters. Constructor injection is preferred as it enables immutability and makes dependencies explicit. Since Spring 4.3, @Autowired is optional for single-constructor classes."

### Code Example
```java
// 1. CONSTRUCTOR INJECTION (RECOMMENDED)
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // @Autowired optional for single constructor
    public UserService(UserRepository userRepository, 
                      EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// 2. FIELD INJECTION (Simple but not recommended for production)
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private PaymentService paymentService;
}

// 3. SETTER INJECTION (For optional dependencies)
@Service
public class ReportService {
    
    private PdfGenerator pdfGenerator;
    
    @Autowired
    public void setPdfGenerator(PdfGenerator pdfGenerator) {
        this.pdfGenerator = pdfGenerator;
    }
}

// 4. METHOD INJECTION (Multiple dependencies)
@Service
public class NotificationService {
    
    private EmailService emailService;
    private SmsService smsService;
    
    @Autowired
    public void setServices(EmailService emailService, 
                           SmsService smsService) {
        this.emailService = emailService;
        this.smsService = smsService;
    }
}

// 5. OPTIONAL DEPENDENCY
@Service
public class AnalyticsService {
    
    @Autowired(required = false)
    private MetricsCollector metricsCollector; // May not exist
    
    public void track() {
        if (metricsCollector != null) {
            metricsCollector.collect();
        }
    }
}

// 6. COLLECTION INJECTION (All beans of type)
@Service
public class NotificationDispatcher {
    
    @Autowired
    private List<NotificationChannel> channels; // All implementations
    
    public void notify(String message) {
        channels.forEach(ch -> ch.send(message));
    }
}

// 7. MAP INJECTION (Bean name as key)
@Service
public class PaymentProcessor {
    
    @Autowired
    private Map<String, PaymentGateway> gateways;
    
    public void pay(String gateway, double amount) {
        gateways.get(gateway).process(amount);
    }
}
```

### Key Points
- Injects by type (byType)
- Constructor injection preferred
- `required = false` for optional
- Can inject collections/maps
- Single constructor doesn't need @Autowired
- Throws exception if no matching bean

---

## @Qualifier

| Aspect | Details |
|--------|---------|
| **Purpose** | Disambiguates autowiring |
| **Used On** | Field, Parameter, Method |
| **Module** | Spring Core |

### What It Does
Specifies which bean to inject when multiple beans of the same type exist. Works with @Autowired to resolve ambiguity.

### Interview Answer
> "@Qualifier resolves ambiguity when multiple beans of the same type exist. Used with @Autowired to specify the exact bean name to inject. Without it, Spring throws NoUniqueBeanDefinitionException. Can also create custom qualifier annotations for better semantics."

### Code Example
```java
// Multiple implementations
@Component("emailNotification")
public class EmailNotification implements Notification { }

@Component("smsNotification")
public class SmsNotification implements Notification { }

// Use @Qualifier to specify which one
@Service
public class AlertService {
    
    private final Notification notification;
    
    @Autowired
    public AlertService(@Qualifier("emailNotification") Notification notification) {
        this.notification = notification;
    }
}

// Field injection with @Qualifier
@Service
public class UserService {
    
    @Autowired
    @Qualifier("smsNotification")
    private Notification notification;
}

// Multiple qualifiers in constructor
@Service
public class NotificationService {
    
    private final Notification primary;
    private final Notification secondary;
    
    @Autowired
    public NotificationService(
            @Qualifier("emailNotification") Notification primary,
            @Qualifier("smsNotification") Notification secondary) {
        this.primary = primary;
        this.secondary = secondary;
    }
}

// Custom Qualifier Annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Email { }

@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Sms { }

// Use custom qualifiers
@Component
@Email
public class EmailNotification implements Notification { }

@Component
@Sms
public class SmsNotification implements Notification { }

@Service
public class CustomQualifierService {
    
    @Autowired
    @Email
    private Notification emailNotif;
    
    @Autowired
    @Sms
    private Notification smsNotif;
}

// With collections
@Service
public class MultiNotificationService {
    
    @Autowired
    @Qualifier("urgent")
    private List<Notification> urgentNotifications;
}
```

### Key Points
- Resolves multiple beans ambiguity
- Specifies bean name to inject
- Works with @Autowired
- Can create custom qualifiers
- Bean names are case-sensitive
- Alternative to @Primary

---

## @Primary

| Aspect | Details |
|--------|---------|
| **Purpose** | Marks default bean for autowiring |
| **Used On** | Class, Method |
| **Module** | Spring Core |

### What It Does
Designates a bean as the primary candidate when multiple beans of the same type exist. Autowiring prefers @Primary bean unless @Qualifier is specified.

### Interview Answer
> "@Primary marks a bean as the default choice when multiple candidates exist. Unlike @Qualifier (used at injection point), @Primary is declared at bean definition. When Spring finds multiple matching beans, it chooses the @Primary one. @Qualifier always overrides @Primary."

### Code Example
```java
// Multiple DataSource implementations
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Primary  // This is the default
    public DataSource primaryDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:3306/primary");
        return ds;
    }
    
    @Bean
    public DataSource secondaryDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:3306/secondary");
        return ds;
    }
}

// Gets primary automatically
@Service
public class UserService {
    
    @Autowired
    private DataSource dataSource; // Injects primaryDataSource
}

// Override with @Qualifier
@Service
public class ReportService {
    
    @Autowired
    @Qualifier("secondaryDataSource")
    private DataSource dataSource; // Explicitly uses secondary
}

// Component-level @Primary
public interface PaymentGateway {
    void pay(double amount);
}

@Component
@Primary
public class StripeGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Stripe: " + amount);
    }
}

@Component
public class PayPalGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("PayPal: " + amount);
    }
}

@Service
public class CheckoutService {
    
    @Autowired
    private PaymentGateway gateway; // Gets StripeGateway
}

// Real-world example
@Configuration
public class CacheConfig {
    
    @Bean
    @Primary
    public CacheManager redisCacheManager() {
        return new RedisCacheManager();
    }
    
    @Bean
    public CacheManager caffeineCacheManager() {
        return new CaffeineCacheManager();
    }
}
```

### Key Points
- Marks default bean
- Declared at bean definition
- One @Primary per type allowed
- @Qualifier overrides @Primary
- Cleaner than @Qualifier everywhere
- Used for default strategy

---

## @Resource

| Aspect | Details |
|--------|---------|
| **Purpose** | Inject by name (Jakarta EE standard) |
| **Used On** | Field, Setter |
| **Module** | Jakarta EE (javax.annotation) |

### What It Does
Jakarta EE annotation for dependency injection. Injects by name first, then by type. Similar to @Autowired but name-based.

### Interview Answer
> "@Resource is a Jakarta EE annotation for dependency injection. Unlike @Autowired (byType), @Resource injects by name first. If no name specified, uses field/method name. Falls back to byType if byName fails. Part of Java standard, not Spring-specific."

### Code Example
```java
// Inject by field name
@Service
public class UserService {
    
    @Resource  // Looks for bean named "userRepository"
    private UserRepository userRepository;
    
    @Resource  // Looks for bean named "emailService"
    private EmailService emailService;
}

// Explicit name
@Service
public class OrderService {
    
    @Resource(name = "primaryDataSource")
    private DataSource dataSource;
}

// Setter injection
@Service
public class ProductService {
    
    private ProductRepository repository;
    
    @Resource(name = "productRepository")
    public void setRepository(ProductRepository repository) {
        this.repository = repository;
    }
}

// Comparison with @Autowired
@Service
public class ComparisonService {
    
    // @Autowired - byType, needs @Qualifier for disambiguation
    @Autowired
    @Qualifier("primaryCache")
    private CacheManager autowiredCache;
    
    // @Resource - byName directly
    @Resource(name = "primaryCache")
    private CacheManager resourceCache;
}

// Falls back to type if name not found
@Service
public class FallbackService {
    
    @Resource  // No bean named "someService", falls back to type
    private SomeService someService;
}
```

### Key Points
- Jakarta EE standard (JSR-250)
- Injects by name first, then type
- Uses field/method name if not specified
- Field and setter injection only
- Not Spring-specific
- Alternative to @Autowired + @Qualifier

---

## @Inject

| Aspect | Details |
|--------|---------|
| **Purpose** | Jakarta EE standard injection |
| **Used On** | Constructor, Field, Setter |
| **Module** | Jakarta EE (javax.inject) |

### What It Does
Jakarta EE standard for dependency injection (JSR-330). Functionally equivalent to @Autowired but part of Java standard.

### Interview Answer
> "@Inject is the Jakarta EE standard for dependency injection (JSR-330). Functionally similar to @Autowired but standardized. Useful for portability across DI frameworks. Unlike @Autowired, has no 'required' attribute. Use @Named instead of @Qualifier."

### Code Example
```java
// Constructor injection
@Service
public class UserService {
    
    private final UserRepository repository;
    
    @Inject
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}

// Field injection
@Service
public class OrderService {
    
    @Inject
    private OrderRepository orderRepository;
}

// With @Named (equivalent to @Qualifier)
@Service
public class PaymentService {
    
    @Inject
    @Named("stripeGateway")
    private PaymentGateway gateway;
}

// Comparison
@Service
public class ComparisonService {
    
    // Spring way
    @Autowired(required = false)
    @Qualifier("cacheA")
    private Cache springCache;
    
    // Jakarta EE way
    @Inject
    @Named("cacheB")
    private Cache javaCache;
}

// Optional with Java 8 Optional
@Service
public class OptionalService {
    
    @Inject
    private Optional<MetricsCollector> metrics;
    
    public void track() {
        metrics.ifPresent(MetricsCollector::collect);
    }
}

// Provider for lazy/dynamic injection
@Service
public class DynamicService {
    
    @Inject
    private Provider<PrototypeBean> prototypeProvider;
    
    public void doWork() {
        PrototypeBean bean = prototypeProvider.get(); // New instance
        bean.process();
    }
}
```

### Key Points
- Jakarta EE standard (JSR-330)
- Similar to @Autowired
- No `required` attribute
- Use @Named instead of @Qualifier
- Supports Provider for lazy injection
- Framework-agnostic

---

## @Lazy

| Aspect | Details |
|--------|---------|
| **Purpose** | Lazy initialization |
| **Used On** | Class, Method, Field, Parameter |
| **Module** | Spring Core |

### What It Does
Delays bean creation until first use. Can be applied to bean definitions or injection points.

### Interview Answer
> "@Lazy delays bean initialization until it's first accessed. Useful for expensive beans, circular dependencies, or conditional usage. Can be used on @Component classes, @Bean methods, or injection points. Improves startup time but may cause runtime delays on first access."

### Code Example
```java
// Lazy bean definition
@Component
@Lazy
public class ExpensiveService {
    
    public ExpensiveService() {
        System.out.println("ExpensiveService created");
        // Heavy initialization
    }
}

// Lazy injection point
@Service
public class UserService {
    
    @Autowired
    @Lazy  // ExpensiveService created only when used
    private ExpensiveService expensiveService;
    
    public void process() {
        expensiveService.doWork(); // Created here
    }
}

// Lazy @Bean
@Configuration
public class AppConfig {
    
    @Bean
    @Lazy
    public HeavyResource heavyResource() {
        return new HeavyResource();
    }
}

// Breaking circular dependency
@Component
public class ServiceA {
    
    @Autowired
    @Lazy  // Breaks circular reference
    private ServiceB serviceB;
}

@Component
public class ServiceB {
    
    @Autowired
    private ServiceA serviceA;
}

// Constructor injection with @Lazy
@Service
public class OrderService {
    
    private final ReportGenerator reportGenerator;
    
    @Autowired
    public OrderService(@Lazy ReportGenerator reportGenerator) {
        this.reportGenerator = reportGenerator;
    }
}

// Conditional usage
@Service
public class MultiFormatService {
    
    @Autowired
    @Lazy
    private PdfGenerator pdfGenerator;
    
    @Autowired
    @Lazy
    private ExcelGenerator excelGenerator;
    
    public void generate(String format) {
        if ("pdf".equals(format)) {
            pdfGenerator.generate();  // Only PDF bean created
        } else {
            excelGenerator.generate(); // Only Excel bean created
        }
    }
}
```

### Key Points
- Delays initialization
- Can be class or injection-level
- Improves startup time
- Useful for circular dependencies
- First access has delay
- Use sparingly (loses fail-fast)

---

## @Lookup

| Aspect | Details |
|--------|---------|
| **Purpose** | Method injection for prototype beans |
| **Used On** | Method |
| **Module** | Spring Core |

### What It Does
Injects a new instance each time the method is called. Solves the problem of injecting prototype beans into singleton beans.

### Interview Answer
> "@Lookup enables method injection, mainly for getting new instances of prototype-scoped beans from singleton beans. Spring overrides the annotated method to return a new bean instance each time. The method must be abstract or have no implementation. Alternative to ObjectFactory or ApplicationContext.getBean()."

### Code Example
```java
// Prototype bean
@Component
@Scope("prototype")
public class ShoppingCart {
    
    private String id = UUID.randomUUID().toString();
    private List<Item> items = new ArrayList<>();
    
    public void addItem(Item item) {
        items.add(item);
    }
}

// Singleton bean using @Lookup
@Component
public abstract class CartService {
    
    public void processOrder() {
        ShoppingCart cart = createCart(); // New instance each time
        cart.addItem(new Item());
        System.out.println("Cart ID: " + cart.getId());
    }
    
    @Lookup  // Spring implements this method
    protected abstract ShoppingCart createCart();
}

// Another example
@Component
@Scope("prototype")
public class Command {
    
    private String id = UUID.randomUUID().toString();
    
    public void execute() {
        System.out.println("Executing command: " + id);
    }
}

@Component
public abstract class CommandExecutor {
    
    public void run() {
        for (int i = 0; i < 5; i++) {
            Command cmd = getCommand(); // New instance each time
            cmd.execute();
        }
    }
    
    @Lookup
    protected abstract Command getCommand();
}

// Can also work with concrete methods (Spring overrides)
@Component
public class OrderProcessor {
    
    @Lookup
    public OrderContext getOrderContext() {
        return null; // Spring overrides this
    }
    
    public void process() {
        OrderContext ctx = getOrderContext();
        // Use new context
    }
}

// Alternative: ObjectFactory
@Component
public class AlternativeCartService {
    
    @Autowired
    private ObjectFactory<ShoppingCart> cartFactory;
    
    public void processOrder() {
        ShoppingCart cart = cartFactory.getObject(); // New instance
    }
}
```

### Key Points
- Method injection for prototypes
- Method must be abstract or empty
- Spring implements/overrides method
- Solves prototype in singleton issue
- Alternative: ObjectFactory, Provider
- Less common than other DI methods

---

## @DependsOn

| Aspect | Details |
|--------|---------|
| **Purpose** | Controls bean initialization order |
| **Used On** | Class, Method |
| **Module** | Spring Core |

### What It Does
Explicitly defines bean creation order. Ensures specified beans are created before the current bean.

### Interview Answer
> "@DependsOn controls bean initialization order when there's no direct dependency but order matters. For example, Bean A must initialize before Bean B for setup reasons, but B doesn't inject A. Can specify multiple dependencies. Also controls destruction order (reverse)."

### Code Example
```java
// Bean that must initialize first
@Component("databaseInitializer")
public class DatabaseInitializer {
    
    @PostConstruct
    public void init() {
        System.out.println("1. Database initialized");
    }
}

// Depends on database being ready
@Component
@DependsOn("databaseInitializer")
public class DataLoader {
    
    @PostConstruct
    public void loadData() {
        System.out.println("2. Loading data");
    }
}

// Multiple dependencies
@Component
@DependsOn({"databaseInitializer", "cacheManager", "configLoader"})
public class ApplicationService {
    
    public ApplicationService() {
        System.out.println("3. ApplicationService ready");
    }
}

// With @Bean
@Configuration
public class AppConfig {
    
    @Bean
    public SchemaCreator schemaCreator() {
        return new SchemaCreator();
    }
    
    @Bean
    @DependsOn("schemaCreator")
    public DataMigration dataMigration() {
        return new DataMigration();
    }
}

// Real-world: Event system
@Component("eventRegistry")
public class EventRegistry {
    
    private static Map<String, List<Handler>> handlers = new HashMap<>();
    
    @PostConstruct
    public void init() {
        System.out.println("Event registry ready");
    }
}

@Component
@DependsOn("eventRegistry")
public class EventHandlerRegistrar {
    
    @PostConstruct
    public void register() {
        System.out.println("Registering handlers");
        // Register handlers to registry
    }
}

// Circular @DependsOn causes error
@Component
@DependsOn("beanB")
public class BeanA { }

@Component
@DependsOn("beanA")  // ERROR: Circular dependency
public class BeanB { }
```

### Key Points
- Controls initialization order
- Used when no direct dependency
- Accepts bean names array
- Controls destruction order too
- Circular @DependsOn throws error
- Bean names are case-sensitive

---

## @Order

| Aspect | Details |
|--------|---------|
| **Purpose** | Defines bean ordering in collections |
| **Used On** | Class, Method |
| **Module** | Spring Core |

### What It Does
Defines the order of beans when multiple beans are injected as a collection. Lower values have higher priority.

### Interview Answer
> "@Order determines the order of beans in autowired collections. Lower values come first. Useful when you have multiple implementations and order matters (like filter chains, interceptors). Doesn't affect singleton creation order - only collection ordering."

### Code Example
```java
// Multiple filters with order
public interface RequestFilter {
    void filter(Request request);
}

@Component
@Order(1)  // Executes first
public class AuthenticationFilter implements RequestFilter {
    public void filter(Request request) {
        System.out.println("1. Authentication");
    }
}

@Component
@Order(2)  // Executes second
public class LoggingFilter implements RequestFilter {
    public void filter(Request request) {
        System.out.println("2. Logging");
    }
}

@Component
@Order(3)  // Executes third
public class ValidationFilter implements RequestFilter {
    public void filter(Request request) {
        System.out.println("3. Validation");
    }
}

// Inject as ordered list
@Component
public class FilterChain {
    
    @Autowired
    private List<RequestFilter> filters; // Ordered by @Order
    
    public void process(Request request) {
        filters.forEach(f -> f.filter(request));
    }
}

// With negative values (higher priority)
@Component
@Order(-1)  // Highest priority
public class SecurityFilter implements RequestFilter {
    public void filter(Request request) {
        System.out.println("0. Security check");
    }
}

// Default order (no annotation) = Ordered.LOWEST_PRECEDENCE
@Component
public class DefaultFilter implements RequestFilter {
    public void filter(Request request) {
        System.out.println("4. Default");
    }
}

// Using Ordered interface (alternative)
@Component
public class CustomFilter implements RequestFilter, Ordered {
    
    @Override
    public int getOrder() {
        return 5;
    }
    
    public void filter(Request request) {
        System.out.println("5. Custom");
    }
}

// Real-world: Event listeners
@Component
@Order(1)
public class EmailEventListener implements ApplicationListener<UserCreatedEvent> {
    public void onApplicationEvent(UserCreatedEvent event) {
        System.out.println("Send welcome email");
    }
}

@Component
@Order(2)
public class AnalyticsEventListener implements ApplicationListener<UserCreatedEvent> {
    public void onApplicationEvent(UserCreatedEvent event) {
        System.out.println("Track user creation");
    }
}
```

### Key Points
- Orders beans in collections
- Lower value = higher priority
- Doesn't affect creation order
- Works with List injection
- Default = LOWEST_PRECEDENCE
- Alternative: Ordered interface

---

## @Priority

| Aspect | Details |
|--------|---------|
| **Purpose** | Jakarta EE standard for ordering |
| **Used On** | Class |
| **Module** | Jakarta EE (javax.annotation) |

### What It Does
Jakarta EE standard for ordering beans. Similar to @Order but part of Java standard. Lower values have higher priority.

### Interview Answer
> "@Priority is Jakarta EE's standard for ordering (JSR-250). Functionally similar to Spring's @Order but standardized. Lower values = higher priority. Useful for portability and when working with Jakarta EE standards. Spring respects both @Order and @Priority."

### Code Example
```java
// Using @Priority
public interface MessageHandler {
    void handle(String message);
}

@Component
@Priority(1)  // Highest priority
public class CriticalMessageHandler implements MessageHandler {
    public void handle(String message) {
        System.out.println("Critical: " + message);
    }
}

@Component
@Priority(2)
public class NormalMessageHandler implements MessageHandler {
    public void handle(String message) {
        System.out.println("Normal: " + message);
    }
}

@Component
@Priority(3)  // Lowest priority
public class LowPriorityHandler implements MessageHandler {
    public void handle(String message) {
        System.out.println("Low: " + message);
    }
}

// Inject as ordered list
@Component
public class MessageProcessor {
    
    @Autowired
    private List<MessageHandler> handlers; // Ordered by @Priority
    
    public void process(String message) {
        handlers.forEach(h -> h.handle(message));
    }
}

// Comparison with @Order
@Component
@Order(1)  // Spring-specific
public class SpringOrderedBean { }

@Component
@Priority(1)  // Jakarta EE standard
public class JakartaPriorityBean { }

// Both work in Spring
@Service
public class BeanCollector {
    
    @Autowired
    private List<SomeInterface> beans; // Ordered by @Order or @Priority
}

// Constants for common priorities
@Component
@Priority(Priorities.USER_DATA_CONSTRAINT)  // 1000
public class UserDataFilter { }

@Component
@Priority(Priorities.AUTHORIZATION)  // 2000
public class AuthorizationFilter { }
```

### Key Points
- Jakarta EE standard (JSR-250)
- Similar to @Order
- Lower value = higher priority
- Works in Spring
- More portable than @Order
- Spring respects both

---

## Summary Table

| Annotation | Purpose | Inject By | Module |
|------------|---------|-----------|--------|
| @Autowired | DI by type | Type | Spring |
| @Qualifier | Disambiguate | Name | Spring |
| @Primary | Default bean | - | Spring |
| @Resource | DI by name | Name→Type | Jakarta EE |
| @Inject | DI standard | Type | Jakarta EE |
| @Lazy | Lazy init | - | Spring |
| @Lookup | Method injection | - | Spring |
| @DependsOn | Init order | - | Spring |
| @Order | Collection order | - | Spring |
| @Priority | Collection order | - | Jakarta EE |

---

## Best Practices

1. **Prefer constructor injection** - immutability, testability
2. **Use @Primary for defaults** - cleaner than @Qualifier everywhere
3. **@Qualifier for clarity** - when @Primary isn't appropriate
4. **Avoid @Lazy overuse** - lose fail-fast benefit
5. **Use @DependsOn sparingly** - prefer proper dependencies
6. **@Order for filters/chains** - when order matters
7. **Constructor over field** - explicit dependencies

---

## Common Interview Questions

**Q: Constructor vs Field vs Setter injection?**
- Constructor: Immutable, required deps, testable (recommended)
- Field: Simple, but hard to test, not immutable
- Setter: Optional deps, can change after construction

**Q: How to inject multiple beans of same type?**
- List/Set injection: `List<Interface> beans`
- Map injection: `Map<String, Interface> beans`
- Use @Qualifier for specific one

**Q: @Autowired vs @Inject vs @Resource?**
- @Autowired: Spring, byType, most features
- @Inject: Jakarta EE standard, byType
- @Resource: Jakarta EE standard, byName first

**Q: How to solve circular dependencies?**
- Use @Lazy on one dependency
- Redesign to remove circularity (preferred)
- Setter injection instead of constructor
