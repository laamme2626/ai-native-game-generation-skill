---

# AI Native 小游戏平台工程技能

## 何时使用本技能

当开发、调试、扩展、验证或编写 AI Native 小游戏生成平台相关文档时，可以使用本技能。

特别适用于包含以下能力的项目：

* 自然语言生成游戏；
* 多 Agent 或分阶段生成流程；
* 结构化 `game_spec` 输出；
* 生成运行时产物，例如 `manifest.json` 和 `index.html`；
* 基于 iframe 沙箱的游戏运行；
* 本地或远程对象存储；
* fallback 生成器以及可选的真实 LLM Provider；
* 创建、预览、发布、游玩、收藏、Remix、管理作品等用户流程。

目标是让平台具备：

* 安全性；
* 可调试性；
* 可游玩性；
* 对当前能力边界保持诚实。

---

## 核心原则

AI Native 小游戏平台不应该允许模型自由生成不受控制的应用代码。

更安全、更易维护的架构如下：

```text
用户 Prompt
→ 需求解析
→ 安全检查
→ 结构化 Game Spec 生成
→ Schema / QA 校验
→ 受控 Runtime Builder
→ 产物存储
→ 沙箱运行环境
```

LLM 应当生成受约束的 `game_spec` 或类似结构化数据。

真正的运行时代码应由可信模板或受控组件生成，而不是直接信任模型输出。

---

## 典型架构

一个实用的 MVP 通常包含：

* 首页、创建页、任务详情页、游戏详情页、Play 页、我的作品等前端页面；
* Auth、生成任务、游戏、发布、收藏、Remix、统计等后端 API；
* 用户、游戏、任务、日志、收藏、发布状态等数据库表；
* 游戏产物对象存储；
* 多 Agent 或分阶段编排器；
* 将结构化 Spec 转换为 HTML 的 Runtime Builder；
* iframe 沙箱运行页面；
* 本地 / 离线 / Demo 模式下的 fallback generator；
* 可选的真实 LLM Provider。

常见产物目录：

```text
generated/games/{gameId}/
  game_spec.json
  manifest.json
  index.html
```

其中：

* `game_spec.json`：结构化游戏协议；
* `manifest.json`：暴露标题、描述、标签、入口 URL 和 spec URL；
* `index.html`：游戏运行入口；
* Play 页面动态读取 manifest，并在 iframe 中启动游戏。

---

## 推荐的多 Agent 工作流

一个 AI Native 游戏生成流程可以由多个 Agent 实现，也可以由单一编排器分阶段完成。

推荐阶段：

```text
Requirement Parser Agent
Safety Check Agent
Game Designer Agent
QA Validator Agent
Runtime Builder Agent
Storage Publisher Agent
```

### Requirement Parser Agent（需求解析 Agent）

职责：

* 解析用户 Prompt；
* 识别游戏类型；
* 提取主题、角色、目标、难度、风格、时长；
* 检测不支持或过于复杂的玩法；
* 检测商业 IP 或风格模仿风险；
* 输出标准化生成请求。

不要只依赖关键词匹配。

很多简单描述背后可能隐藏复杂需求。

例如：

```text
Generate a cozy life simulation game like Animal Crossing.
```

它可能意味着：

* 长周期模拟经营；
* 村庄探索；
* NPC 社交关系；
* 建造与装饰；
* 经济系统；
* 资源循环；
* 商业 IP 风险。

如果平台无法支持，应明确返回“不支持”，或者在说明原因后进行降级。

---

### Safety Check Agent（安全检查 Agent）

职责：

* 拒绝危险 Prompt；
* 防御 Prompt Injection；
* 限制输入长度；
* 校验上传文件类型和大小；
* 防止泄露密钥或生成危险代码；
* 防止模型覆盖系统规则。

---

### Game Designer Agent（游戏设计 Agent）

职责：

* 生成结构化 game spec；
* 保证输出合法 JSON；
* 遵循 Schema；
* 保持内容与用户需求一致；
* 避免生成任意 JavaScript；
* 避免输出 Runtime 无法渲染的字段。

这是最主要的 LLM 集成点。

---

### QA Validator Agent（质量校验 Agent）

职责：

* 校验 Schema；
* 检查必填字段；
* 检查类型专属字段；
* 检查 Runtime 是否存在；
* 检查游戏是否具备目标、输入、反馈和重开机制；
* 检测可见的 `undefined`；
* 检测类型与 Runtime 不匹配；
* 在保存错误产物之前快速失败。

