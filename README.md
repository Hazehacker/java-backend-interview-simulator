# Java Backend Interview Simulator

[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

使用自然语言与 AI 面试官互动，模拟真实中国大厂（字节、腾讯、阿里、美团、快手）Java 后端技术面试。支持按求职者身份和面试官风格定制，全程记录状态并输出结构化评估报告。

---

## 核心特性

**多身份模拟** — 针对不同阶段候选人设计差异化问题：日常实习、暑期实习、校招（应届）、社招 1-3 年。难度和追问深度随身份自动调整。

**六种面试官风格** — 从"严厉拷打"到"温和鼓励"，从"工程实践"到"深挖学术"，覆盖不同偏好。同一场面试，不同体验。

**实时状态跟踪** — 面试全程维护结构化状态（已考察点、候选人弱点、待跟进项），避免重复提问，确保评估连贯。

**高频技术题库** — 基于 JavaGuide 整理，覆盖并发编程、JVM、MySQL、Redis、Spring、分布式系统等大厂常考方向。

**完整评估反馈** — 面试结束后输出综合评分、各维度评分及风格化改进建议，反馈措辞随面试官风格变化。

---

## 快速开始

### 环境要求

- [Claude Code](https://claude.com/code) (桌面端)
- Node.js 18+

### 安装方式

将 `SKILL.md` 和 `references/` 目录复制到 Claude Code 的 skills 目录：

```markdown
# 克隆项目
git clone https://github.com/Hazenix/java-backend-interview-simulator.git

# 或手动复制
将 SKILL.md 和 references/ 放入 ~/.claude/skills/your-skill-name/
```

### 使用方式

在 Claude Code 对话框中直接表达面试意图即可触发：

```
# 直接开始（系统会引导你选择身份和风格）
开始一场模拟面试

# 提供更多信息可跳过配置步骤
我是应届生，想练习字节的面试，温和鼓励型风格
```

首次使用，系统会依次确认：

1. **求职者身份** — 日常实习 / 暑期实习 / 校招 / 社招 1-3 年
2. **面试时长** — 30 / 40 / 45 分钟
3. **面试官风格** — 严厉拷打 / 温和鼓励 / 专业高效 / 深挖学术 / 工程实践 / 综合平衡
4. **是否提供简历** — 如有可按经历定制项目深挖问题

---

## 项目结构

```
java-backend-interview-simulator/
├── SKILL.md                        # 技能主文件（Claude Code skill 格式）
├── references/
│   ├── tech-knowledge-base.md      # 技术知识库（高频面试题与答案要点）
│   ├── evaluation-rubric.md        # 评分细则与分人群反馈模板
│   └── interviewer-styles.md       # 六种面试官风格详解
├── JavaGuide/                      # 题库来源（子模块，不直接影响运行时）
├── .claude/                        # Claude Code 本地配置
└── LICENSE
```

---

## 技术架构

本项目基于 **Claude Code Skill 框架**构建，采用三层加载架构：

| 层级 | 文件 | 作用 |
|------|------|------|
| **元数据** | `SKILL.md` frontmatter | 触发条件与能力概述，始终加载 |
| **主体逻辑** | `SKILL.md` body | 面试流程、风格适配、状态跟踪规则 |
| **知识库** | `references/*.md` | 按需加载：技术题库、评分标准、风格指南 |

面试流程采用**有限状态机**思路：破冰 → 项目深挖 → 技术考察 → AI 能力 → 压力测试 → 总结反馈，各阶段时长和深度由身份和剩余时间共同决定。

---

## 当前局限

- **面试时长依赖估算**：实际面试中时间分配是动态的，本项目按参考时长设计，但不做精确计时
- **不支持实时语音**：当前为文字交互，暂不涉及口语表达评估（表达能力通过文字间接评估）
- **题库覆盖有边界**：主要覆盖 Java 后端高频方向，Rust/Go/C++ 等方向暂不涉及
- **简历解析有限**：可读取简历文本，但暂不支持结构化简历解析（PDF/图片）

如有问题或建议，欢迎提交 Issue 或 Pull Request。

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
- [Claude Code](https://claude.com/code) — AI 能力支撑
