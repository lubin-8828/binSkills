---
name: AppAnalyst
description: 项目架构分析技能。解析和阅读项目代码，从多个通用维度（技术栈、接口定义、鉴权流程、AOP切面、日志、缓存、配置管理、异步线程、数据访问、中间件、异常处理、加解密、部署、运维等）分析项目，生成并维护 .analysed 目录下的结构化文档。支持三种模式：/AppAnalyst init（初始化分析）、/AppAnalyst update（增量更新）、/AppAnalyst explore [方向]（定向探索）。当用户说"分析项目"、"项目架构分析"、"init项目文档"、"更新项目分析"、"探索项目"时触发。也适用于用户想了解一个陌生项目的架构、快速梳理项目通用能力、或者维护项目技术文档的场景。即使用户只是说"帮我看一下这个项目"，也应考虑使用此技能。
---

# AppAnalyst — 项目架构分析技能

## 调用方式

使用 `/AppAnalyst` 触发技能，通过参数指定运行模式：

| 命令 | 说明 |
|------|------|
| `/AppAnalyst init` | 初始化分析：完整扫描项目，创建 `.analysed/` 目录及所有文档 |
| `/AppAnalyst update` | 增量更新：基于 git diff 分析代码变更，更新受影响的维度文档 |
| `/AppAnalyst explore` | 定向探索：指定方向或维度，主动阅读代码，交互确认后更新文档 |
| `/AppAnalyst explore <方向>` | 定向探索并指定方向，如 `/AppAnalyst explore 缓存策略`、`/AppAnalyst explore aopAndInterceptor` |
| `/AppAnalyst` | 不指定模式时，根据项目状态自动判断：无 `.analysed/` 目录则走 init；已有则询问用户选择模式 |

**示例**：
```
/AppAnalyst init
/AppAnalyst update
/AppAnalyst explore 鉴权流程
/AppAnalyst explore cachingStrategy
/AppAnalyst
```

## 核心理念

本技能从**通用维度**分析项目，记录的是"这个项目有什么能力、怎么找到入口"，而不是"某个具体业务接口的处理流程"。后者看代码就行，本技能解决的是**快速定位和认知**问题。

文档中记录的内容应该是：
- ✅ 通用能力与实现方式（如：鉴权在 Filter 层通过 JWT 校验 token）
- ✅ 入口与定位信息（如：AOP 切面定义在 `com.xx.aspect` 包下）
- ✅ 约定与规范（如：接口统一返回 `Result<T>` 包装类）
- ❌ 具体业务流程（如：用户注册接口的完整调用链）
- ❌ 具体参数处理逻辑
- ❌ 某个接口调了哪个 service 的哪个方法

## 目录结构

在项目根目录下创建并维护 `.analysed/` 目录：

```
.analysed/
 |- overview.md                        # 总览文件
 |- doc/
  |- core/                             # 核心框架层
  |  |- technologyStack.md
  |  |- module.md
  |  |- configurationManagement.md
  |  |- designPatterns.md
  |
  |- web/                              # Web/接口层
  |  |- endpointDefinition.md
  |  |- authenticationProcess.md
  |  |- apiVersioning.md
  |  |- i18n.md
  |
  |- data/                             # 数据层
  |  |- dataSourceDefinition.md
  |  |- dataAccessLayer.md
  |  |- cachingStrategy.md
  |  |- distributedTransaction.md
  |
  |- infrastructure/                   # 基础设施层
  |  |- middlewareIntegration.md
  |  |- fileStorage.md
  |  |- observability.md
  |
  |- crosscutting/                     # 横切关注点
  |  |- aopAndInterceptor.md
  |  |- logging.md
  |  |- exceptionHandling.md
  |  |- encryptionAndDecryption.md
  |  |- resiliencePattern.md
  |
  |- runtime/                          # 运行时
  |  |- asyncAndThreadPool.md
  |  |- eventDriven.md
  |  |- scheduledTask.md
  |
  |- devops/                           # 工程化
  |  |- deploy.md
  |  |- buildAndCI.md
  |
  |- ops/                             # 运维
  |  |- opsGuide.md
```

**重要**：不涉及的维度不创建对应文件，绝不创建空文件或占位文件。

## 模式判断逻辑

技能被调用后，按以下逻辑判断运行模式：