---

### Runtime Builder Agent（运行时构建 Agent）

职责：

* 选择正确模板；
* 根据结构化 Spec 生成 HTML/CSS/JS；
* 避免直接执行模型生成代码；
* 保证游戏可玩；
* 包含重开逻辑；
* 提供默认容错；
* 避免白屏和 `undefined`。

---

### Storage Publisher Agent（存储发布 Agent）

职责：

* 写入生成产物；
* 更新数据库；
* 保存 manifest URL、entry URL、spec URL；
* 清理旧文件；
* 记录路径但不泄露敏感信息。

---

## 游戏类型设计

每一种游戏类型都应包含：

```text
类型定义
→ Spec Schema
→ Runtime Template
→ QA Validator
```

不要只在 Prompt 层增加一个新的 `type`。

例如新增 `side_battle` 时，应同时增加：

* 类型注册；
* 玩家、敌人、HP、攻击、防御、移动、胜负规则等字段；
* fallback 生成逻辑；
* LLM 输出约束；
* Runtime 模板；
* 键盘和点击输入；
* QA 校验；
* 手动测试；
* 文档更新。

常见轻量类型：

```text
choice_adventure
quiz
clicker
memory
dodge
escape_room
```

实验类型：

```text
side_battle
runner
platformer
```

复杂类型通常应拒绝或明确降级：

```text
3D open world
MMO
large racing simulation
complex RTS
sandbox building
deckbuilding combat
rhythm chart game
life simulation with persistent world
complex management simulation
```

---

## Runtime 产物协议

即使在本地开发环境，也应将生成游戏视为远程产物。

推荐 manifest：

```json
{
  "id": "game-id",
  "title": "Game Title",
  "description": "Short description",
  "tags": ["tag1", "tag2"],
  "entry": {
    "url": "/generated/games/game-id/index.html"
  },
  "specUrl": "/generated/games/game-id/game_spec.json"
}
```

Play 页面应：

1. 从数据库读取元数据；
2. 获取 `manifest.json`；
3. 校验字段；
4. 在 iframe 中加载 `entry.url`；
5. 保持 sandbox；
6. 显示加载状态；
7. 出错时显示错误，而不是白屏；
8. 成功启动后更新统计。

除非架构发生变化，不要用硬编码 React 组件替代生成游戏。

---

## Play Runtime 规则

Play 页面是最容易出现细节问题的地方。

应支持：

* 读取游戏信息；
* 加载 manifest；
* 启动 Runtime；
* Runtime 加载成功；
* 加载失败；
* 重试；
* 返回首页；
* 重开游戏；
* 返回任务详情；
* 草稿状态下执行发布。

iframe 应保持：

```html
<iframe sandbox="allow-scripts" src="..."></iframe>
```

如果游戏使用键盘控制：

* 添加 `tabindex`；
* 点击后聚焦 canvas；
* 在 iframe 内监听键盘事件；
* 不依赖父页面处理按键。

---

## LLM Provider 规则

平台应默认支持 fallback 模式。

推荐：

```env
LLM_PROVIDER=fallback
OPENAI_API_KEY=
OPENAI_MODEL=
OPENAI_BASE_URL=
```

真实模型模式：

```env
LLM_PROVIDER=openai-compatible
OPENAI_API_KEY=your_key
OPENAI_MODEL=your_model
OPENAI_BASE_URL=https://your-openai-compatible-endpoint/v1
```

规则：

* fallback 不依赖 API Key；
* 模型调用只能发生在服务端；
* 不要在客户端暴露 Key；
* 不要使用 `NEXT_PUBLIC_`；
* 不要把 Key 写入日志、数据库或产物；
* 输出必须解析为 JSON；
* 必须通过 Schema 校验；
* 非法输出应回退或明确失败；
* 不要静默保存错误产物。

重要区别：

```text
Codex 或开发工具使用的凭证
≠
项目运行时使用的 LLM Provider 凭证
```

不要混淆两者。

---

## Runtime Builder 原则

不要把模型当作代码生成器。

推荐：

```text
LLM 输出 game_spec
→ Runtime Builder 选择可信模板
→ 模板生成 index.html
```

避免：

```text
LLM 直接生成任意 JavaScript
→ 应用未经校验直接保存
```

