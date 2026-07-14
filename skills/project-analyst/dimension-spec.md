# 维度文档内容规范

每个维度文件应记录通用能力、定位信息和关键约定。以下是各维度的详细记录规范。

---

## core/ — 核心框架层

### technologyStack.md

记录项目的技术栈，包括但不限于：
- 编程语言及版本
- 核心框架及版本（如 Spring Boot 3.2、Django 4.2、Express 4.x）
- 构建工具及版本（Maven、Gradle、npm、pip）
- 测试框架及版本
- 关键依赖及版本（列出核心依赖即可，不必穷举）

**示例条目**：
```
- Java 17
- Spring Boot 3.2.1
- MyBatis Plus 3.5.5
- MySQL 8.0（驱动版本 8.0.33）
- Redis 7.x（Lettuce 客户端）
- Maven 3.9.x
- JUnit 5 + Mockito
```

### module.md

记录项目的模块/包划分：
- 模块划分方式（按业务功能 / 按技术层次 / 混合）
- 各模块的职责与边界
- 模块间的依赖关系（如果有明确规则）
- 入口代码路径

**不应记录**：每个模块包含哪些具体接口或方法

### configurationManagement.md

记录项目的配置管理方式：
- 配置文件组织方式（profile 机制、yaml/properties、环境变量）
- 配置中心（如 Nacos、Apollo、Spring Cloud Config）
- 配置加载优先级
- 敏感配置管理方式（加密、Vault、环境变量注入）
- 自定义配置类的定义方式（`@ConfigurationProperties` 等）

### designPatterns.md

记录项目中普遍使用的设计模式及实现方式：
- 常见模式及其在项目中的实现（如策略模式通过 Spring Bean 选择、工厂模式、模板方法等）
- 项目约定的编码模式（如 Builder 模式构造对象、Converter/Assembler 转换模式）
- 仅记录**项目级通用模式**，不记录仅出现一次的局部模式

---

## web/ — Web/接口层

### endpointDefinition.md

记录项目如何定义和暴露接口：
- 接口入口注解/方式（`@RestController`、自定义注解、路由表等）
- 接口路径约定（RESTful 风格、版本前缀等）
- 参数校验方式（`@Valid`、自定义校验器、手动校验）
- 响应包装格式（统一 `Result<T>`、自定义 ResponseBodyAdvice 等）
- 序列化约定（日期格式、空值处理、命名策略等）
- 接口文档方式（Swagger/OpenAPI 等，如有）

**不应记录**：某个具体接口的参数定义和返回结构

### authenticationProcess.md

记录项目的鉴权流程：
- 认证方式（JWT、Session、OAuth2、SSO 等）
- 认证入口（Filter、Interceptor、Middleware、装饰器等）
- Token 校验流程（在哪校验、校验什么字段、过期处理）
- 角色/权限定义方式（注解、配置、硬编码等）
- 权限校验位置（接口层、Service 层、自定义注解等）
- 白名单/免认证接口的配置方式

### apiVersioning.md

记录接口版本管理策略（如项目有版本管理）：
- 版本方式（URL 路径 `/v1/`、Header、请求参数）
- 版本兼容性规则
- 版本路由实现

### i18n.md

记录国际化/多语言实现（如项目有国际化需求）：
- i18n 实现方式（资源文件、数据库、配置中心等）
- 资源文件组织方式
- Locale 解析逻辑（Header、Cookie、参数等）
- 消息解析器配置

---

## data/ — 数据层

### dataSourceDefinition.md

记录数据源定义（连接层面）：
- 数据库类型与版本
- 连接池配置（HikariCP、Druid 等，关键参数如 maxPoolSize）
- 多数据源配置方式（如果有）
- 读写分离配置方式（如果有）
- 数据源路由逻辑（如果有）

### dataAccessLayer.md