1. **解析用户输入的参数**（即调用 `/AppAnalyst` 时附带的 args）：
   - 参数以 `init` 开头 → 进入 **init 模式**
   - 参数以 `update` 开头 → 进入 **update 模式**
   - 参数以 `explore` 开头 → 进入 **explore 模式**，`explore` 后面的内容作为探索方向
   - 无参数 → 自动判断：项目根目录下不存在 `.analysed/` 则走 init；已存在则询问用户选择模式

2. **如果不是通过 `/AppAnalyst` 命令触发的**（而是通过自然语言触发），根据用户意图判断：
   - "初始化分析"、"第一次分析"、"从零开始" → init 模式
   - "更新分析"、"代码变了更新一下" → update 模式
   - "探索一下xxx"、"帮我看看xxx" → explore 模式
   - 无法判断时，询问用户选择模式

## 三种运行模式

### 模式一：init — 初始化分析

完整分析项目并创建 `.analysed/` 目录及所有文档。

#### 执行流程

**阶段1：自动扫描（无需用户参与）**

先独立探索项目，收集以下信息：
1. 扫描项目目录结构（顶层目录树、包结构）
2. 识别构建文件（`pom.xml` / `build.gradle` / `package.json` / `requirements.txt` / `go.mod` / `Cargo.toml` 等）
3. 识别框架特征（Spring Boot 启动类、Django settings、Flask app、Express 入口等）
4. 识别配置文件（`application.yml` / `.env` / `config.py` / `nginx.conf` 等）
5. 识别代码分层（controller/service/dao/repository/mapper 等包结构）
6. 识别中间件依赖（Redis、MQ、ES 等从依赖和配置中推断）
7. 识别部署相关文件（Dockerfile、docker-compose、k8s manifests、CI 配置等）

**阶段2：交互式确认（按优先级提问）**

基于阶段1的发现，逐维度向用户确认。提问时遵循以下原则：

1. **先探索再提问**：每个问题提出前，必须先在代码中搜索答案，给用户提供选项而非空问
   - ❌ "你的项目使用了什么缓存框架？"
   - ✅ "我在代码中发现了 Redis 和 Caffeine 的依赖，请确认：A) 主要用 Redis B) 主要用 Caffeine C) 两者配合使用 D) 其他"

2. **按优先级提问**：技术栈 → 核心架构 → 横切关注点 → 基础设施 → 工程化

3. **尊重用户节奏**：
   - 如果用户表示不想回答某个维度，标记为 `⏳ 待补充`
   - 如果用户不知道且你自己也查不到实现，标记为 `❓ 未知`
   - 不纠缠用户，尽快推进

4. **确认后深挖**：用户确认了某个维度后，深入代码找到具体实现位置和方式

**阶段3：深度探索与文档生成**

1. 对每个已确认的维度，深入代码验证并记录通用能力
2. 对"未知"维度，尝试在代码中搜索证据
3. 生成所有文档文件
4. 生成 `overview.md`（包含 git 提交编号、模块表、维度状态表、**维度内容索引**、代码路径映射）。维度内容索引必须从各文档的实际内容中提炼，列出每个文件记录的能力/知识点主题，而非照搬模板默认值

#### 提问维度参考

按以下顺序引导提问，但不必全部覆盖，根据项目实际情况选择：

- **技术栈**：语言版本、核心框架版本、构建工具
- **核心架构**：项目分层、模块划分、包组织方式
- **部署方式**：集群/单节点、选主机制、环境差异、升级流程
- **数据库/中间件**：类型、版本、作用、连接方式
- **接口定义**：入口注解、路由方式、参数校验、响应包装
- **鉴权流程**：认证方式、校验位置、角色/权限定义
- **AOP/拦截器**：切面定义、拦截器链、使用场景
- **日志**：框架选型、日志级别约定、MDC/TraceId
- **缓存**：框架、Key 设计规范、更新/失效策略
- **配置管理**：配置文件组织、配置中心、敏感配置处理
- **异步/线程**：线程池配置、@Async、定时任务
- **异常处理**：全局异常处理器、异常码体系
- **加解密**：算法、使用场景
- **运维**：日志目录、常用命令、虚拟机登录方式、节点名称与角色、服务启停、日常巡检
- **其他**：根据阶段1发现追加

### 模式二：update — 增量更新

基于 git 提交差异，分析代码变更对 `.analysed/` 文档的影响并更新。

#### 执行流程