未来如果增加 Runtime Planner Agent，应输出受约束的数据：

```json
{
  "runtimeType": "side_battle",
  "components": ["hp_bar", "keyboard_movement", "enemy_ai"],
  "difficulty": "easy"
}
```

而不是自由 JS。

---

## 常见失败模式与处理方式

### 1. 游戏界面出现 `undefined`

现象：

* 标题、敌人名称、HP、分数显示 `undefined`。

原因：

* Schema 与模板字段不一致；
* LLM 缺失字段；
* fallback 未填默认值；
* QA 未检查 HTML。

解决：

* 检查 `game_spec.json`；
* 检查模板字段；
* 增加默认值；
* 增加类型校验；
* 检查生成 HTML 中是否出现 `undefined`。

---

### 2. 游戏类型与 Runtime 不匹配

现象：

* 战斗游戏变成 dodge；
* 标题和玩法不一致。

解决：

* 建立类型注册表；
* 明确拒绝不支持类型；
* 联合校验 type 和字段；
* 不要静默降级。

---

### 3. Play 页面白屏

原因：

* manifest 路径错误；
* entry URL 错误；
* iframe src 错误；
* Runtime 崩溃。

解决：

* 检查数据库；
* 检查 manifest；
* 检查生成文件；
* 查看浏览器 Console；
* 增加错误提示和重试。

---

### 4. 键盘控制失效

原因：

* iframe 未获得焦点；
* keydown 注册位置错误。

解决：

* 添加 tabindex；
* 点击后聚焦；
* 在 iframe 内监听键盘事件。

---

### 5. 游戏能运行但不可玩

现象：

* 页面正常；
* 没有目标、反馈、挑战。

解决：

* 强制要求：

  * 开始条件；
  * 玩家输入；
  * 分数或 HP；
  * 胜负状态；
  * 重开机制。

---

### 6. 复杂需求被静默降级

现象：

用户要求 MMO，结果生成 clicker。

解决：

* 增加复杂度检测；
* 返回明确提示；
* 必要时提供简化版本。

---

### 7. 真实模型调用导致进度卡住

解决：

* 显示当前 Agent；
* 显示日志时间；
* 显示“正在调用模型”；
* 记录耗时。

---

### 8. UI 美化降低可用性

现象：

* 按钮看不清；
* 对比度过低；
* 卡片过于花哨。

解决：

* 优先保证可读性；
* 使用真实数据测试；
* 避免过度设计。

---

### 9. 收藏和统计状态不一致

解决：

* 使用唯一收藏记录；
* 保持数据库为唯一真相来源；
* 更新后重新验证。

---

### 10. Seed 数据重复

解决：

* 使用 upsert；
* 使用稳定 ID；
* 覆盖旧 Demo 数据。

---

### 11. 删除流程体验差

解决：

* 增加确认；
* 显示成功提示；
* 清理文件；
* 显示错误原因。

---

### 12. 文档过度宣传

解决：

* 明确 Demo 边界；
* 区分稳定功能和实验功能；
* 诚实描述未来规划。

---

## 开发原则

1. 不要随意重写项目。
2. 不要轻易更换技术栈。
3. 保留 fallback 模式。
4. 真实 LLM Provider 应保持可选。
5. API Key 只能存在于服务端。
6. 保持 iframe sandbox。
7. 不允许模型自由生成运行时代码。
8. 不要静默保存非法输出。
9. 不要强行映射不支持类型。
10. 不要用 Loading 掩盖错误。
11. 不要为了美观牺牲可读性。
12. 不要夸大 Demo 能力。
13. 保留用户未提交修改。
14. 修改后尽量执行 lint 和 build。
15. 文档应与实际行为保持一致。

---

## 常用验证命令

```bash
npm install
npm run db:generate
npm run db:migrate
npm run db:seed
npm run db
npm run lint
npm run build
npm run dev
```

Windows PowerShell：

```bash
npm.cmd run lint
npm.cmd run build
npm.cmd run dev
```

---

## 预期结果

遵循本技能构建的平台应具备：

* 可本地运行；
* 默认安全；
* 对 Demo 边界保持诚实；
* 支持多种轻量游戏类型；
* 支持动态加载生成产物；
* 可通过日志观察生成过程；
* 避免明显的密钥泄露；
* 在 Runtime、模板或 Play 页面出现问题时更容易调试。
