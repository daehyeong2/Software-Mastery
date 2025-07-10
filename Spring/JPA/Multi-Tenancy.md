## 멀티 테넌시(Multi-Tenancy)
---
멀티테넌시는 하나의 소프트웨어 인스턴스가 여러 사용자 그룹 또는 테넌트를 지원하기 위해 공유 환경 내에서 운영되는 소프트웨어 아키텍처입니다.

> 테넌트란? 자신의 자원이 아닌 서비스 제공자의 클라우드 자원을 빌려서 서비스를 이용하는 주체가 Tenant 입니다.

멀티 테넌시의 반대 개념은 싱글 테넌시입니다. 싱글 테넌시는 하나의 소프트웨어 인스턴스 또는 컴퓨터 시스템이 1명의 최종 사용자 또는 단일 사용자 그룹을 지원함을 의미합니다.

#### # 멀티 테넌시의 장점
멀티 테넌시의 비용 절감 효과
- 규모가 커질수록 컴퓨터 비용이 저렴해지므로, 멀티테넌시에서는 효율적으로 리소스를 통합하고 할당하여 궁극적으로 운영 비용을 절감할 수 있습니다.
멀티 테넌시의 효율성
- 멀티테넌시에서는 개별 사용자가 인프라를 관리하고 업데이트 및 유지 관리를 수행하는 일이 줄어듭니다. 개별 테넌트는 이 반복적이고 번거로운 작업을 직접 처리하는 대신 중앙의 클라우드 제공업체에 맡길 수 있습니다.

## Spring JPA에서의 멀티 테넌시
일반적으로 사용자마다 격리된 데이터베이스 리소스를 제공하고 싶을 때 JPA에서 멀티 테넌시 아키텍쳐를 활용합니다.
JPA의 멀티테넌시에서는 리소스를 Tenant 별로 공유(Sharing)하거나 격리(Isolating)하여 제공할 수 있습니다. 여기에서는 Database를 Tenant 마다 격리하는 방법에 대해서 알아보겠습니다.

#### # Multi-Tenancy in Hibernate
Hibernate 에서는 두가지 방식 (Separate database, Separate Schema) 를 지원합니다.
- Separate Database - Database per Tenant
	![[Pasted image 20250710230915.png]]
- Separate Schema - Schema per Tenant
	![[Pasted image 20250710231012.png]]

(파란색 사다리꼴 -> DB)

#### # Spring에서 구현하기
본 문서에서는 테넌트마다 물리적으로 분리된 Database를 사용하는 Separate Database 방식에 대해 정리하겠습니다.

먼저 Separate Database 방식은 Main/Tenant 스키마(도메인)을 분리하여 구현해야 합니다.
- Main
	- 서비스에 공통적으로 필요한 데이터를 저장합니다.
- Tenant
	- 테넌트마다 분리 보관해야 하는 데이터

테넌트는 공유 Database를 사용하거나 격리된 데이터 베이스를 사용 할 수 있습니다.
- Shared Tenant
- Isolated Tenant
	![[Pasted image 20250710231734.png]]

멀티테넌시를 구현하기 위해서는 다음과 같은 작업이 필요합니다.
- Main/Tenant Database 구현
- Tenant 식별하기
- Tenant Database 생성 및 업데이트
#### Main Database
Main Database 관련 DataSource, EntityManager 를 설정한다.  

*MainDatabaseConfig.java*
```java
@Configuration
@EnableJpaRepositories(basePackages = "~.src.multitenancy.main.repository")
@RequiredArgsConstructor
public class MasterDatabaseConfig {

  public static final String PERSISTENCE_UNIT_NAME = "main";
  private static final String DOMAIN_PACKAGE = "~.multitenancy.main.domain";
  private final DataSource dataSource;
  private final EntityManagerFactoryBuilder builder;

  @Bean
  @Primary
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    return builder.dataSource(dataSource)
        .packages(DOMAIN_PACKAGE)
        .persistenceUnit(PERSISTENCE_UNIT_NAME)
        .build();
  }


  @Bean
  @Primary
  public PlatformTransactionManager transactionManager(@Qualifier("entityManagerFactory") EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }


}
```