1. 从 `overview.md` 读取上次分析的 git 提交编号
2. 获取当前最新的 git 提交编号
3. 计算两次提交之间的差异：`git diff <old_commit>..<new_commit> --stat`
4. 根据差异文件路径，对照 `overview.md` 中的"维度与代码路径映射"，确定受影响的维度文档
5. 对受影响的维度，读取变更内容，判断是否涉及通用能力的变化：
   - **需要更新**：新增/修改了拦截器、配置类、AOP 定义、异常处理器、数据源配置、依赖版本变更等
   - **不需要更新**：纯业务逻辑变更、新增业务接口、修改业务参数处理等
6. 更新受影响的文档
7. 更新 `overview.md` 中受影响维度对应的**维度内容索引**条目——从更新后的文档内容中重新提炼能力/知识点主题
8. 更新 `overview.md` 中的 git 提交编号

**关键判断原则**：`.analysed/` 记录的是通用能力，只有通用能力发生变化时才更新。业务代码变更通常不触发更新。

#### 受影响路径快速判断

| 变更路径模式 | 可能影响的维度文档 |
|---|---|
| `pom.xml`, `build.gradle`, `requirements.txt` | technologyStack.md |
| `application*.yml`, `application*.properties`, `.env` | configurationManagement.md, dataSourceDefinition.md |
| `*Filter.java`, `*Interceptor.java`, `*Security*.java` | authenticationProcess.md, aopAndInterceptor.md |
| `*Aspect.java`, `*Aop*.java` | aopAndInterceptor.md |
| `*ExceptionHandler*.java`, `*ControllerAdvice*.java` | exceptionHandling.md |
| `*Config.java`, `*Configuration.java` | 对应配置类所属的维度 |
| `Dockerfile`, `docker-compose*.yml`, `*.yaml` (k8s) | deploy.md, opsGuide.md |
| `*Scheduler*.java`, `*Job*.java`, `*Task*.java` | scheduledTask.md, asyncAndThreadPool.md |
| `*Cache*.java`, `cache*.yml` | cachingStrategy.md |
| `*ThreadPool*.java`, `*Async*.java` | asyncAndThreadPool.md |

### 模式三：explore — 定向探索

用户给出一个方向或代码片段，技能主动阅读代码和探索，与用户交互讨论后更新到文档中。

#### 执行流程

1. 如果用户没有指定方向，提供维度模板供选择：

```
你想探索哪个方向？可选：
1. 🔍 切面与拦截器（aopAndInterceptor.md）
2. 🔍 缓存策略（cachingStrategy.md）
3. 🔍 消息队列使用方式（middlewareIntegration.md）
4. 🔍 鉴权流程（authenticationProcess.md）
5. 🔍 数据访问层（dataAccessLayer.md）
6. 🔍 异步与线程模型（asyncAndThreadPool.md）
7. 🔍 运维指南（opsGuide.md）
8. 🔍 自定义方向：_____
```

2. 根据用户选择的方向，从代码中搜索相关实现
3. 阅读核心代码，整理发现
4. 与用户交互讨论，确认理解是否正确
5. 更新对应的维度文档
6. 如果该维度文档尚不存在，创建之，并更新 `overview.md` 中的维度状态表和**维度内容索引**（从新建文档内容中提炼能力/知识点主题）。如果文档已存在但内容有更新，同步更新 overview 中对应的内容索引条目

## 各维度文档内容规范

每个维度文档应记录**通用能力**和**定位信息**，参考以下规范。如项目不涉及某维度则不创建文件。

详细的内容规范参见 [references/dimension-spec.md](references/dimension-spec.md)。

## overview.md 模板

**核心要求**：`overview.md` 是 Agent 进入 `.analysed/` 目录后最先看到的文件，必须具备**详细索引能力**——Agent 读到 overview 后应能清楚知道每个维度文档里有哪些知识点和通用能力，从而决定是否需要进一步打开某个子文档。索引描述的是"这个文件记录了什么能力/主题"，而非具体细节内容。

