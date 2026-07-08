---
description: 扫描并分析项目代码，从多维度生成结构化知识文档，帮助 Agent 快速了解项目架构与通用能力。支持 init（初始化分析）、update（增量更新）、explore（定向探索）三种模式。
---

# Project Analyst — 项目分析助手

将项目代码转化为结构化的知识文档，让 Agent 快速理解项目的框架能力、通用能力和架构约定，无需每次从头阅读代码。

## 核心理念

**代码就是说明书。** 文档记录的是：

| ✅ 应该记录 | ❌ 不应该记录 |
|------------|--------------|
| 通用能力与实现方式（如：鉴权在中间件层通过 JWT 校验 Token） | 创建某个资源的完整调用链条 |
| 入口与定位信息（如：AOP 切面定义在 `aop/` 目录下） | 某个接口所有参数的处理逻辑 |
| 约定与规范（如：统一返回体为 `{code, data, msg}`） | 具体业务规则的细节描述 |

## 三种模式

### 模式识别

根据用户输入自动判断模式：

| 触发词 | 模式 |
|--------|------|
| `init`、`初始化`、`分析项目`、`开始分析` | 【init】初始化及项目分析 |
| `update`、`更新`、`增量`、`刷新` | 【update】增量更新 |
| `explore`、`探索`、`深入`、`看看` + 具体方向 | 【explore】探索模式 |

---

## 【init】初始化及项目分析

### 步骤 1：创建目录结构

在**被分析的项目**根目录创建：
```
.analysed/
├── overview.md
├── L1-基础技术栈/
│   ├── 编程语言及版本.md
│   ├── 核心框架及版本.md
│   └── ...
├── L2-项目架构/
├── L3-接口与通信/
├── ...
```

### 步骤 2：快速扫描识别

扫描项目，自动识别以下信息（**不要向用户提问，直接从代码中读取**）：

1. **编程语言**：统计文件后缀，主语言 + 辅助语言
2. **构建文件**：`pom.xml` / `build.gradle` / `package.json` / `Cargo.toml` / `go.mod` / `pyproject.toml` / `*.csproj` 等，提取核心依赖
3. **框架特征**：从依赖和配置文件推断主框架
4. **目录结构**：顶层目录和关键子目录
5. **配置文件**：所有配置文件汇总
6. **入口文件**：`main` 方法、`index.ts`、`app.py`、`main.go`、`Program.cs` 等

输出确认格式：
```
📋 自动识别结果：

| 项目 | 识别结果 |
|------|---------|
| 主语言 | TypeScript 5.x |
| 构建工具 | pnpm (package.json) |
| 核心框架 | NestJS 10.x |
| 模块结构 | 单仓库，src/ 下按功能分子目录 |
| 测试框架 | Jest + Supertest |

这些信息正确吗？有需要补充的吗？
```

### 步骤 3：逐维度确认

- 先判断是否多模块/微服务项目；若是，询问分析全部还是指定模块
- 按 L1→L13 顺序逐层确认（L14 为用户自定义，随时可插入）
- **关键原则：先搜索再提问，绝不空手提问**

对每个维度的流程：
1. 在代码中搜索相关证据（见下方维度搜索策略表，根据项目语言选择对应的搜索模式）
2. 找到证据 → 告知用户发现了什么，给选项确认/纠正
3. 未找到 → 告知"未发现相关实现"，跳过
4. 不确定 → 列出猜测和证据，询问用户

**尊重用户节奏**：
- 说"跳过" → 标记 `⏳待补充`，不再追问
- 说"不需要" → 标记 `❌不适用`
- 不知道且代码查不到 → 标记 `❓未知`
- 批量反馈 → 一次性处理，高效推进
- 连续跳过 ≥ 5 个维度 → 询问"后续是否加快节奏，改为批量确认？"

**确认后深挖**：
- 具体实现位置（文件路径 + 行号）
- 关键类/函数/装饰器/注解/接口
- 配置项和约定

### 步骤 4：生成文档

对每个已确认的维度生成文档，写入 `.analysed/`。

#### overview.md 模板

核心要求：**overview.md 是所有维度文档的索引**，Agent 读它就能判断任务涉及哪些维度、去哪里找详细内容。

```markdown
# {{项目名称}} 项目分析总览

> 📅 分析时间：{{当前时间}}
> 🔖 基于 Git 提交：`{{commit_sha}}`
> 📝 分析工具：Project Analyst

## 项目概览

| 项目 | 信息 |
|------|------|
| 项目名称 | {{项目名称}} |
| 主语言 | {{主语言及版本}} |
| 辅助语言 | {{辅助语言，无可省略}} |
| 核心框架 | {{框架1}}、{{框架2}} |
| 构建工具 | {{工具}}，编译命令：`{{build_cmd}}` |
| 模块结构 | {{单体 / 多模块 / Monorepo}}，共 {{N}} 个模块 |
| 运行环境 | {{JDK / Node / Python / Go 版本要求}} |
| 启动入口 | {{入口文件路径}} |
| 启动命令 | `{{run_cmd}}` |

## 快速启动

```bash
# 安装依赖
{{install_cmd}}