*Tenant.java*
```java
@Entity
@Getter
@Setter
@ToString
@Audited
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode(callSuper = false, of = "id")
public class Tenant {

  public static final String DEFAULT_TENANT_ID = "default";
  public static final String DATABASE_NAME_PREFIX = "demo_multitenancy_";

  @Id
  @GenericGenerator(name = "uuid", strategy = "uuid2")
  @GeneratedValue(generator = "uuid")
  @Column(length = 50)
  private String id;

  @Column(unique = true)
  private String name;

  private String dbName;
  private String dbAddress;
  private String dbUsername;
  private String dbPassword;
  private int maxTotal = 10;
  private int maxIdle = 10;
  private int minIdle = 0;
  private int initialSize = 0;

  @Builder
  public Tenant(String name, String dbAddress, String dbUsername, String dbPassword) {
    this.name = name;
    this.dbName = name;
    this.dbAddress = dbAddress;
    this.dbUsername = dbUsername;
    this.dbPassword = dbPassword;
  }

  @Transient
  public String getJdbcUrl() {
    return String.format("jdbc:postgresql://%s/%s?createDatabaseIfNotExist=true", dbAddress, getDatabaseName());
  }

  @Transient
  public String getDatabaseName() {
    return String.format("%s%s", DATABASE_NAME_PREFIX, dbName);
  }

}
```

#### Tenant Database
Tenant를 식별하고 각 테넌트 데이터베이스 연결을 위한 DataSource 를 제공하기 위해, CurrentTenantIdentifierResolver, MultiTenantConnectionProvide 를 구현해야 합니다.

*CurrentTenantIdentifierResolver*
```java
public class CurrentTenantIdentifierResolverImpl implements CurrentTenantIdentifierResolver {

  @Override
  public String resolveCurrentTenantIdentifier() {
    String tenant = TenantContextHolder.getTenantId();
    return StringUtils.hasText(tenant) ? tenant : DEFAULT_TENANT_ID;
  }

  @Override
  public boolean validateExistingCurrentSessions() {
    return true;
  }
}
```

TenantContext
```java
public class TenantContext {
  private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

  public static void setTenantId(String tenantId) {
    contextHolder.set(tenantId);
  }

  public static String getTenantId() {
    return contextHolder.get();
  }

  public static void clear() {
    contextHolder.remove();
  }
}
```

*DataSourceBasedMultiTenantConnectionProviderImpl*
```java
@Slf4j
@RequiredArgsConstructor
public class DataSourceBasedMultiTenantConnectionProviderImpl extends AbstractDataSourceBasedMultiTenantConnectionProviderImpl {

  private static final long serialVersionUID = 2353465673130594043L;

  private final BasicDataSource dataSource;
  private final TenantDataSources tenantDataSources;

  @Override
  protected DataSource selectAnyDataSource() {
    log.info("selectAnyDataSource: masterDataSource selected.");
    return dataSource;
  }

  @Override
  protected DataSource selectDataSource(String tenantId) {
    if (DEFAULT_TENANT_ID.equals(tenantId)) {
      log.info("MasterDataSource selected");
      return dataSource;
    }

    BasicDataSource tenantDataSource = (BasicDataSource) tenantDataSources.get(tenantId);
    log.info("Tenant DataSource selected. url: [{}], cacheSize: [{}]", tenantDataSource.getUrl(), tenantDataSources.size());
    return tenantDataSource;
  }

}
```

#### Tenant DataSource Caching