```markdown
# 项目分析总览

## 基本信息
- 项目名称：
- 分析时间：YYYY-MM-DD
- Git 提交编号：`<commit_hash>`
- 技术栈简述：

## 项目结构
（简洁的目录树 + 每个顶层包/目录的一句话说明）

## 功能模块
| 模块 | 说明 | 入口代码路径 |
|------|------|-------------|
|      |      |             |

## 分析维度

> 内容索引列列出该维度文档记录的**能力/知识点主题**，Agent 读到 overview 后即可知道"哪个文件里有我需要的信息"，再打开对应子文档获取详细描述与定位信息。索引只列出主题，不展开具体内容。

| 类别 | 维度 | 文档 | 内容索引 | 状态 |
|------|------|------|----------|------|
| 核心 | 技术栈 | [technologyStack.md](doc/core/technologyStack.md) | 语言版本、核心框架版本、构建工具、关键依赖版本 | ✅ 已完成 |
| 核心 | 模块定义 | [module.md](doc/core/module.md) | 模块划分方式、各模块职责与边界、模块间依赖关系、入口代码路径 | ✅ 已完成 |
| 核心 | 配置管理 | [configurationManagement.md](doc/core/configurationManagement.md) | 配置文件组织方式、配置中心、加载优先级、敏感配置管理、自定义配置类定义方式 | ⏳ 待补充 |
| 核心 | 设计模式约定 | [designPatterns.md](doc/core/designPatterns.md) | 项目级通用设计模式实现、约定编码模式（Builder/Converter/Assembler 等） | ❓ 未知 |
| Web  | 接口定义 | [endpointDefinition.md](doc/web/endpointDefinition.md) | 接口入口注解/方式、路径约定、参数校验方式、响应包装格式、序列化约定、接口文档方式 | ✅ 已完成 |
| Web  | 鉴权流程 | [authenticationProcess.md](doc/web/authenticationProcess.md) | 认证方式、认证入口、Token 校验流程、角色/权限定义方式、权限校验位置、白名单配置方式 | ⏳ 待补充 |
| Web  | 接口版本管理 | [apiVersioning.md](doc/web/apiVersioning.md) | 版本方式（URL/Header/参数）、版本兼容性规则、版本路由实现 | ❓ 未知 |
| Web  | 国际化 | [i18n.md](doc/web/i18n.md) | i18n 实现方式、资源文件组织、Locale 解析逻辑、消息解析器配置 | ❓ 未知 |
| 数据 | 数据源定义 | [dataSourceDefinition.md](doc/data/dataSourceDefinition.md) | 数据库类型与版本、连接池配置、多数据源配置、读写分离配置、数据源路由逻辑 | ✅ 已完成 |
| 数据 | 数据访问层 | [dataAccessLayer.md](doc/data/dataAccessLayer.md) | ORM/数据访问框架、Mapper/Repository 定义方式、分页实现、通用查询封装、数据库迁移工具 | ✅ 已完成 |
| 数据 | 缓存策略 | [cachingStrategy.md](doc/data/cachingStrategy.md) | 缓存框架、缓存注解使用方式、Key 设计规范、更新/失效策略、多级缓存策略、序列化方式 | ✅ 已完成 |
| 数据 | 分布式事务 | [distributedTransaction.md](doc/data/distributedTransaction.md) | 分布式事务框架、使用方式和范围、一致性保证机制、补偿/回滚策略 | ❓ 未知 |
| 基础设施 | 中间件集成 | [middlewareIntegration.md](doc/infrastructure/middlewareIntegration.md) | 消息队列使用方式与Topic约定、搜索引擎使用方式、对象存储使用方式、RPC框架使用方式 | ✅ 已完成 |
| 基础设施 | 文件存储 | [fileStorage.md](doc/infrastructure/fileStorage.md) | 文件上传/下载机制、存储后端、文件类型校验、大文件处理方式、大小限制配置 | ⏳ 待补充 |
| 基础设施 | 可观测性 | [observability.md](doc/infrastructure/observability.md) | 监控框架、健康检查端点、指标暴露方式、链路追踪、告警配置方式 | ❓ 未知 |
| 横切 | AOP与拦截器 | [aopAndInterceptor.md](doc/crosscutting/aopAndInterceptor.md) | 切面定义方式、拦截器链顺序、各切面用途概览、定义位置、自定义注解驱动的切面 | ✅ 已完成 |
| 横切 | 日志 | [logging.md](doc/crosscutting/logging.md) | 日志框架选型、级别约定、格式配置、输出目标、MDC/TraceId 传递、敏感信息脱敏、轮转策略 | ✅ 已完成 |
| 横切 | 异常处理 | [exceptionHandling.md](doc/crosscutting/exceptionHandling.md) | 全局异常处理器定义、异常码体系、异常分类方式、与HTTP状态码映射规则、日志记录方式 | ✅ 已完成 |
| 横切 | 加解密 | [encryptionAndDecryption.md](doc/crosscutting/encryptionAndDecryption.md) | 加解密算法、使用场景（接口/数据库/配置文件加解密）、密钥管理方式、自定义加解密注解 | ⏳ 待补充 |
| 横切 | 限流熔断降级 | [resiliencePattern.md](doc/crosscutting/resiliencePattern.md) | 限流实现方式、熔断机制、降级策略、重试机制、超时配置 | ❓ 未知 |
| 运行时 | 异步与线程池 | [asyncAndThreadPool.md](doc/runtime/asyncAndThreadPool.md) | 线程池定义与配置、@Async 使用方式及线程池绑定、异步编排方式、线程上下文传递 | ✅ 已完成 |
| 运行时 | 定时任务 | [scheduledTask.md](doc/runtime/scheduledTask.md) | 定时任务框架、任务定义方式、调度方式（单机/分布式）、任务监控与告警 | ⏳ 待补充 |
| 运行时 | 事件驱动 | [eventDriven.md](doc/runtime/eventDriven.md) | 领域事件定义方式、事件发布/订阅机制、事件处理方式（同步/异步）、事件溯源 | ❓ 未知 |
| 工程化 | 部署 | [deploy.md](doc/devops/deploy.md) | 部署架构、选主机制、环境差异管理、容器化方式、升级流程、健康检查配置 | ✅ 已完成 |
| 工程化 | 构建与CI | [buildAndCI.md](doc/devops/buildAndCI.md) | 构建工具及配置、构建产物、CI/CD 流水线、制品发布流程、代码质量检查工具 | ⏳ 待补充 |
| 运维 | 运维指南 | [opsGuide.md](doc/ops/opsGuide.md) | 日志目录与查看方式、常用运维命令、虚拟机登录方式、虚拟机/节点名称与角色、服务启停方式、日常巡检项 | ⏳ 待补充 |
```