记录数据访问层的通用实现（访问层面）：
- ORM/数据访问框架（MyBatis、JPA、JDBC、SQLAlchemy 等）
- Mapper/Repository 的定义方式与约定
- 分页实现方式
- 通用查询封装（如通用 Mapper、BaseRepository）
- 数据库迁移工具（Flyway、Liquibase 等）

### cachingStrategy.md

记录缓存策略：
- 缓存框架（Redis、Caffeine、Guava Cache、Spring Cache 等）
- 缓存注解使用方式（`@Cacheable`、`@CacheEvict` 等或自定义注解）
- 缓存 Key 设计规范
- 缓存更新/失效策略
- 多级缓存策略（如有）
- 缓存序列化方式

### distributedTransaction.md

记录分布式事务方案（如项目涉及）：
- 分布式事务框架（Seata、XA、TCC、Saga 等）
- 使用方式和范围
- 一致性保证机制
- 补偿/回滚策略

---

## infrastructure/ — 基础设施层

### middlewareIntegration.md

记录中间件集成方式：
- 消息队列（Kafka/RabbitMQ/RocketMQ）：使用方式、Topic/Exchange 定义约定、消费者定义方式、消息序列化方式
- 搜索引擎（Elasticsearch）：使用方式、索引管理、查询封装
- 对象存储（MinIO/OSS/S3）：使用方式、Bucket 管理方式
- RPC 框架（Dubbo/gRPC）：使用方式、服务注册发现
- 其他中间件：ZooKeeper、etcd 等

**不应记录**：某个具体 Topic 的消息结构或某个索引的具体 mapping

### fileStorage.md

记录文件存储与处理方式（如项目涉及文件操作）：
- 文件上传/下载机制
- 存储后端（本地/OSS/MinIO）
- 文件类型校验方式
- 大文件处理方式（分片上传、断点续传等）
- 文件大小限制配置

### observability.md

记录监控与可观测性（如项目有监控体系）：
- 监控框架（Spring Boot Actuator、Prometheus、Micrometer 等）
- 健康检查端点
- 指标暴露方式
- 链路追踪（Sleuth、Zipkin、SkyWalking、OpenTelemetry 等）
- 告警配置方式（如有）

---

## crosscutting/ — 横切关注点

### aopAndInterceptor.md

记录AOP切面和拦截器的通用定义：
- 切面定义方式（`@Aspect`、XML 配置等）
- 拦截器链（Filter → Interceptor → AOP 的顺序，如果有自定义逻辑）
- 各切面/拦截器的用途概览（日志切面、权限切面、事务切面、限流切面等）
- 切面/拦截器的定义位置（包路径）
- 自定义注解驱动的切面（如 `@RateLimit`、`@OperationLog` 等）

**不应记录**：某个切面的具体 pointcut 表达式或通知体中的业务逻辑

### logging.md

记录日志框架与规范：
- 日志框架选型（Logback、Log4j2、SLF4J 等）
- 日志级别约定（何时用 DEBUG/INFO/WARN/ERROR）
- 日志格式（pattern 配置）
- 日志输出目标（控制台、文件、ELK 等）
- MDC/TraceId 传递方式
- 敏感信息脱敏规则（如有）
- 日志文件轮转策略

### exceptionHandling.md

记录异常处理流程：
- 全局异常处理器定义（`@ControllerAdvice`/`@ExceptionHandler` 或等价方式）
- 异常码体系（如果有统一的错误码定义）
- 异常分类方式（业务异常、系统异常、参数异常等）
- 异常与 HTTP 状态码的映射规则
- 异常日志记录方式

**不应记录**：某个具体异常码的含义列表

### encryptionAndDecryption.md

记录加解密逻辑：
- 加解密算法（AES、RSA、SM2/SM4 等）
- 加解密使用场景（接口加解密、数据库字段加解密、配置文件加解密等）
- 密钥管理方式
- 自定义加解密注解（如 `@EncryptedField` 等）

### resiliencePattern.md