Database Connection 필요한 경우 생성하고, 일정시간 지나면 반환될 수 있도록 Guava Cache([https://github.com/google/guava](https://github.com/google/guava)) 를 사용하여 DataSource를 구성한다.

*TenantDataSources*
```java
public class TenantDataSources implements InitializingBean {

  private final TenantDataSourceCacheProperties properties;
  private final BasicDataSource dataSource;
  private final TenantRepository tenantRepository;

  private LoadingCache<String, DataSource> caches;

  @Override
  public void afterPropertiesSet() {
    createDataSourceCache();
  }

  private void createDataSourceCache() {
    log.info("DataSourceCache create. maxSize: {}, expireMinutes: {}", properties.getMaxSize(), properties.getExpireMinutes());
    caches = CacheBuilder.newBuilder()
        .maximumSize(properties.getMaxSize())
        .expireAfterAccess(properties.getExpireMinutes(), TimeUnit.MINUTES)
        .removalListener((RemovalListener<String, DataSource>) removal -> {
          BasicDataSource ds = (BasicDataSource) removal.getValue();
          try {
            ds.close();
            log.info("Closed datasource(url:[{}]).", ds.getUrl());
          } catch (SQLException e) {
            log.warn(e.toString());
          }
        })
        .build(new CacheLoader<>() {
          public DataSource load(String key) {
            Tenant tenant = tenantRepository.findById(key)
                .orElseThrow(() -> new IllegalStateException(String.format("Tenant not exists. id([%s])", key)));
            return createDataSource(tenant);
          }
        });
  }
  ....
}
```

#### TenantDatabaseConfig
앞에서 구현한 CurrentTenantIdentifierResolver, MultiTenantConnectionProvider 를 사용하여 EntityManagerFactoryBean 을 등록합니다.
```java
@Configuration
	@EnableJpaRepositories(basePackages = {"~.multitenancy.tenant.repository"},
    entityManagerFactoryRef = "tenantEntityManagerFactory",
    transactionManagerRef = "tenantTransactionManager")
@Slf4j
public class TenantDatabaseConfig {
  public static final String PERSISTENCE_UNIT_NAME = "tenant";
  public static final String DOMAIN_PACKAGE = "~.multitenancy.tenant.domain";

  @Bean
  @ConfigurationProperties(prefix = "tenant.datasource-cache")
  public TenantDataSourceCacheProperties tenantDataSourceCacheProperties() {
    return new TenantDataSourceCacheProperties();
  }

  @Bean
  @DependsOn("entityManagerFactory")
  public TenantDataSources tenantDataSources(TenantDataSourceCacheProperties tenantDataSourceCacheProperties,
      TenantRepository tenantRepository, BasicDataSource dataSource) {
    log.info("TenantDataSources create.");
    return new TenantDataSources(tenantDataSourceCacheProperties, dataSource, tenantRepository);
  }
  
  @Bean
  public MultiTenantConnectionProvider multiTenantConnectionProvider(BasicDataSource dataSource, TenantDataSources tenantDataSources) {
    log.info("multiTenantConnectionProvider create.");
    return new DataSourceBasedMultiTenantConnectionProviderImpl(dataSource, tenantDataSources);
  }

  @Bean
  public CurrentTenantIdentifierResolver currentTenantIdentifierResolver() {
    return new CurrentTenantIdentifierResolverImpl();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean tenantEntityManagerFactory(
      @Qualifier("multiTenantConnectionProvider") MultiTenantConnectionProvider connectionProvider,
      @Qualifier("currentTenantIdentifierResolver") CurrentTenantIdentifierResolver tenantIdentifierResolver,
      @Lazy JpaProperties jpaProperties) {
    log.info("tenantEntityManagerFactory create.");
    LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
    entityManagerFactoryBean.setPackagesToScan(DOMAIN_PACKAGE);
    entityManagerFactoryBean.setPersistenceUnitName(PERSISTENCE_UNIT_NAME);
    entityManagerFactoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    Map<String, Object> properties = new HashMap<>();
    properties.put(Environment.MULTI_TENANT, MultiTenancyStrategy.DATABASE);
    properties.put(Environment.MULTI_TENANT_CONNECTION_PROVIDER, connectionProvider);
    properties.put(Environment.MULTI_TENANT_IDENTIFIER_RESOLVER, tenantIdentifierResolver);
    properties.putAll(jpaProperties.getProperties());
    entityManagerFactoryBean.setJpaPropertyMap(properties);
    return entityManagerFactoryBean;
  }

  @Bean
  public PlatformTransactionManager tenantTransactionManager(@Qualifier("tenantEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }

  @Bean
  public JPAQueryFactory tenantQueryFactory(@Qualifier("tenantEntityManagerFactory") EntityManager entityManager) {
    return new JPAQueryFactory(entityManager);
  }

}
```

### Tenant 식별하기
테넌트 Database를 접속하기 위해서는 CurrentTenantIdentifierResolver 가 tenatId를 식별할 수 있도록 TenantContext 에 tenantId를 세팅해야 합니다. 테넌트 Database에 접속하는 상황에 따라 tenantId를 결정하는 방식이 달라집니다. 서비스의 경우 로그인한 사용자의 UserContext나 Token 에서 tenantId 를 얻을 수 있을 것입니다. 관리자 시스템의 경우 접근하고자 하는 테넌트 정보에 따라 각각 tenantId를 얻어야 합니다. 그리고 클라이언트가 tenantId를 가지고 있다면 요청 헤더에 tenantId 를 보내도록 하여 tenatId를 식별할 수 있습니다.

- 요청헤더로 TenantContext 구성하기
```java
@Component
@Slf4j
public class TenantFilter extends OncePerRequestFilter {
  public static final String X_TENANT_ID_HEADER = "X-Tenant-ID";

  @Override
  protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
      throws ServletException, IOException {

    String tenantId = request.getHeader(X_TENANT_ID_HEADER);

    if (StringUtils.isNotBlank(tenantId)) {
      log.debug("TenantContext set, {}", tenantId);
      TenantContext.setTenantId(tenantId);
    }
    try {
      chain.doFilter(request, response);
    } finally {
      TenantContext.clear();
      log.debug("TenantContext deleted");
    }

  }
}
```
- 접근하고자하는 테넌트 정보로 Tenant 구성하기  
    Controller 에서 tenantId 로 사용될 파라미터를 지정하고 동적으로 해당 값을 가지고 Tenant Context 를 구성한다.

*TenantSetter*
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TenantSetter {

  String key() default "";

}
```

*TenantAspect*
```java
@Aspect
@Component
@RequiredArgsConstructor
@Slf4j
@Order(Ordered.HIGHEST_PRECEDENCE + 1)
public class TenantAspect {

  private final TenantService tenantService;
  private final ExpressionParser parser = new SpelExpressionParser();

  @Around("@annotation(~.multitenancy.tenant.context.TenantSetter)")
  public Object invoke(ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();

    TenantSetter tenantSetter = method.getAnnotation(TenantSetter.class);
    String key = (String)getDynamicValue(signature.getParameterNames(), joinPoint.getArgs(), tenantSetter.key());
    log.debug("key: {}", key);
    Tenant tenant = tenantService.findById(key);

    try {
      if (tenant != null) {
        TenantContext.setTenantId(tenant.getId());
        log.debug("TenantContext created");
      }
      return joinPoint.proceed();
    } finally {
      TenantContext.clear();
      log.debug("TenantContext deleted");
    }

  }

  private Object getDynamicValue(String[] parameterNames, Object[] args, String key) {
    StandardEvaluationContext context = new StandardEvaluationContext();
    for (int i = 0; i < parameterNames.length; i++) {
      context.setVariable(parameterNames[i], args[i]);
    }

    return parser.parseExpression(key).getValue(context, Object.class);
  }


}
```

*Controller*
```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/users")
public class UserController {

  private final UserService userService;

  @PostMapping
  @TenantSetter(key = "#userCreateRequest.tenantId")
  public User save(@RequestBody UserCreateRequest userCreateRequest) {
    User user = User.builder()
        .tenantId(userCreateRequest.tenantId)
        .username(userCreateRequest.username)
        .build();

    return userService.create(user);
  }
...
```

참고 자료
https://www.redhat.com/ko/topics/cloud-computing/what-is-multitenancy
https://maily.so/saascenter/posts/wjzd512vo3p
https://velog.io/@jhkim105/Multi-Tenancy-in-Spring-Data-JPA

#### 작성 과정
Sh4re V2 백엔드의 구조를 설계하던 중 학교마다 분리된 데이터베이스 리소스에 접근해야했기 때문에 구현 방법을 찾아보다 멀티 테넌시 아키텍쳐를 알게 되었습니다.