**内容索引填写原则**：
- ✅ 写"虚拟机登录方式"——指向能力主题
- ❌ 不写"虚拟机 IP 是 10.0.0.1，用户名 admin"——不写具体细节
- ✅ 写"缓存 Key 设计规范、更新/失效策略"——指出知识范畴
- ❌ 不写"Key 格式为 `业务:实体:ID`，过期时间 30 分钟"——不写具体规则
- 对于已创建的维度文档，内容索引应从该文档的实际内容中提炼，而不是照搬模板默认值
- 对于 `⏳ 待补充` 或 `❓ 未知` 状态的维度，内容索引列写模板中的默认索引主题，待文档完成后再从实际内容更新

## 维度与代码路径映射
（供 update 模式快速定位使用）

| 维度文件 | 关键代码路径 |
|----------|-------------|
| authenticationProcess.md | src/main/java/com/xx/filter/, src/main/java/com/xx/security/ |
| aopAndInterceptor.md | src/main/java/com/xx/aspect/, src/main/java/com/xx/interceptor/ |
| logging.md | logback.xml, log4j2.xml, src/main/java/com/xx/config/LogConfig.java |

## 工作原则

1. **先探索再提问**：绝不空手提问，每个问题提出前必须先在代码中寻找答案
2. **通用而非具体**：记录通用能力和定位方式，不记录具体业务流程
3. **不存在就不创建**：项目没有的维度不创建文件，不留空占位
4. **确认而非猜测**：探索发现的内容需要与用户确认后再写入文档
5. **增量而非全量**：update 模式只分析变更部分，不全量重新扫描
6. **实用而非完美**：文档的目的是帮助快速定位和理解，不是替代代码阅读
7. **尊重用户节奏**：用户不想回答的维度标记待补充，不纠缠
8. **保持文档精炼**：每个维度文档控制在合理长度，重点突出、层次清晰

## 非Java项目适配

虽然维度规范以 Java/Spring Boot 为主要示例，但本技能适用于所有类型的项目。对于非 Java 项目：

- Python 项目：关注 Django/Flask/FastAPI 框架特征、中间件、装饰器、Celery 等
- Node.js 项目：关注 Express/Koa/NestJS 框架特征、中间件、事件循环等
- Go 项目：关注 Gin/Echo 框架特征、中间件链、goroutine 等
- 其他语言类推

维度文件名和目录结构保持一致，内容根据实际技术栈调整。