记录限流/熔断/降级/重试策略（如项目有相关实现）：
- 限流实现方式（Guava RateLimiter、Sentinel、自定义注解等）
- 熔断机制（Hystrix、Sentinel、Resilience4j 等）
- 降级策略（降级方法定义方式、降级逻辑位置）
- 重试机制（Spring Retry、自定义重试等）
- 超时配置

---

## runtime/ — 运行时

### asyncAndThreadPool.md

记录异步与线程模型：
- 线程池定义与配置（核心线程数、最大线程数、队列、拒绝策略）
- `@Async` 使用方式及线程池绑定
- 异步编排方式（CompletableFuture、WebFlux 等）
- 线程上下文传递（如 TransmittableThreadLocal）

### scheduledTask.md

记录定时任务：
- 定时任务框架（Spring Scheduler、Quartz、XXL-Job、Celery Beat 等）
- 任务定义方式（`@Scheduled`、Cron 表达式、配置中心动态下发等）
- 任务调度方式（单机/分布式）
- 任务监控与告警（如有）

### eventDriven.md

记录事件驱动机制（如项目有事件模型）：
- 领域事件定义方式
- 事件发布/订阅机制（Spring ApplicationEvent、Kafka、消息总线等）
- 事件处理方式（同步/异步）
- 事件溯源（如有）

---

## devops/ — 工程化

### deploy.md

记录部署与升级：
- 部署架构（集群/单节点）
- 选主机制（如有：实现方式，如 ZooKeeper、Redis、数据库等）
- 环境差异管理（dev/test/prod 配置差异）
- 容器化方式（Docker 镜像构建、K8s 部署等）
- 升级流程（滚动升级、蓝绿部署等）
- 健康检查配置

### buildAndCI.md

记录构建与 CI/CD：
- 构建工具及配置（Maven profiles、Gradle tasks 等）
- 构建产物（JAR/WAR/Docker Image 等）
- CI/CD 流水线（Jenkins、GitLab CI、GitHub Actions 等）
- 制品发布流程
- 代码质量检查工具（SonarQube、Checkstyle 等，如有）

---

## ops/ — 运维

### opsGuide.md

记录项目运行后的日常运维知识（与 devops/ 工程化维度的区别：devops 关注"怎么把代码变成可运行的服务"，ops 关注"服务跑起来后怎么管"）：
- 日志目录与查看方式（日志文件存放路径、查看命令、日志清理策略）
- 常用运维命令（服务启停、状态检查、资源查看等常用命令清单）
- 虚拟机登录方式（SSH 地址/端口、跳板机方式、密钥/密码认证方式等——只记录登录方式，不记录具体密码）
- 虚拟机/节点名称与角色（各环境节点名称、IP、角色分配——如 app-server-01 负责业务服务，redis-node-01 负责缓存）
- 服务启停方式（启动/停止/重启命令、进程名、端口）
- 日常巡检项（需要定期检查的指标和命令，如磁盘空间、内存、连接数、队列积压等）

**不应记录**：具体密码、密钥内容等敏感凭据的明文

---

## 通用写作规范

1. **使用 Markdown 格式**，层次清晰，适当使用表格和代码块
2. **记录位置而非内容**：例如记录"鉴权在 `AuthFilter` 中实现"而非复制 `AuthFilter` 的全部代码
3. **记录约定而非实例**：例如记录"缓存 Key 格式为 `业务:实体:ID`"而非列出所有 Key
4. **使用相对路径**：代码路径使用项目相对路径，如 `src/main/java/com/xx/config/`
5. **保持精炼**：每个维度文档控制在 50-200 行，重点突出
6. **标记不确定性**：不确定的内容用 `❓` 标记，待补充的用 `⏳` 标记
7. **为 overview 提炼内容索引**：每个维度文档完成后，必须从文档实际内容中提炼出能力/知识点主题列表，写入 `overview.md` 的"维度内容索引"部分。索引描述的是"这个文件记录了什么能力/主题"，而非具体细节。例如 deploy.md 中记录了虚拟机登录方式，则索引写"虚拟机登录方式"，而非写具体的 IP 和用户名
