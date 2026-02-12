# Configuration Annotations

Annotations for configuring the Spring container and application context.

---

## Table of Contents
- [@SpringBootApplication](#springbootapplication)
- [@EnableAutoConfiguration](#enableautoconfiguration)
- [@ComponentScan](#componentscan)
- [@Import](#import)
- [@ImportResource](#importresource)
- [@Conditional](#conditional)
- [@ConditionalOnProperty](#conditionalonproperty)
- [@ConditionalOnClass](#conditionalonclass)
- [@ConditionalOnMissingBean](#conditionalonmissingbean)
- [@PostConstruct](#postconstruct)
- [@PreDestroy](#predestroy)

---

## @SpringBootApplication

| Aspect | Details |
|--------|---------|
| **Purpose** | Main Spring Boot application entry point |
| **Used On** | Class |
| **Module** | Spring Boot |

### What It Does
Combines three annotations: `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan`. Marks the main class of a Spring Boot application.

### Interview Answer
> "@SpringBootApplication is a convenience annotation that combines @Configuration, @EnableAutoConfiguration, and @ComponentScan. It's placed on the main application class and triggers component scanning from the package containing this class. It's the starting point of every Spring Boot application."

### Code Example
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

// Equivalent to:
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class MyApplication { }

// With custom scan packages
@SpringBootApplication(scanBasePackages = "com.example")
public class CustomScanApp { }

// Exclude specific auto-configurations
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    SecurityAutoConfiguration.class
})
public class ExcludeAutoConfigApp { }

// Disable auto-configuration entirely
@SpringBootApplication(
    exclude = {},
    scanBasePackages = "com.example"
)
@EnableAutoConfiguration(exclude = HibernateJpaAutoConfiguration.class)
public class MinimalAutoConfigApp { }
```

### Key Points
- Combines 3 annotations in one
- Scans current package and sub-packages
- Enables Spring Boot auto-configuration
- Can exclude specific auto-configs
- Main class must be in root package

---

## @EnableAutoConfiguration

| Aspect | Details |
|--------|---------|
| **Purpose** | Enables Spring Boot's auto-configuration |
| **Used On** | Class |
| **Module** | Spring Boot |

### What It Does
Tells Spring Boot to automatically configure beans based on classpath dependencies, properties, and other beans. Uses `spring.factories` to discover configurations.

### Interview Answer
> "@EnableAutoConfiguration enables Spring Boot's auto-configuration mechanism. It automatically configures beans based on the jars in the classpath. For example, if H2 is on classpath, it auto-configures an in-memory database. Uses @Conditional annotations internally to decide which configurations to apply. Configurations are defined in META-INF/spring.factories."

### Code Example
```java
@Configuration
@EnableAutoConfiguration
public class AppConfig { }

// Exclude specific auto-configurations
@EnableAutoConfiguration(exclude = {
    DataSourceAutoConfiguration.class
})
public class NoDataSourceConfig { }

// Exclude by name (when class not available)
@EnableAutoConfiguration(excludeName = {
    "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration"
})
public class ExcludeByNameConfig { }

// In application.properties
// spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

// How auto-configuration works internally
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

### Key Points
- Configures beans based on classpath
- Uses @Conditional annotations
- Reads META-INF/spring.factories
- Can exclude specific configs
- Included in @SpringBootApplication
- Only configures if bean doesn't exist

---

## @ComponentScan

| Aspect | Details |
|--------|---------|
| **Purpose** | Configures component scanning directives |
| **Used On** | Class |
| **Module** | Spring Core |

### What It Does
Defines which packages Spring should scan for components (@Component, @Service, @Repository, @Controller). Can include/exclude specific patterns.

### Interview Answer
> "@ComponentScan tells Spring where to look for annotated components. By default, it scans the package of the annotated class and all sub-packages. Can specify explicit packages, use filters to include/exclude components, and enable lazy initialization. Included in @SpringBootApplication."

### Code Example
```java
// Scan specific packages
@Configuration
@ComponentScan(basePackages = "com.example.services")
public class ServiceConfig { }

// Type-safe scanning
@Configuration
@ComponentScan(basePackageClasses = {
    UserService.class,
    OrderRepository.class
})
public class TypeSafeConfig { }

// Multiple packages
@Configuration
@ComponentScan(basePackages = {
    "com.example.web",
    "com.example.services",
    "com.example.repositories"
})
public class MultiPackageConfig { }

// With filters
@Configuration
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @ComponentScan.Filter(
        type = FilterType.ANNOTATION,
        classes = MyCustomComponent.class
    ),
    excludeFilters = @ComponentScan.Filter(
        type = FilterType.REGEX,
        pattern = "com.example.legacy.*"
    )
)
public class FilteredScanConfig { }

// Filter types
@ComponentScan(
    excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes = Deprecated.class),
        @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = OldService.class),
        @Filter(type = FilterType.ASPECTJ, pattern = "com.example..*Service"),
        @Filter(type = FilterType.REGEX, pattern = ".*Test.*"),
        @Filter(type = FilterType.CUSTOM, classes = MyTypeFilter.class)
    }
)
public class AdvancedFilterConfig { }

// Lazy initialization
@ComponentScan(lazyInit = true)
public class LazyConfig { }
```

### Key Points
- Scans for @Component stereotypes
- Default: current package + sub-packages
- `basePackages` for package names
- `basePackageClasses` for type safety
- Supports include/exclude filters
- Can enable lazy initialization

---

## @Import

| Aspect | Details |
|--------|---------|
| **Purpose** | Imports configuration classes |
| **Used On** | Class |
| **Module** | Spring Core |

### What It Does
Imports one or more @Configuration classes or ImportSelector/ImportBeanDefinitionRegistrar implementations into current configuration.

### Interview Answer
> "@Import allows importing additional configuration classes. Used for modular configuration design - split large configs into focused modules and compose them. Can import @Configuration classes, ImportSelector for conditional imports, or ImportBeanDefinitionRegistrar for programmatic bean registration."

### Code Example
```java
// Basic import
@Configuration
@Import({DatabaseConfig.class, CacheConfig.class})
public class AppConfig { }

// ImportSelector - conditional import
public class MyImportSelector implements ImportSelector {
    
    @Override
    public String[] selectImports(AnnotationMetadata metadata) {
        String env = System.getProperty("env");
        if ("prod".equals(env)) {
            return new String[]{"com.example.ProdConfig"};
        }
        return new String[]{"com.example.DevConfig"};
    }
}

@Configuration
@Import(MyImportSelector.class)
public class ConditionalConfig { }

// ImportBeanDefinitionRegistrar - programmatic registration
public class MyBeanRegistrar implements ImportBeanDefinitionRegistrar {
    
    @Override
    public void registerBeanDefinitions(
            AnnotationMetadata metadata,
            BeanDefinitionRegistry registry) {
        
        GenericBeanDefinition beanDef = new GenericBeanDefinition();
        beanDef.setBeanClass(MyService.class);
        beanDef.setScope("singleton");
        registry.registerBeanDefinition("myService", beanDef);
    }
}

@Configuration
@Import(MyBeanRegistrar.class)
public class ProgrammaticConfig { }

// Real-world modular config
@Configuration
public class SecurityConfig { }

@Configuration
public class WebConfig { }

@Configuration
public class DataConfig { }

@Configuration
@Import({SecurityConfig.class, WebConfig.class, DataConfig.class})
public class MainConfig { }
```

### Key Points
- Imports other @Configuration classes
- Supports ImportSelector for conditional logic
- Supports ImportBeanDefinitionRegistrar
- Enables modular configuration
- Can import multiple classes
- Alternative to @ComponentScan for configs

---

## @ImportResource

| Aspect | Details |
|--------|---------|
| **Purpose** | Imports Spring XML configuration |
| **Used On** | Class |
| **Module** | Spring Core |

### What It Does
Imports bean definitions from XML configuration files into Java-based configuration. Used for gradual migration from XML to Java config.

### Interview Answer
> "@ImportResource imports legacy XML configuration files into Java-based Spring configuration. Useful when migrating from XML to Java config gradually, or when integrating third-party libraries that provide XML configs. Can import multiple files and supports classpath/file system paths."

### Code Example
```java
// Import single XML file
@Configuration
@ImportResource("classpath:beans.xml")
public class XmlImportConfig { }

// Multiple files
@Configuration
@ImportResource({
    "classpath:database-config.xml",
    "classpath:security-config.xml"
})
public class MultiXmlConfig { }

// beans.xml
/*
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="legacyService" class="com.example.LegacyService">
        <property name="timeout" value="30"/>
    </bean>
</beans>
*/

// Mixing XML and Java config
@Configuration
@ImportResource("classpath:legacy-beans.xml")
public class HybridConfig {
    
    @Bean
    public ModernService modernService(LegacyService legacyService) {
        return new ModernService(legacyService);
    }
}
```

### Key Points
- Imports XML bean definitions
- Used for gradual migration
- Supports multiple files
- Can mix with Java config
- Supports classpath and file paths

---

## @Conditional

| Aspect | Details |
|--------|---------|
| **Purpose** | Conditionally registers beans |
| **Used On** | Class, Method |
| **Module** | Spring Core |

### What It Does
Base annotation for conditional bean registration. Bean is only created if the specified Condition evaluates to true.

### Interview Answer
> "@Conditional enables conditional bean creation based on custom logic. The condition is evaluated at runtime. If it returns true, the bean is registered; otherwise, it's skipped. Spring Boot provides many built-in conditional annotations like @ConditionalOnClass, @ConditionalOnProperty, etc., which are all based on @Conditional."

### Code Example
```java
// Custom condition
public class WindowsCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext context, 
                          AnnotatedTypeMetadata metadata) {
        return context.getEnvironment()
            .getProperty("os.name")
            .toLowerCase()
            .contains("windows");
    }
}

// Use custom condition
@Configuration
@Conditional(WindowsCondition.class)
public class WindowsConfig {
    
    @Bean
    public FileSystem windowsFileSystem() {
        return new WindowsFileSystem();
    }
}

// Multiple conditions (AND logic)
@Configuration
@Conditional({WindowsCondition.class, DevEnvironmentCondition.class})
public class WindowsDevConfig { }

// Method-level conditional
@Configuration
public class DatabaseConfig {
    
    @Bean
    @Conditional(MySqlAvailableCondition.class)
    public DataSource mySqlDataSource() {
        return new MySqlDataSource();
    }
    
    @Bean
    @Conditional(PostgresAvailableCondition.class)
    public DataSource postgresDataSource() {
        return new PostgresDataSource();
    }
}

// Property-based condition
public class PropertyCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext context, 
                          AnnotatedTypeMetadata metadata) {
        return "true".equals(
            context.getEnvironment().getProperty("feature.enabled")
        );
    }
}
```

### Key Points
- Base for all conditional annotations
- Requires Condition implementation
- Can check properties, classes, beans
- Evaluated at runtime
- Multiple conditions use AND logic
- Works on classes and methods

---

## @ConditionalOnProperty

| Aspect | Details |
|--------|---------|
| **Purpose** | Conditional on property value |
| **Used On** | Class, Method |
| **Module** | Spring Boot |

### What It Does
Registers bean only if specified property exists and matches the expected value. Commonly used for feature toggles.

### Interview Answer
> "@ConditionalOnProperty creates beans based on configuration properties. Useful for feature flags and environment-specific beans. Can check if property exists, equals specific value, or doesn't equal value. Supports default values and matching if missing."

### Code Example
```java
// Property must exist
@Configuration
@ConditionalOnProperty(name = "feature.cache.enabled")
public class CacheConfig {
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
}

// Property must equal value
@Configuration
@ConditionalOnProperty(
    name = "database.type",
    havingValue = "mysql"
)
public class MySqlConfig { }

// Multiple properties (AND logic)
@ConditionalOnProperty({
    "feature.email.enabled",
    "feature.sms.enabled"
})
public class NotificationConfig { }

// With default - create if property missing
@Configuration
@ConditionalOnProperty(
    name = "feature.analytics.enabled",
    havingValue = "true",
    matchIfMissing = true  // default true if not specified
)
public class AnalyticsConfig { }

// Prefix for grouped properties
@Configuration
@ConditionalOnProperty(
    prefix = "app.features",
    name = "reporting",
    havingValue = "enabled"
)
public class ReportingConfig { }

// application.properties
// feature.cache.enabled=true
// database.type=mysql
// app.features.reporting=enabled

// Method-level
@Configuration
public class FeatureConfig {
    
    @Bean
    @ConditionalOnProperty(name = "feature.pdf", havingValue = "true")
    public PdfGenerator pdfGenerator() {
        return new PdfGenerator();
    }
    
    @Bean
    @ConditionalOnProperty(name = "feature.excel", havingValue = "true")
    public ExcelGenerator excelGenerator() {
        return new ExcelGenerator();
    }
}
```

### Key Points
- Checks property existence/value
- `havingValue` for expected value
- `matchIfMissing` for default behavior
- Supports prefix for grouped properties
- Common for feature flags
- Can check multiple properties

---

## @ConditionalOnClass

| Aspect | Details |
|--------|---------|
| **Purpose** | Conditional on class presence |
| **Used On** | Class, Method |
| **Module** | Spring Boot |

### What It Does
Creates bean only if specified classes are present on the classpath. Used for optional dependency support.

### Interview Answer
> "@ConditionalOnClass creates beans only when specific classes are on the classpath. Essential for auto-configuration - Spring Boot uses it to configure beans only when required dependencies exist. For example, DataSourceAutoConfiguration only runs if DataSource class is available."

### Code Example
```java
// Configure if Kafka is on classpath
@Configuration
@ConditionalOnClass(KafkaTemplate.class)
public class KafkaConfig {
    
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

// Multiple classes (AND logic)
@Configuration
@ConditionalOnClass({
    DataSource.class,
    JdbcTemplate.class
})
public class JdbcConfig { }

// By class name (when class might not be available at compile time)
@Configuration
@ConditionalOnClass(
    name = "org.apache.kafka.clients.producer.KafkaProducer"
)
public class KafkaByNameConfig { }

// Method-level
@Configuration
public class MessagingConfig {
    
    @Bean
    @ConditionalOnClass(RabbitTemplate.class)
    public RabbitTemplate rabbitTemplate() {
        return new RabbitTemplate();
    }
    
    @Bean
    @ConditionalOnClass(KafkaTemplate.class)
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>();
    }
}

// Real auto-config example
@Configuration
@ConditionalOnClass({DataSource.class, EmbeddedDatabaseType.class})
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {
    
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}
```

### Key Points
- Checks classpath for classes
- Used heavily in auto-configuration
- Can check multiple classes
- Can use class name strings
- Only loads if dependencies exist
- Enables optional features

---

## @ConditionalOnMissingBean

| Aspect | Details |
|--------|---------|
| **Purpose** | Conditional on bean absence |
| **Used On** | Method |
| **Module** | Spring Boot |

### What It Does
Creates bean only if no bean of the specified type exists. Allows user customization - auto-configuration backs off if user provides their own bean.

### Interview Answer
> "@ConditionalOnMissingBean creates a bean only if no bean of that type already exists. It's the foundation of Spring Boot's auto-configuration philosophy - provide defaults but allow easy customization. If you define your own DataSource, Spring Boot won't create its default one."

### Code Example
```java
// Auto-configure only if user hasn't provided one
@Configuration
public class DataSourceConfig {
    
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource defaultDataSource() {
        return new HikariDataSource(); // Default
    }
}

// User overrides
@Configuration
public class CustomDataSourceConfig {
    
    @Bean  // This takes precedence
    public DataSource dataSource() {
        return new MyCustomDataSource();
    }
}

// By bean name
@Bean
@ConditionalOnMissingBean(name = "myService")
public MyService defaultMyService() {
    return new MyServiceImpl();
}

// Multiple types (OR logic - missing ANY of these)
@Bean
@ConditionalOnMissingBean({DataSource.class, JdbcTemplate.class})
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder().build();
}

// Real auto-config pattern
@Configuration
public class EmailConfig {
    
    @Bean
    @ConditionalOnMissingBean
    public EmailService emailService() {
        return new SmtpEmailService(); // Default implementation
    }
}

// User can override
@Configuration
public class CustomEmailConfig {
    
    @Bean
    public EmailService emailService() {
        return new MockEmailService(); // Custom implementation
    }
}

// Search strategy
@Bean
@ConditionalOnMissingBean(
    value = CacheManager.class,
    search = SearchStrategy.CURRENT  // Only search current context
)
public CacheManager cacheManager() {
    return new SimpleCacheManager();
}
```

### Key Points
- Creates bean if none exists
- Enables easy customization
- Core of auto-configuration
- Can check by type or name
- Multiple types use OR logic
- User beans take precedence

---

## @PostConstruct

| Aspect | Details |
|--------|---------|
| **Purpose** | Initialization callback |
| **Used On** | Method |
| **Module** | Jakarta EE (javax.annotation) |

### What It Does
Marks a method to be executed after dependency injection is complete. Called once after bean construction and DI.

### Interview Answer
> "@PostConstruct marks an initialization method that runs after all dependencies are injected but before the bean is used. Useful for setup logic that requires injected dependencies. It's called exactly once in the bean lifecycle. Alternative to InitializingBean interface or init-method in @Bean."

### Code Example
```java
@Component
public class DataLoader {
    
    @Autowired
    private DatabaseService databaseService;
    
    @PostConstruct
    public void loadInitialData() {
        System.out.println("Loading initial data...");
        databaseService.loadData();
    }
}

// Multiple @PostConstruct methods (order not guaranteed)
@Service
public class ApplicationStartup {
    
    @Autowired
    private CacheService cacheService;
    
    @Autowired
    private ConfigService configService;
    
    @PostConstruct
    public void init1() {
        configService.loadConfig();
    }
    
    @PostConstruct
    public void init2() {
        cacheService.warmUp();
    }
}

// With @Bean and initMethod (alternative)
@Configuration
public class AppConfig {
    
    @Bean(initMethod = "initialize")
    public MyService myService() {
        return new MyService();
    }
}

public class MyService {
    public void initialize() {
        System.out.println("Init method called");
    }
}

// Real-world example
@Component
public class EmailTemplateLoader {
    
    @Autowired
    private ResourceLoader resourceLoader;
    
    private Map<String, String> templates;
    
    @PostConstruct
    public void loadTemplates() throws IOException {
        templates = new HashMap<>();
        Resource resource = resourceLoader.getResource("classpath:templates/");
        // Load all email templates
        templates.put("welcome", loadTemplate("welcome.html"));
        templates.put("reset", loadTemplate("reset.html"));
        System.out.println("Email templates loaded");
    }
    
    private String loadTemplate(String name) throws IOException {
        // Load template logic
        return "template content";
    }
}
```

### Key Points
- Runs after dependency injection
- Called once per bean
- Requires dependencies to be injected
- Can throw checked exceptions
- Alternative: InitializingBean interface
- Part of Jakarta EE (javax.annotation)

---

## @PreDestroy

| Aspect | Details |
|--------|---------|
| **Purpose** | Destruction callback |
| **Used On** | Method |
| **Module** | Jakarta EE (javax.annotation) |

### What It Does
Marks a method to be executed before bean destruction. Used for cleanup logic like closing connections, releasing resources, etc.

### Interview Answer
> "@PreDestroy marks a cleanup method that runs before the bean is destroyed by the container. Essential for releasing resources, closing connections, and cleanup tasks. Called during application shutdown or when a prototype bean is explicitly destroyed. Alternative to DisposableBean interface or destroyMethod in @Bean."

### Code Example
```java
@Component
public class DatabaseConnection {
    
    private Connection connection;
    
    @PostConstruct
    public void connect() {
        // Establish connection
        System.out.println("Database connected");
    }
    
    @PreDestroy
    public void disconnect() {
        if (connection != null) {
            try {
                connection.close();
                System.out.println("Database disconnected");
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

// File cleanup
@Component
public class TempFileManager {
    
    private List<File> tempFiles = new ArrayList<>();
    
    public void createTempFile(String name) {
        File file = new File("/tmp/" + name);
        tempFiles.add(file);
    }
    
    @PreDestroy
    public void cleanup() {
        System.out.println("Cleaning up temp files...");
        tempFiles.forEach(File::delete);
    }
}

// Thread pool shutdown
@Component
public class AsyncTaskExecutor {
    
    private ExecutorService executorService;
    
    @PostConstruct
    public void init() {
        executorService = Executors.newFixedThreadPool(10);
    }
    
    @PreDestroy
    public void shutdown() {
        System.out.println("Shutting down executor...");
        executorService.shutdown();
        try {
            if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
                executorService.shutdownNow();
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
        }
    }
}

// With @Bean destroyMethod (alternative)
@Configuration
public class AppConfig {
    
    @Bean(destroyMethod = "cleanup")
    public ResourceManager resourceManager() {
        return new ResourceManager();
    }
}

public class ResourceManager {
    public void cleanup() {
        System.out.println("Cleaning up resources");
    }
}

// Real-world cache cleanup
@Component
public class CacheManager {
    
    private Cache cache;
    
    @PostConstruct
    public void init() {
        cache = new ConcurrentHashMap<>();
    }
    
    @PreDestroy
    public void clearCache() {
        System.out.println("Clearing cache before shutdown");
        cache.clear();
    }
}
```

### Key Points
- Runs before bean destruction
- Used for cleanup/resource release
- Called on graceful shutdown
- NOT called for prototype beans (unless manual)
- Can throw exceptions
- Alternative: DisposableBean interface

---

## Summary Table

| Annotation | Purpose | When to Use |
|------------|---------|-------------|
| @SpringBootApplication | Main app entry | Every Boot app main class |
| @EnableAutoConfiguration | Enable auto-config | Included in @SpringBootApplication |
| @ComponentScan | Configure scanning | Custom package scanning |
| @Import | Import configurations | Modular config composition |
| @ImportResource | Import XML config | Legacy XML migration |
| @Conditional | Custom condition | Custom conditional logic |
| @ConditionalOnProperty | Property-based | Feature flags, env-specific |
| @ConditionalOnClass | Class-based | Optional dependency support |
| @ConditionalOnMissingBean | Bean absence | Allow user customization |
| @PostConstruct | Initialization | Setup after DI |
| @PreDestroy | Cleanup | Resource release |

---

## Best Practices

1. **Use @SpringBootApplication** on main class only
2. **Exclude unnecessary auto-configs** for faster startup
3. **Use @ConditionalOnProperty** for feature toggles
4. **Always cleanup resources** in @PreDestroy
5. **Keep conditions simple** - complex logic in Condition classes
6. **Use @ConditionalOnMissingBean** for defaults
7. **Cleanup in @PreDestroy** - close connections, threads, files

---

## Common Interview Questions

**Q: What does @SpringBootApplication do?**
- Combines @Configuration + @EnableAutoConfiguration + @ComponentScan
- Marks main application class
- Enables all Spring Boot features

**Q: How does auto-configuration work?**
- Reads META-INF/spring.factories
- Uses @Conditional annotations
- Configures beans based on classpath and properties
- User beans take precedence

**Q: @PostConstruct vs constructor?**
- Constructor: Dependencies not yet injected
- @PostConstruct: After DI, can use dependencies
- @PostConstruct for initialization logic

**Q: When is @PreDestroy called?**
- Application shutdown (graceful)
- Context close
- NOT for prototype scope (unless manual)
