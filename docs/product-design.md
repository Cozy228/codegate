# CodeGate 产品设计

## 背景

AI coding 正在让个人和团队更快地产生代码，但 brownfield 项目里最大的风险不是“代码是不是 AI 写的”，而是已有项目测试不足、业务逻辑隐含、架构边界脆弱，AI 或人类都可能在测试全绿的情况下改坏关键行为。

CodeGate 的产品目标是帮助个人、小团队和企业在提高编码效率的同时控制风险：任何变更进入主干或生产前，都必须证明它没有扩大不可控影响。

## 一句话定位

CodeGate 是面向 brownfield codebase 的代码变更门禁系统，用 facts + rules + evidence 判断一次变更是否可以合并。

## 核心原则

### 1. 不识别 AI 代码

识别“这是不是 AI 写的”不可靠，也不是最核心问题。CodeGate 关注的是变更本身的风险、影响范围和证据是否充分。

### 2. AI 不做裁判

LLM 可以作为可选增强，用于解释失败原因、生成测试建议、挖掘候选业务规则或把自然语言规范转成规则草稿。

最终 allow / warn / block 必须由确定性的 facts + rules 决定。

### 3. 证据优先于观点

报告不应该说“AI 认为这里有风险”，而应该说：

- changed-line coverage 低于阈值
- mutation survived
- public API changed but contract evidence missing
- security annotation removed
- transaction boundary changed without approval

### 4. 只拦新增风险

brownfield 项目不能一接入就全量清算历史旧账。CodeGate 应该使用 baseline + ratchet：

- 旧问题记录为 baseline
- 新增问题阻断
- 风险变差阻断
- 敏感变更缺证据阻断
- 历史问题不因当前 PR 自动失败

### 5. 一套产品覆盖不同用户

不拆成个人版、小团队版、企业版三套实现。个人、本地 CI、企业内网都运行同一个 CodeGate runner，只是入口、配置、报告归档位置不同。

## 目标用户

### 个人开发者

使用 Cursor、Claude Code、Copilot 等工具快速修改老项目，希望提交前知道 AI 是否越界、是否缺测试证据。

### 小团队

需要一个 PR required check，避免高风险变更在测试不足时合并。

### 金融企业

已有 Java brownfield 系统、Jenkins / GitLab / Sonar / Fortify / Checkmarx / JaCoCo 等工具，需要一个离线、可审计、可复现的变更门禁。

## V1 目标场景

第一版聚焦：

> Java 金融 brownfield app 的 PR / MR 变更门禁。

典型系统特征：

- Spring Boot / Spring MVC / Spring Security
- Maven / Gradle 多模块
- JUnit / JaCoCo
- PIT 可作为 mutation evidence
- 现有 SAST / SCA / Sonar / Fortify / Checkmarx 可通过报告导入
- 内网运行，不依赖 SaaS 或公网模型

## 核心工作流

```text
1. 读取当前 PR / MR diff
2. 提取 changed files / classes / methods / lines
3. 调用语言 analyzer 生成语义 facts
4. 导入测试、覆盖率、mutation、SARIF、JUnit 等 evidence
5. 与 baseline 对比，过滤历史旧问题
6. 使用 typed rule evaluator 评估规则
7. 生成 decision 和 evidence report
8. CI 根据结果 allow / warn / require evidence / require approval / block
```

## 决策状态

CodeGate 不应该只有 pass / fail。

- `ALLOW`: 证据满足要求
- `WARN`: 有风险信号，但不阻断
- `REQUIRE_EVIDENCE`: 缺少测试、覆盖率、mutation、契约、回滚等证据
- `REQUIRE_APPROVAL`: 需要 owner / security / DBA / domain approval
- `BLOCK`: 明确违反不可接受规则

## V1 重点门禁

V1 不追求覆盖所有风险，而是先打穿高价值闭环：

- changed-line coverage for sensitive changes
- mutation evidence for money / auth logic
- Spring Security annotation change detection
- transaction boundary change detection
- database migration risk detection
- API / public contract change evidence
- baseline + new-risk-only blocking

## 非目标

V1 不做：

- 自动修复
- 聊天式 code review
- AI 代码比例识别
- 全语言深度分析
- 复杂 dashboard
- 企业审批系统
- 对业务规则的全自动理解

## 产品护城河

CodeGate 的长期核心不是某个 Java parser，而是：

- 稳定的语言无关 Fact Model
- analyzer protocol
- evidence aggregation
- baseline / ratchet
- typed rule evaluator
- audit-friendly decision report
- brownfield change safety workflow

新增语言时，只新增 analyzer；core 的 rule、baseline、decision、report、CI adapter 不重写。