# 编译
{{build_cmd}}

# 启动
{{run_cmd}}

# 测试
{{test_cmd}}
```

## 维度索引

> 图例：✅=已分析 ❌=不适用 ⏳=待补充 ❓=待确认

### L1 基础技术栈
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 编程语言及版本 | [📄](./L1-基础技术栈/编程语言及版本.md) | Java 17，Kotlin 1.9 用于协程 | ✅ |
| 核心框架及版本 | [📄](./L1-基础技术栈/核心框架及版本.md) | Spring Boot 3.2、Spring Cloud 2023.0 | ✅ |
| 包管理与构建工具 | [📄](./L1-基础技术栈/包管理与构建工具.md) | Gradle 8.x，`./gradlew build` | ✅ |
| 运行时环境 | [📄](./L1-基础技术栈/运行时环境.md) | JDK 17、Docker 镜像 `eclipse-temurin:17-jre` | ✅ |

### L2 项目架构
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 模块/子项目结构 | [📄](./L2-项目架构/模块结构.md) | 5 个 Gradle 子模块：api/core/domain/infra/common | ✅ |
| 代码分层与命名规范 | [📄](./L2-项目架构/代码分层.md) | controller→service→repository，包名 `com.xxx.{模块}` | ✅ |
| 服务架构类型 | [📄](./L2-项目架构/服务架构.md) | 微服务，Nacos 注册中心 | ✅ |
| 启动流程与入口 | [📄](./L2-项目架构/启动流程.md) | `ApiApplication.main()`，加载顺序见文档 | ✅ |

### L3 接口与通信
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| API 定义 | [📄](./L3-接口与通信/API定义.md) | REST via Spring MVC，`/api/v1/*` | ✅ |
| 请求/响应规范 | [📄](./L3-接口与通信/请求响应规范.md) | 统一返回体 `Result<T>{code, msg, data}`，分页用 `PageResult` | ✅ |
| 序列化机制 | [📄](./L3-接口与通信/序列化.md) | Jackson，日期 `yyyy-MM-dd HH:mm:ss`，null 不输出 | ✅ |
| 服务间调用 | [📄](./L3-接口与通信/服务间调用.md) | OpenFeign + Sentinel 熔断，定义在 `feign/` 包 | ✅ |
| 消息队列与事件 | [📄](./L3-接口与通信/消息队列.md) | RocketMQ，Topic 清单见文档 | ✅ |

### L4 数据层
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 数据源定义 | [📄](./L4-数据层/数据源.md) | MySQL 8.0 主从，HikariCP 连接池，Druid 监控 | ✅ |
| ORM/数据访问 | [📄](./L4-数据层/ORM.md) | MyBatis-Plus 3.5，Mapper 在 `mapper/` 包 | ✅ |
| 数据库版本管理 | [📄](./L4-数据层/数据库版本.md) | Flyway，脚本在 `resources/db/migration/` | ✅ |
| 缓存策略 | [📄](./L4-数据层/缓存.md) | Redis Cluster + Caffeine 本地缓存，`@Cacheable` 注解 | ✅ |
| 读写分离/分库分表 | — | — | ❌ |

### L5 安全
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 认证机制 | [📄](./L5-安全/认证.md) | Spring Security + JWT，Token 刷新见文档 | ✅ |
| 鉴权/权限模型 | [📄](./L5-安全/鉴权.md) | RBAC，`@PreAuthorize` 注解，角色定义在 `enums/RoleType` | ✅ |
| 过滤器/拦截器链 | [📄](./L5-安全/过滤器链.md) | JwtFilter → RateLimitFilter → LogFilter，顺序见文档 | ✅ |
| 加解密与签名 | [📄](./L5-安全/加解密.md) | AES 加密敏感配置，RSA 签名验签 | ✅ |
| 安全防护 | [📄](./L5-安全/安全防护.md) | CORS 配置、XSS Filter、SQL 参数化 | ✅ |
| 敏感数据脱敏 | [📄](./L5-安全/脱敏.md) | `@Sensitive` 注解，手机号/身份证/银行卡 | ✅ |

### L6 横切关注点
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| AOP 切面定义 | [📄](./L6-横切关注点/AOP.md) | `aop/` 包：日志切面、权限切面、分布式锁切面 | ✅ |
| 异常处理体系 | [📄](./L6-横切关注点/异常处理.md) | `@ControllerAdvice` 全局捕获，`BizException` 自定义异常 | ✅ |
| 日志框架与追踪 | [📄](./L6-横切关注点/日志.md) | Logback，MDC 传递 traceId，ELK 输出 | ✅ |
| 国际化 | — | — | ❌ |
| 错误码体系 | [📄](./L6-横切关注点/错误码.md) | `ErrorCode` 枚举，格式：模块(2位)+错误(3位) | ✅ |
| 审计能力 | [📄](./L6-横切关注点/审计.md) | `@AuditLog` 注解 + AOP，记录到 `t_audit_log` 表 | ✅ |
| 请求上下文传递 | [📄](./L6-横切关注点/请求上下文.md) | `RequestContext` ThreadLocal，Header 透传 userId/traceId | ✅ |

### L7 中间件与基础设施
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 中间件配置 | [📄](./L7-中间件/中间件.md) | Redis、RocketMQ、ES 连接配置 | ✅ |
| 服务注册与发现 | [📄](./L7-中间件/服务注册.md) | Nacos，namespace 隔离环境 | ✅ |
| 配置中心 | [📄](./L7-中间件/配置中心.md) | Nacos Config，`@RefreshScope` 热刷新 | ✅ |
| 分布式协调 | [📄](./L7-中间件/分布式协调.md) | Redisson 分布式锁，`@DistributedLock` 注解 | ✅ |
| 第三方 SDK | [📄](./L7-中间件/第三方SDK.md) | 阿里云 OSS、企业微信、微信支付 | ✅ |

### L8 弹性与韧性
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 熔断/降级 | [📄](./L8-弹性/熔断降级.md) | Sentinel，Feign 降级工厂 `fallback/` 包 | ✅ |
| 限流策略 | [📄](./L8-弹性/限流.md) | Sentinel 网关层 + Nginx `limit_req_zone` | ✅ |
| 重试机制 | [📄](./L8-弹性/重试.md) | Spring Retry `@Retryable`，MQ 消费重试 3 次 | ✅ |
| 超时配置 | [📄](./L8-弹性/超时.md) | Feign 3s, DB 5s, 网关 30s | ✅ |
| 健康检查与优雅关闭 | [📄](./L8-弹性/健康检查.md) | Actuator `/health`，`GracefulShutdown` Hook | ✅ |
| 分布式事务 | — | — | ❌ |

### L9 任务与异步
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 定时任务 | [📄](./L9-任务/定时任务.md) | XXL-Job 2.4，任务清单见文档 | ✅ |
| 异步处理 | [📄](./L9-任务/异步.md) | `@Async("taskExecutor")` 线程池，核心 10 最大 50 | ✅ |
| 事件驱动 | [📄](./L9-任务/事件驱动.md) | Spring Event，`@EventListener`，事件类在 `event/` 包 | ✅ |
| 批处理 | — | — | ❌ |

### L10 可观测性
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 监控指标 | [📄](./L10-可观测性/监控指标.md) | Micrometer → Prometheus，`/actuator/metrics` | ✅ |
| 链路追踪 | [📄](./L10-可观测性/链路追踪.md) | SkyWalking Agent，traceId 自动注入 MDC | ✅ |
| 健康检查端点 | [📄](./L10-可观测性/健康检查.md) | `/health`、`/readiness`、`/liveness` | ✅ |
| 日志聚合与告警 | — | — | ⏳ |

### L11 测试与质量
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 单元测试框架 | [📄](./L11-测试质量/单元测试.md) | JUnit 5 + Mockito + AssertJ | ✅ |
| 集成测试 | [📄](./L11-测试质量/集成测试.md) | `@SpringBootTest`，Testcontainers MySQL | ✅ |
| 静态代码分析 | [📄](./L11-测试质量/静态分析.md) | SonarQube + Checkstyle (`checkstyle.xml`) | ✅ |
| 代码格式化规范 | [📄](./L11-测试质量/格式化.md) | `.editorconfig`，Spotless Gradle 插件 | ✅ |

### L12 部署与运维
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| CI/CD | [📄](./L12-部署运维/CICD.md) | Jenkinsfile，构建→单测→镜像→部署 | ✅ |
| 容器化 | [📄](./L12-部署运维/容器化.md) | Dockerfile 多阶段构建，K8s Deployment | ✅ |
| 环境配置管理 | [📄](./L12-部署运维/环境配置.md) | `application-{dev,test,prod}.yml` | ✅ |
| 运维手册 | [📄](./L12-部署运维/运维手册.md) | SSH: bastion→vm1，日志: `/var/log/app/`，进程: `java -jar` | ⏳ |

### L13 业务扩展
| 维度 | 文档 | 关键摘要 | 状态 |
|------|------|---------|:--:|
| 工作流/状态机 | — | — | ❌ |
| 规则引擎 | — | — | ❌ |
| 通知推送 | [📄](./L13-业务扩展/通知推送.md) | 企业微信 + 邮件，模板在 `templates/` | ✅ |
| 报表/导出 | [📄](./L13-业务扩展/报表导出.md) | EasyExcel，导出接口在 `export/` 包 | ✅ |

### L14 自定义维度
（按需追加）

## Agent 使用指引

阅读本文档后，请根据任务类型按需加载维度：
- **鉴权/权限问题** → L5 安全
- **接口/API 问题** → L3 接口与通信
- **数据库/缓存问题** → L4 数据层
- **异常/日志/追踪问题** → L6 横切关注点
- **性能/稳定性问题** → L8 弹性与韧性 + L10 可观测性
- **部署/环境问题** → L12 部署与运维
- **定时任务/异步问题** → L9 任务与异步
- **新增功能/架构设计** → L2 项目架构 + L3 接口与通信
```

#### 维度文档模板

```markdown
# {{维度名称}}

> 所属分类：{{分类}} | 状态：{{状态}} | 最后更新：{{时间}}

## 概述
(1-2 句话说明该维度在项目中的作用)

## 通用能力与实现

| 能力 | 实现方式 | 关键文件 | 备注 |
|------|---------|---------|------|
| | | | |

## 入口与定位

(关键入口：文件路径、类名、函数名、装饰器/注解、配置 key)

## 约定与规范

(项目在该维度的约定，如命名规范、返回体格式)

## 相关配置

(配置文件路径 + 关键配置项)

## 设计决策

(可选：为什么选这个方案)
```

### 步骤 5：写入 CLAUDE.md

在**被分析项目**根目录检查 `CLAUDE.md`：

- **已存在**：在末尾追加知识库使用提示
- **不存在**：创建 CLAUDE.md 并写入

```markdown

---

## 项目知识库

本项目已通过 Project Analyst 完成分析，知识文档位于 `.analysed/` 目录。

**使用规则**：
1. 接到任务后，**首先阅读** `.analysed/overview.md`，通过维度索引快速定位
2. 根据任务类型按需加载对应维度文档（overview.md 末尾有维度→任务映射表）
3. **不要**一次性加载全部文档，只读任务相关的维度
4. 文档记录通用能力和框架约定，具体业务逻辑仍需阅读代码

分析基准提交：`{{commit_sha}}`
```

---

## 【update】增量更新

### 步骤 1：获取变更范围

```bash
git diff --name-status {{old_sha}} {{new_sha}}
```

从 overview.md 获取 `{{old_sha}}`。

### 步骤 2：映射到维度

根据变更文件路径，对照 overview.md 中各维度文档的"关键文件"字段判断影响范围。

| 变更类型 | 是否更新 | 说明 |
|---------|:------:|------|
| 构建文件新增/删除框架依赖 | ✅ | 可能引入新能力或废弃旧能力 |
| 新增/修改 Filter/Interceptor/Middleware/AOP | ✅ | 横切关注点变更 |
| 新增/修改全局异常处理器 | ✅ | 异常处理体系变更 |
| 新增/修改全局配置（`application.yml`、`.env` 等） | ✅ | 评估具体变更项 |
| 新增模块/子项目 | ✅ | L2 项目架构 |
| 新增 Controller/Handler/Route | ⚠️ 仅评估 | 确认有无新规范 |
| 业务逻辑代码（Service/Repository 等） | ❌ | 不记录业务细节 |
| 注释/文档/测试 | ❌ | 不影响通用能力 |

### 步骤 3：更新文档

对受影响的维度：告知用户 → 确认 → 更新维度文档 → 更新 overview.md 关键摘要和 commit_sha

### 步骤 4：检测新框架

检查构建文件中的新增依赖，即使没在 diff 的关键文件列表中出现，也可能引入了新框架能力。

---

## 【explore】探索模式

### 步骤 1：明确方向

用户给出方向但未说明维度时，列出最接近的选项：
```
🔍 你想探索的方向，以下哪个更接近？
A. L4.4 缓存策略
B. L7.1 中间件配置
C. 其他（请描述）
D. 新增自定义维度
```

### 步骤 2：深入探索

1. 先读 overview.md 了解概况
2. 已有文档 → 在基础上扩展；无文档 → 从代码搜索
3. 追踪：入口 → 实现 → 配置 → 调用关系
4. 遵循"通用能力"原则，不深入业务

### 步骤 3：交互确认

呈现结果 → 用户确认 → 创建/更新维度文档 → 更新 overview.md 索引

完成后主动建议相关维度：
```
💡 探索异常处理时发现它和日志追踪（L6.3）、错误码体系（L6.5）紧密关联，需要继续吗？
```

---

## 维度搜索策略（按语言通用）

> 使用前先根据步骤 2 识别的语言选择对应列的模式。若项目是多语言混合，按主语言为主、辅助语言补充。

### L1 基础技术栈

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 1.1 语言版本 | 构建文件中的版本声明 | `pom.xml`→`<java.version>`，`build.gradle`→`sourceCompatibility` | `pyproject.toml`→`requires-python`，`.python-version` 文件 | `go.mod`→`go 1.xx` | `package.json`→`engines.node`，`tsconfig.json`→`target` | `*.csproj`→`<TargetFramework>` |
| 1.2 核心框架 | 顶级框架依赖 | Spring Boot、Quarkus、Micronaut 的 starter 依赖 | Django→`django`，FastAPI→`fastapi`，Flask→`flask` | Gin→`gin`，Echo→`echo`，Fiber→`fiber` | NestJS→`@nestjs/core`，Express→`express`，Next.js→`next` | ASP.NET Core→`Microsoft.AspNetCore` |
| 1.3 构建工具 | 构建文件和锁文件 | Maven→`pom.xml`，Gradle→`build.gradle`+`settings.gradle` | Poetry→`pyproject.toml`，pip→`requirements.txt`，uv→`uv.lock` | Go modules→`go.mod`+`go.sum` | npm→`package-lock.json`，pnpm→`pnpm-lock.yaml`，yarn→`yarn.lock` | MSBuild→`*.sln`+`*.csproj`，`dotnet` CLI |
| 1.4 运行时 | 运行时版本要求 | `Dockerfile`→FROM 镜像，`.sdkmanrc`，`system.properties` | `Dockerfile`→FROM，`runtime.txt` | `Dockerfile`→FROM | `Dockerfile`→FROM，`.nvmrc`，`.node-version` | `Dockerfile`→FROM，`global.json`→SDK 版本 |

### L2 项目架构

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 2.1 模块结构 | 多模块/workspace 定义 | `settings.gradle`→`include`，父 `pom.xml`→`<modules>` | `pyproject.toml`→`[tool.poetry.workspaces]` | `go.work` 文件，`go.mod` 的 replace 指令 | `pnpm-workspace.yaml`，`lerna.json`，`turbo.json`，Nx workspace | `.sln`→Project 列表 |
| 2.2 代码分层 | 源码顶层目录结构 | `src/main/java/` 下的包结构 | `src/` 或项目名目录下的子目录 | `cmd/`、`internal/`、`pkg/` 目录 | `src/` 下的子目录结构 | `src/` 下的项目/命名空间结构 |
| 2.3 服务架构 | 注册中心/网关/服务间调用 | `spring-cloud-starter`、Dubbo、Nacos/Eureka 依赖 | `uvicorn` workers、Gunicorn、K8s Service | 是否使用 go-micro、Kratos 等微服务框架 | NestJS microservices、`@nestjs/microservices` | Ocelot 网关、Service Fabric |
| 2.4 启动入口 | 主函数/启动脚本 | `@SpringBootApplication` 类，`main()` 方法 | `app.py`、`main.py`、`asgi.py`/`wsgi.py` | `cmd/` 目录下的 `main.go` | `main.ts`、`index.ts`、`server.ts` | `Program.cs`、`Startup.cs` |

### L3 接口与通信

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 3.1 API 定义 | 路由/端点定义 | `@RestController`、`@GetMapping`、`@PostMapping` | FastAPI→`@app.get`，Django→`urls.py`，Flask→`@app.route` | Gin→`router.GET`，Echo→`e.GET`，标准库→`http.HandleFunc` | NestJS→`@Controller`+`@Get`，Express→`app.get`，Hono→`app.get` | `[ApiController]`+`[Route]`，Minimal API→`app.MapGet` |
| 3.2 请求响应规范 | 统一返回体类/函数 | `Result<T>`、`ApiResponse`、`R<T>` 等通用响应类 | `JSONResponse`、Pydantic 模型 | 统一 response struct，JSON 序列化标签 | 统一 response 格式，DTO 定义 | `ActionResult<T>`、`IActionResult`，统一响应模型 |
| 3.3 序列化 | JSON/Protobuf 配置 | Jackson/Gson 注解，`ObjectMapper` 配置 | Pydantic→`model_dump`，`json.dumps` default 参数 | `json.Marshal`，struct tag，protobuf | `class-transformer`，`JSON.stringify` replacer | `System.Text.Json`，`Newtonsoft.Json`，protobuf-net |
| 3.4 服务间调用 | RPC/HTTP 客户端 | Feign→`@FeignClient`，RestTemplate，Dubbo→`@DubboReference` | httpx、aiohttp、gRPC client | gRPC→`google.golang.org/grpc`，标准库 `net/http` | `@nestjs/microservices`，gRPC，axios | HttpClient、Refit、gRPC client |
| 3.5 消息队列 | MQ 依赖和配置 | RocketMQ/Kafka/RabbitMQ starter，`@RocketMQMessageListener` | Celery、RQ、kafka-python、aio-pika | kafka-go、sarama、amqp | `@nestjs/microservices`（Kafka/RMQ），BullMQ，amqplib | MassTransit、NServiceBus、RabbitMQ.Client |

### L4 数据层

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 4.1 数据源 | 数据库连接配置 | `spring.datasource`、Druid、HikariCP 配置 | `DATABASE_URL`、SQLAlchemy→`create_engine`、Django→`DATABASES` 配置 | `database/sql`、GORM→`gorm.Open`，连接字符串 | TypeORM→`DataSource`、Prisma→`datasource` 块、Knex 配置 | `ConnectionStrings`、DbContext→`OnConfiguring` |
| 4.2 ORM | 数据访问层模式 | MyBatis→Mapper XML，JPA→Repository，jOOQ | SQLAlchemy ORM、Django ORM、Peewee | GORM、sqlx、ent | TypeORM→`@Entity`，Prisma→`schema.prisma`，Drizzle | EF Core→DbContext+DbSet，Dapper |
| 4.3 DB 版本管理 | 迁移脚本目录 | Flyway→`db/migration/`，Liquibase→changelog | Alembic→`alembic/versions/`，Django→`migrations/` | golang-migrate→`migrations/` | Prisma→`prisma/migrations/`，TypeORM→`migrations/` | EF Core→Migrations 目录 |
| 4.4 缓存 | 缓存依赖和注解/装饰器 | `@Cacheable`，Redis/Caffeine，JetCache | `@cache`(Django)，fastapi-cache，redis-py | go-redis，ristretto，bigcache | `@nestjs/cache-manager`，ioredis，lru-cache | `IMemoryCache`/`IDistributedCache`，StackExchange.Redis |
| 4.5 读写分离 | 分库分表/数据源路由 | ShardingSphere，`@DS` 注解，AbstractRoutingDataSource | SQLAlchemy→bind，Django→DATABASE_ROUTERS | GORM→多数据源 plugin，sharding | TypeORM→多数据源，Prisma→多 datasource | EF Core→多 DbContext |

### L5 安全

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 5.1 认证 | 认证框架和方式 | Spring Security、Shiro、JWT Filter、OAuth2 Client | Django Auth、FastAPI→OAuth2PasswordBearer、JWT | Gin→JWT middleware，OAuth2 proxy | Passport.js、`@nestjs/passport`、NextAuth.js | ASP.NET Identity、JWT Bearer、Cookie Auth |
| 5.2 鉴权 | 权限注解/装饰器 | `@PreAuthorize`、`@RolesAllowed`、`@RequiresPermissions` | Django→`@permission_required`，FastAPI→Depends | Casbin、自定义 middleware→ctx 注入权限 | `@Roles`、`@Permissions`(NestJS)，CASL | `[Authorize(Roles="")]`，Policy-based auth |
| 5.3 过滤器链 | Filter/Interceptor/Middleware | `@WebFilter`、`HandlerInterceptor`、`OncePerRequestFilter` | Django→MIDDLEWARE，FastAPI→middleware，Starlette→Middleware | Gin→`router.Use`，Echo→`e.Use`，标准库→`http.Handler` 链 | NestJS→`@Injectable`+`implements NestMiddleware`，Express→`app.use` | Middleware→`app.Use()`，Filter→`[ServiceFilter]` |
| 5.4 加解密 | 加密工具类和注解 | `@Encrypt`、AES/RSA 工具类，`CipherUtils` | cryptography、PyCryptodome，自定义 `encrypt` 函数 | crypto/aes、crypto/rsa 标准库 | crypto 模块，`bcrypt`，`crypto-js` | `System.Security.Cryptography`，`DataProtection` API |
| 5.5 安全防护 | CSRF/XSS/CORS 配置 | CSRF Token、XSS Filter、CorsFilter、Spring Security headers | Django→CSRF middleware，FastAPI→CORSMiddleware | Gin→CORS middleware，secure headers | helmet、cors、csrf(S express)，NestJS→CorsModule | CSRF→`AddAntiforgery`，CORS→`AddCors`，XSS 编码 |
| 5.6 数据脱敏 | 脱敏注解/工具 | `@Sensitive`、`@JsonSerialize`(自定义)、日志脱敏 Regex | Pydantic→自定义 validator，日志 filter | struct tag→自定义 marshal，日志脱敏 | class-transformer→`@Transform`，自定义 interceptor | `[SensitiveData]` attribute，自定义 JsonConverter |

### L6 横切关注点

| 维度 | 搜索什么 | Java / JVM | Python | Go | TypeScript / Node | .NET |
|------|---------|-----------|--------|-----|-------------------|------|
| 6.1 AOP 切面 | AOP/装饰器的定义 | `@Aspect`+`@Around`/`@Before`，AspectJ 注解 | Django→`@receiver`(signals)，FastAPI→Depends | Gin→middleware（模拟 AOP） | NestJS→`@Interceptor`、`@Guard`、`@Pipe`、`@Middleware` | `[ActionFilter]`、`[ExceptionFilter]`，Castle DynamicProxy |
| 6.2 异常处理 | 全局异常处理器 | `@ControllerAdvice`+`@ExceptionHandler`，`ErrorController` | FastAPI→`@app.exception_handler`，Django→`handler500` | Gin→`router.Use(Recovery())`，Echo→`e.HTTPErrorHandler` | NestJS→`@Catch`+`ExceptionFilter`，Express→error middleware | `[ExceptionHandler]`、`IExceptionHandler`，middleware→`UseExceptionHandler` |
| 6.3 日志追踪 | 日志框架配置 | Logback→`logback-spring.xml`，Log4j2，MDC→traceId | logging→`logging.conf`，loguru，structlog | zap、logrus、slog | winston、pino、NestJS Logger | Serilog、NLog、`ILogger<T>` |
| 6.4 国际化 | i18n 消息文件 | `messages*.properties`，`MessageSource` Bean | Django→`locale/`，gettext，babel | go-i18n→`active.*.toml` | i18next→`locales/`，nestjs-i18n | `.resx` 资源文件，`IStringLocalizer<T>` |
| 6.5 错误码 | 错误码枚举/常量 | `ErrorCode`/`ResultCode` 枚举，`code.properties` | 自定义 Exception 类+code 属性，`errors.py` | errors package→错误常量，`errors.New` | 自定义 Exception 类+code，`error-code.ts` | `ProblemDetails`，自定义 `ApiException`，error code enum |
| 6.6 审计 | 审计日志注解/AOP | `@Audit`、`@AuditLog`+AOP，JPA→`@EntityListeners` | Django→`Auditlog`，自定义 decorator | 自定义日志 middleware | NestJS→`@AuditLog`，TypeORM→`@AfterLoad` subscriber | `[Audit]` attribute，`SaveChanges` 拦截 |
| 6.7 请求上下文 | ThreadLocal/上下文传递 | `RequestContext`(ThreadLocal)、`RequestContextHolder` | `contextvars`、`threading.local`、FastAPI→`request.state` | `context.WithValue`，自定义 context | NestJS→`@Req`+request scope，AsyncLocalStorage | `HttpContextAccessor`、`AsyncLocal<T>`、`CallContext` |

### L7 中间件与基础设施

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 7.1 中间件配置 | 中间件连接配置类/文件 | 搜索：Redis 连接工厂、MQ 配置 Bean/Provider、ES Client 初始化代码、中间件地址和凭证配置 |
| 7.2 服务注册发现 | 注册中心配置 | 搜索：`nacos`/`eureka`/`consul`/`etcd` 相关的配置文件、服务名定义、`@EnableDiscoveryClient` / `register` 函数 |
| 7.3 配置中心 | 配置中心 client 配置 | 搜索：`nacos config`/`apollo`/`consul`/`vault` 配置、`@RefreshScope`/动态刷新相关代码、`bootstrap.yml`/`bootstrap.properties` |
| 7.4 分布式协调 | 分布式锁/选举 | 搜索：`lock`/`distributed`/`leader` 相关的导入和实现、`Redisson`/`etcd`/`Zookeeper` 使用 |
| 7.5 第三方 SDK | 第三方服务依赖 | 搜索：OSS/SMS/Push/支付/地图等 SDK 的 maven/npm/go.mod 依赖、Client 初始化代码、`*Client`/`*SDK` 类 |

### L8 弹性与韧性

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 8.1 熔断降级 | 熔断器配置 | 搜索：`sentinel`/`hystrix`/`resilience4j`/`circuit-breaker` 依赖和规则配置、fallback/降级方法 |
| 8.2 限流 | 限流器配置 | 搜索：`ratelimit`/`rate_limit`/`RateLimiter`/`throttle`、网关限流配置 |
| 8.3 重试 | 重试注解/装饰器 | 搜索：`retry`/`Retryable`/`@Retry`、MQ 消费者中的重试逻辑、`max_retries` |
| 8.4 超时 | 超时配置项 | 搜索：`timeout`/`connect-timeout`/`read-timeout`/`request-timeout`、HTTP Client/Database 超时设置 |
| 8.5 健康检查 | 健康检查端点 | 搜索：`health`/`healthcheck`/`readiness`/`liveness`/`actuator`、`graceful`/`gracefully`/`shutdown`/`PreDestroy` |
| 8.6 分布式事务 | 分布式事务框架 | 搜索：`seata`/`tcc`/`saga`/`transaction` + `distributed`、事务消息配置 |

### L9 任务与异步

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 9.1 定时任务 | 调度框架 | 搜索：`scheduled`/`cron`/`Schedule`/`Job`/`quartz`/`xxl-job`、`celery` beat、`@Schedule` 装饰器、`cron` 表达式 |
| 9.2 异步处理 | 异步注解/装饰器 | 搜索：`@Async`/`async`/`await`/`go routine`/`thread_pool`/`ThreadPool`/`executor`、线程池/协程池配置 |
| 9.3 事件驱动 | 事件发布/监听 | 搜索：`event`/`emit`/`fire`/`publish`/`subscribe`/`EventListener`/`listener`、EventBus/RxJS Subject |
| 9.4 批处理 | 批处理框架 | 搜索：`batch`/`job`/`step`/`chunk`/`spring-batch`、批量处理相关代码 |

### L10 可观测性

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 10.1 监控指标 | Metrics 采集 | 搜索：`metrics`/`prometheus`/`micrometer`/`meter`/`gauge`/`counter`、`@Timed` 注解/装饰器 |
| 10.2 链路追踪 | Tracing 配置 | 搜索：`tracing`/`trace`/`skywalking`/`jaeger`/`zipkin`/`opentelemetry`/`otel`、Agent 配置 |
| 10.3 健康端点 | 就绪/存活探针 | 搜索：`/health`/`/ready`/`/live`/`healthcheck` 路由、`HealthIndicator`/自定义健康检查 |
| 10.4 日志聚合 | 日志输出格式 | 搜索：`logstash`/`fluentd`/`elk`/`elasticsearch`、JSON 日志格式配置、`filebeat` |

### L11 测试与质量

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 11.1 单元测试 | 测试框架依赖 | 搜索：`junit`/`pytest`/`go test`/`jest`/`vitest`/`xUnit`/`nUnit`、`@Test` 注解、`test_` 前缀函数、`_test.go` 文件、`*.test.ts` 文件 |
| 11.2 集成测试 | 集成测试配置 | 搜索：`testcontainers`/`docker-compose`（测试用）、`@SpringBootTest`/`integration`/`conftest`、测试数据库配置 |
| 11.3 静态分析 | 代码检查配置 | 搜索：`sonar`/`checkstyle`/`eslint`/`ruff`/`golangci-lint`/`StyleCop`、规则配置文件 |
| 11.4 格式化 | 格式化配置 | 搜索：`.editorconfig`/`prettier`/`spotless`/`black`/`gofmt`/`rustfmt`/`dotnet-format`、格式化规则 |

### L12 部署与运维

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 12.1 CI/CD | CI 配置文件 | 搜索：`Jenkinsfile`/`.github/workflows/`/`.gitlab-ci.yml`/`.circleci/`/`azure-pipelines.yml`/`Makefile` |
| 12.2 容器化 | 容器配置文件 | 搜索：`Dockerfile`/`docker-compose.yml`/`.dockerignore`/`deployment.yaml`/`helm`、K8s 清单 |
| 12.3 环境配置 | 多环境配置 | 搜索：`application-{env}`/`.env`/`.env.*`/`config.{env}`/`appsettings.{env}`、环境变量列表 |
| 12.4 运维手册 | 运维相关文件 | 搜索：启动脚本(`start.sh`/`run.sh`)、systemd unit 文件、`supervisord.conf`、`pm2` 配置、堡垒机/SSH 信息（**需用户提供**） |

### L13 业务扩展

| 维度 | 搜索什么 | 通用搜索模式（跨语言） |
|------|---------|---------------------|
| 13.1 工作流 | 流程引擎 | 搜索：`flowable`/`activiti`/`camunda`/`temporal`/`cadence`/`workflow`/`state machine` 依赖 |
| 13.2 规则引擎 | 规则引擎依赖 | 搜索：`drools`/`easy-rules`/`json-rules-engine`/`rule engine`、规则定义文件(`.drl`/`.json`) |
| 13.3 通知推送 | 通知 SDK | 搜索：`sms`/`mail`/`email`/`wechat`/`dingtalk`/`feishu`/`slack`/`notify`、消息模板 |
| 13.4 报表导出 | 报表/导出库 | 搜索：`export`/`report`/`excel`/`pdf`/`csv`/`jasper`/`poi`/`openpyxl`/`excelize` |

### L14 自定义维度

用户提出的项目特有维度，流程同探索模式。

---

## 全局原则

1. **先探索再提问**：每个问题必须先在代码中搜索答案，给用户选项而非空问
2. **语言无关**：根据步骤 2 识别的语言选择对应搜索模式，禁止对 Python 项目搜索 Java 注解
3. **聚焦通用能力**：除 L12.4 运维手册外，只记录通用能力和框架约定
4. **不存在的维度不创建**：代码中无相关实现且用户确认不需要的，标记 `❌不适用`，不创建目录和文件
5. **所有发现需确认**：主动发现的内容也要经用户确认才能写入文档
6. **update 只分析变更**：不做全量扫描
7. **实用为主**：文档目的是快速定位和理解项目，不是替代代码阅读。保持精炼
8. **尊重用户节奏**：用户说跳过就跳过，不纠缠
9. **overview.md 是核心**：它是索引，Agent 靠它定位维度 → 按需加载文档，你的任务是保证它准确、完整、一目了然
