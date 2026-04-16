# AI 辅助后端开发知识库（可选扩展）

> 本文件为**可选参考**。当候选人表示对 AI 编程工具有兴趣或经验时（如使用 Copilot/Cursor/Claude Code 等工具辅助后端开发），面试官可从本知识库中选题，作为 AI 能力模块的扩展提问。
>
> 覆盖方向：AI 辅助后端开发最佳实践（Spring Boot、数据库、API 设计）。
>
> **注意**：AI 编程工具原理、Agent Loop 架构、上下文管理等通用内容见 `ai-dev-knowledge-base.md`。

---

## 目录

- [1. AI 辅助后端开发](#1-ai-辅助后端开发)
  - [1.1 适合 AI 辅助的后端任务](#11-适合-ai-辅助的后端任务)
  - [1.2 不适合 AI 辅助的后端任务](#12-不适合-ai-辅助的后端任务)
  - [1.3 AI 辅助代码审查](#13-ai-辅助代码审查)
- [2. Spring Boot 项目中的 AI 辅助](#2-spring-boot-项目中的-ai-辅助)
- [3. 数据库相关的 AI 辅助](#3-数据库相关的-ai-辅助)
- [4. API 设计与 AI](#4-api-设计与-ai)

---

## 1. AI 辅助后端开发

### 1.1 适合 AI 辅助的后端任务

**什么任务适合 AI 编程？**
- **样板代码生成**：CRUD 模板、DTO 转换、工具类
- **API 调用示例**：调用第三方 API（支付、短信、地图）的 SDK 使用
- **代码重构**：提取方法、拆分大类、命名优化
- **Bug 修复**：根据报错信息生成修复建议
- **测试用例生成**：单元测试、集成测试用例
- **代码解释**：复杂逻辑的注释生成
- **文档生成**：API 文档、README 注释
- **设计模式应用**：将一段代码改写为设计模式实现

### 1.2 不适合 AI 辅助的后端任务

**什么任务不适合 AI 编程？**
- **复杂业务逻辑设计**：需要深入理解业务规则和边界情况
- **系统架构决策**：技术选型、模块划分、依赖管理
- **性能优化**：需要实际压测数据支撑
- **安全性敏感代码**：认证授权、密码处理、加密解密
- **新技术/框架的第一次使用**：缺乏训练数据，容易出错
- **数据库迁移**：Schema 变更需要精确控制

### 1.3 AI 辅助代码审查

**如何使用 AI 做代码审查？**
1. **提供完整上下文**：代码 diff、相关测试结果
2. **明确审查重点**：性能、安全、可读性、异常处理
3. **分阶段审查**：先看业务逻辑，再看技术实现

**AI 代码审查的优势**
- 发现浅层问题：命名不规范、重复代码、缺少空检查
- 快速理解陌生代码
- 提供改进建议和示例代码

**AI 代码审查的局限**
- 复杂业务逻辑仍需人工判断
- 无法发现运行时问题（空指针、并发问题）
- 对项目的整体架构理解有限

---

## 2. Spring Boot 项目中的 AI 辅助

**典型使用场景**

1. **Service 层代码生成**
```
Prompt: "用 @Service 注解写一个 UserService，包含
  - @Autowired UserRepository
  - getUserByEmail(String email) 方法
  - createUser(UserCreateRequest request) 方法，使用 BCrypt 加密密码
  要求使用 Lombok 简化代码"
```

2. **Controller 层代码生成**
```
Prompt: "用 @RestController 写一个 UserController，
  - GET /users/{id} 返回 UserDTO（需要用 @Mapper 转换）
  - POST /users 接收 UserCreateRequest，返回 201
  - 使用 @Valid 做参数校验
  - 异常统一用 @ControllerAdvice 处理"
```

3. **单元测试生成**
```
Prompt: "为 UserService.generateApiKey() 方法写单元测试，
  - 使用 Mockito 模拟 UserRepository
  - 验证返回的 API key 格式为 user_xxx
  - 使用 JUnit 5 和 AssertJ"
```

---

## 3. 数据库相关的 AI 辅助

**SQL 查询优化**
```
Prompt: "优化以下 SQL 查询，要求：
  1. 分析执行计划
  2. 添加适当的索引
  3. 解释为什么需要这些索引

原始 SQL：
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending'
AND o.created_at > '2024-01-01'
ORDER BY o.created_at DESC"
```

**JPA 查询方法命名**
```
Prompt: "用 Spring Data JPA 的命名查询规则，
  写一个方法：查找订单状态为 pending，
  用户名包含给定关键字的订单列表
答案：List<Order> findByStatusAndUserNameContainingIgnoreCase(
      OrderStatus status, String userName);
```

**MyBatis XML 优化**
```
Prompt: "审查以下 MyBatis XML，识别 N+1 查询风险、
  索引使用情况、全表扫描隐患"
```

---

## 4. API 设计与 AI

**RESTful API 设计辅助**
```
Prompt: "为一个电商系统设计用户相关的 RESTful API，包含：
  - 用户注册、登录、登出
  - 获取用户信息、修改用户信息
  - 密码重置

要求：
  1. 遵循 RESTful 规范（GET/POST/PUT/DELETE）
  2. 路径命名规范
  3. 响应状态码规范
  4. 包含基本的错误处理
```

**API 文档生成**
```
Prompt: "为以下 Controller 接口生成 OpenAPI 3.0 文档注释，
  包含请求参数校验说明、响应码定义、错误示例"
```

---

## 各身份难度参考

| 题目 | 实习 | 应届 | 社招 1-3 年 |
|------|:----:|:----:|:----------:|
| AI 辅助后端任务选择 | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| AI 代码审查技巧 | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Spring Boot + AI 实战 | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 数据库 + AI 优化 | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
