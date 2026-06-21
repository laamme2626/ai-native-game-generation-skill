# AI Native 游戏生成 Skill

一个可复用的工程化 Skill，用于构建、调试、验证和编写 AI Native 小游戏生成平台相关文档。

本 Skill 重点关注：

* 基于自然语言的游戏创建；
* 结构化 `game_spec` 生成；
* Multi-Agent 或分阶段生成工作流；
* 可控的 Runtime Builder 设计；
* `manifest.json` / `index.html` 制品协议；
* 基于 iframe 沙箱的游戏运行机制；
* 回退生成器（fallback generator）与可选 LLM Provider 集成；
* 常见故障模式，例如 `undefined` 输出、运行时不匹配、Play 页面空白、iframe 键盘焦点问题、seed 重复以及密钥泄露；
* 面向公开仓库的 README、验证文档以及 AI 协作文档编写规范。

## Skill 位置

```text
skills/game-generation/SKILL.md
```

## 本 Skill 的用途

当开发能够根据用户提示生成可玩轻量级 Web 游戏的项目时，可以使用本 Skill。

它旨在帮助维护安全、可调试的系统架构：

```text
用户输入
→ 需求解析
→ 安全检查
→ 结构化 Game Spec 生成
→ Schema / QA 验证
→ 可控 Runtime Builder
→ 制品存储
→ 沙箱化 Play Runtime
```

核心原则是：

LLM 应当生成结构化的游戏规格（Game Spec），而不是直接生成不可控的运行时代码。

## 涵盖主题

* AI Native 小游戏平台架构
* Multi-Agent 生成工作流
* Runtime 制品协议
* iframe 沙箱隔离机制
* fallback 与 OpenAI 兼容 Provider 设计
* 支持与不支持的游戏类型处理
* 运行时调试
* 验证检查清单
* 安全检查清单
* 面向公开仓库的文档编写规范

## 推荐使用方式

在使用 Codex 或其他编程助手时，建议在修改 AI Native 游戏生成平台之前，先让其阅读：

```text
skills/game-generation/SKILL.md
```

以确保开发过程遵循统一、可调试且安全的工程实践。
