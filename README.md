# Java Backend Interview Simulator

[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

使用自然语言与 AI 面试官互动，模拟真实中国大厂（字节、腾讯、阿里、美团、快手）Java 后端技术面试。支持按求职者身份和面试官风格定制，全程记录状态并输出结构化评估报告。

---

## 核心特性

**多身份模拟** — 针对不同阶段候选人设计差异化问题：日常实习、暑期实习、校招（应届）、社招 1-3 年。难度和追问深度随身份自动调整。

**六种面试官风格** — 从"严厉拷打"到"温和鼓励"，从"工程实践"到"深挖学术"，覆盖不同偏好。同一场面试，不同体验。

**实时状态跟踪** — 面试全程维护结构化状态（已考察点、候选人弱点、待跟进项），避免重复提问，确保评估连贯。

**高频技术题库** — 基于 JavaGuide 整理，覆盖并发编程、JVM、MySQL、Redis、Spring、分布式系统等大厂常考方向，新增非共享内存并发模型（Actor、Lock-free、MQ）。

**深度追问链** — 简历关键词（CAS、volatile、多级缓存、RAG 等 14 个高频技术点）自动触发 3-5 层追问链，逼近底层原理与权衡，避免"听过名词"式过关。

**完整评估反馈** — 面试结束后输出综合评分、各维度评分及风格化改进建议，新增"追问深度"评估维度，反馈措辞随面试官风格变化。

**简历与 JD 支持** — 提供简历和目标岗位 JD 可获得定制化问题，简历关键词自动预生成追问链，面试后输出 JD 匹配度分析。

**编码题** — 可选环节，面试末尾安排并发编程、设计模式、数据库等手写代码题，难度随身份调整。

**AI 辅助开发考察** — 当候选人展示 AI 编程工具使用经验时，自动深入提问 Agent Loop、上下文管理、Spring Boot AI 辅助、RAG 量化验证等方向。

---

## 快速开始

### 选择你的运行环境

本 skill 兼容 OpenClaw Skills 规范，可在多种平台运行。**推荐手机微信用户走 QClaw + 云端 OpenClaw 路线**，零配置即可开始。

| 你的使用场景 | 推荐路径 | 安装方式 |
|------------|---------|---------|
| 手机微信，想随时随地练 | **QClaw + 云端 OpenClaw** | 让 AI 帮你装（见下方） |
| 企业微信 / 飞书 / 钉钉 / QQ | **WorkBuddy** | 让 AI 帮你装 |
| 飞书重度用户 | **AutoClaw** | 让 AI 帮你装 |
| 本地深度使用、想看源码 | **Claude Code（桌面端）** | 手动复制文件（见下方） |

> **关于"复制 GitHub 目录"的疑问**：除了本地 Claude Code 需要手动复制，其他云端/远程平台都**不需要**你手动操作仓库目录，直接让 AI 安装即可。

### 方式一：让 AI 帮你装（推荐）

把下面这句话连同本仓库链接一起发给 AI（QClaw / WorkBuddy / AutoClaw / Claude Code 都支持）：

```
帮我安装这个 skill：https://github.com/Hazehacker/java-backend-interview-simulator
```

AI 会自动拉取仓库、放到正确的 skills 目录、确认安装成功。然后你就可以直接说"开始模拟面试"触发。

### 方式二：手动安装（仅本地 Claude Code）

将仓库中 `SKILL.md` 和 `references/` 目录复制到 Claude Code 的 skills 目录：

- Windows：`C:\Users\<你的用户名>\.claude\skills\java-backend-interview-simulator\`
- macOS/Linux：`~/.claude/skills/java-backend-interview-simulator/`

重启 Claude Code 后即可触发。

### 触发面试

安装完成后，在对话中直接表达面试意图即可：

```
开始一场模拟面试
我是应届生，想练习 Java 后端面试，温和鼓励型
帮我准备字节后端实习面试
```

系统会依次确认：求职者身份、面试时长（30 / 40 / 45 / 60 分钟）、面试官风格、是否提供简历、是否包含编码题。

### 语音输入（强烈推荐）

文字输入面试问题太慢，**用语音模拟才接近真实面试节奏**：

| 你在哪里聊天 | 语音方式 |
|------------|---------|
| Windows 电脑（任何输入框） | **Win + H** 系统语音转文字 |
| Mac（任何输入框） | **Fn + Fn** 双击启动语音输入 |
| 微信 / QQ | 长按已发送的语音消息 → "转文字" 后再发 |
| 企业微信 / 飞书 | 直接发送语音消息（远程接入支持自动转写） |

---

## 进阶用法

### 控制命令

面试过程中可使用以下命令：

- **继续** — 进入下一环节
- **跳过** — 跳过当前问题
- **结束** — 提前结束并生成报告
- **换个风格** — 实时切换面试官风格
- **反馈** — 查看当前评分

### 简历与 JD 支持

提供简历和/或目标岗位 JD 可获得更有针对性的项目深挖问题和技术考察方向。面试结束后会输出 JD 匹配度分析报告。将简历文本或 JD 内容直接发送给 AI 即可。

---

## 项目结构

```
java-backend-interview-simulator/
├── SKILL.md                        # 技能主文件（OpenClaw skill 格式）
├── README.md                       # 本文档
├── references/
│   ├── tech-knowledge-base.md      # 技术知识库（高频面试题与答案要点）
│   ├── evaluation-rubric.md        # 评分细则与分人群反馈模板（含 JD 匹配分析）
│   ├── interviewer-styles.md        # 六种面试官风格详解
│   ├── coding-challenges.md        # 手写编码题库（可选环节）
│   ├── ai-dev-knowledge-base.md   # AI 应用开发知识库（Agent Loop/上下文管理，可选扩展）
│   └── ai-dev-tools-knowledge-base.md # AI 辅助后端开发知识库（Spring Boot/数据库/API，可选扩展）
├── JavaGuide/                      # 题库来源（子模块，不直接影响运行时）
└── LICENSE
```

---

## 技术架构

本项目基于 **OpenClaw Skill 框架**构建，采用三层加载架构：

| 层级 | 文件 | 作用 |
|------|------|------|
| **元数据** | `SKILL.md` frontmatter | 触发条件与能力概述，始终加载 |
| **主体逻辑** | `SKILL.md` body | 面试流程、风格适配、状态跟踪规则 |
| **知识库** | `references/*.md` | 按需加载：技术题库、评分标准、风格指南 |

面试流程采用**有限状态机**思路：破冰 → 项目深挖 → 技术考察 → AI 能力（可选）→ 编码题（可选）→ 总结反馈，各阶段时长和深度由身份和剩余时间共同决定。

---

## 当前局限

- **面试时长依赖估算**：实际面试中时间分配是动态的，本项目按参考时长设计，但不做精确计时
- **不支持实时语音**：当前为文字交互，暂不涉及口语表达评估（表达能力通过文字间接评估）
- **题库覆盖有边界**：主要覆盖 Java 后端高频方向，Rust/Go/C++ 等方向暂不涉及
- **简历解析有限**：可读取简历文本，但暂不支持结构化简历解析（PDF/图片格式）

---

## 贡献指南

欢迎提交补充，以下是几个优先方向：

- **题库扩充** — 补充更多高频面试题及参考答案（参考 `references/tech-knowledge-base.md` 格式）
- **风格优化** — 改进现有面试官风格的话术或追问逻辑
- **场景扩充** — 添加更多场景设计题（如秒杀系统、分布式 ID、即时通讯等）
- **文档改进** — 修正错误、提升可读性

参与流程：

1. Fork 本仓库
2. 创建分支 (`git checkout -b feature/your-feature`)
3. 提交改动 (`git commit -m 'Add: 简短描述'`)
4. 推送分支 (`git push origin feature/your-feature`)
5. 提交 Pull Request

---

## 许可证

本项目基于 [MIT License](LICENSE) 开源。

技术知识库部分引用了 [JavaGuide](https://github.com/Snailclimb/JavaGuide) 的高频面试题整理，仅作为面试考察点使用，不改变其原有许可。

---

## 致谢

- [JavaGuide](https://github.com/Snailclimb/JavaGuide) — 高频面试题来源
- [OpenClaw / Claude Code](https://claude.com/code) — AI 能力支撑
