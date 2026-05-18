**You:**

有了一个新的idea，如何管理，控制AI 写的代码的质量并作为企业的生产门禁，搜索一下，briansotrm

* * *

**ChatGPT:**

我会按“市场已有方案 → 企业真实门禁点 → 可产品化的空白 → MVP 形态”来搜和整理。重点不是泛泛讲代码规范，而是判断它能不能变成一个可卖的企业生产门禁。

* * *

**ChatGPT:**

初步判断：这个方向有价值，但不能定位成“又一个代码扫描器”。已有玩家已经覆盖 SAST、secret、dependency、quality gate；真正的空白更像“AI 代码进入生产前的证据链 + 策略引擎 + 企业审批门禁”。

* * *

**ChatGPT:**

## 结论

这个 idea **可以做**，但要避开一个坑：

> 不要做“AI 代码 Review 工具”。  
> 要做“AI 代码进入生产前的企业级准入控制系统”。

因为 **AI Code Review / SAST / Secret Scan / Dependency Scan / Quality Gate** 已经有很多成熟玩家。真正有机会的是把这些能力组合成企业生产门禁：**谁写的、AI 写了多少、改了什么敏感区域、证据是否齐全、风险是否可接受、谁批准、能不能上线**。

---

## 为什么这个方向成立

几个信号很明显：

1. **AI 代码已经进入主流研发流** DORA 2025 相关分析提到，AI 提升了个人产出，但也引入了“verification tax”，也就是省下的编码时间被代码审查、验证、返工吃掉；并且有 30% 开发者对 AI 生成代码“little to no trust”。[Dora](https://dora.dev/insights/balancing-ai-tensions/)
2. **AI 代码不是天然安全** Veracode 2025 报告测试了 100+ 个模型在 Java、Python、C#、JavaScript 上生成的代码，结果是 45% 样本引入 OWASP Top 10 安全漏洞；Java 的安全失败率达到 72%。[Veracode](https://www.veracode.com/blog/genai-code-security-report/)
3. **已有厂商开始把“AI 代码质量门禁”产品化** SonarQube 已经有 **AI Code Assurance**，明确针对包含 AI 生成代码的项目设置额外质量标准、AI 专用 Quality Gate、Quality Profile 和 Badge。[Sonar Documentation](https://docs.sonarsource.com/sonarqube-server/2026.1/ai-capabilities/ai-code-assurance)
4. **GitHub、Semgrep、Snyk、腾讯、百度都在靠近这个问题，但角度不同** GitHub Advanced Security 覆盖 code scanning、secret scanning、dependency review、Copilot Autofix 等能力。[GitHub Docs](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) Semgrep 定位为 SAST、SCA、Secrets、AI-assisted security workflow，并明确提到 secure vibe coding。[Semgrep](https://semgrep.dev/) 腾讯 CodeBuddy 提供补全、单测生成、智能评审、代码修复等能力。[Tencent Cloud](https://cloud.tencent.com/document/product/1749?utm_source=chatgpt.com) 百度 Comate 的代码审查支持选择公司规范文件，根据企业规范审查代码。[Baidu Cloud](https://cloud.baidu.com/doc/COMATE/s/Vmbyr4c3x?utm_source=chatgpt.com)

所以市场方向是对的。但竞争也说明：**单点 AI Review 不够强，必须做成控制平面。**

---

## 这个产品应该是什么

我会把它定义成：

> **AI Code Production Gate / AI 代码生产准入平台**  
> 一个接在 GitHub / GitLab / CI/CD / IDE Agent 之后的企业级门禁系统，用策略、证据链和风险分级控制 AI 生成代码能否进入主干、测试、预发和生产。

它不是替代 Sonar、CodeQL、Semgrep、Snyk，而是把它们接进来，形成一个统一判断：

> 这次变更是否可以上线？

---

## 核心价值不是“扫描”，而是“准入判断”

现有工具大多回答：

| 工具类型 | 回答的问题 |
| --- | --- |
| SAST 静态安全扫描 | 代码里有没有安全漏洞 |
| Secret Scan | 有没有密钥泄露 |
| Dependency Scan / SCA | 依赖有没有漏洞或许可证风险 |
| Sonar Quality Gate | 复杂度、重复率、覆盖率、bug 是否达标 |
| AI Code Review | 这段 diff 有没有潜在问题 |
| SBOM / SLSA / Sigstore | 构建产物和供应链是否可信 |

你的产品应该回答：

> **在企业当前规则下，这个 AI 辅助生成的变更是否允许进入生产？如果不允许，缺什么证据？谁能豁免？豁免是否可审计？**

这才是企业愿意付钱的点。

---

## 可以设计成 5 层门禁

### 1. 变更来源门禁：这段代码怎么来的

记录每个 PR / MR 的来源：

| 字段 | 作用 |
| --- | --- |
| human-only | 人手写 |
| AI-assisted | AI 辅助补全 / 改写 |
| agent-generated | Agent 自主生成大部分代码 |
| copied-from-external | 外部粘贴 / StackOverflow / GitHub / 文档 |
| unknown | 无法判断，默认更高风险 |

这里不一定要精准判断“AI 写了百分之几”，因为很难可靠。更实用的是要求开发者/Agent 在 PR 模板里声明，并结合 commit pattern、IDE 插件、agent log、diff 特征做辅助判断。

门禁规则可以是：

```
如果 PR 标记为 agent-generated：
- 必须有需求说明
- 必须有测试证据
- 必须跑 SAST / SCA / Secret Scan
- 必须有人类 code owner approve
- 如果涉及 auth/payment/security/data migration，需要安全 reviewer approve
```

---

### 2. 证据门禁：没有证据，不准进生产

每个 PR 生成一个 **Production Readiness Passport**，可以叫“生产准入护照”。

内容类似：

```YAML
change_id: PR-1234
ai_origin: agent-generated
risk_level: high

evidence:
  requirement_link: present
  design_note: present
  unit_tests: passed
  integration_tests: passed
  coverage_delta: +2.1%
  typecheck: passed
  lint: passed
  sast: passed_with_warnings
  secret_scan: passed
  dependency_scan: failed
  license_scan: passed
  sbom: generated
  owner_approval: present
  security_approval: missing

decision:
  status: blocked
  reason:
    - high severity dependency vulnerability
    - security approval required for auth-related code
```

这比“AI 说这段代码有问题”更企业化，因为它是**可审计、可追踪、可复用**的。

---

### 3. 确定性检查门禁：LLM 不能单独当裁判

生产门禁里，LLM 只能做辅助判断，不能作为唯一 blocking reason。

应该优先依赖确定性检查：

| 类别 | 工具/机制 |
| --- | --- |
| 格式 | ESLint、Prettier、Checkstyle |
| 类型 | TypeScript、mypy、Java compile |
| 测试 | Unit / Integration / E2E |
| 覆盖率 | Coverage delta |
| 安全 | CodeQL、Semgrep、Sonar、Snyk |
| Secret | GitHub Secret Protection、Gitleaks |
| 依赖 | Dependabot、Snyk、OWASP Dependency-Check |
| 许可证 | FOSSA、ScanCode、ORT |
| IaC | Checkov、tfsec、OPA |
| SBOM | CycloneDX、SPDX |
| 供应链 | SLSA、Sigstore、provenance |

解释几个词：

* **SAST**：Static Application Security Testing，静态应用安全测试，不运行代码，直接分析源码漏洞。
* **SCA**：Software Composition Analysis，软件成分分析，主要查第三方依赖漏洞和许可证风险。
* **SBOM**：Software Bill of Materials，软件物料清单，记录软件由哪些组件、依赖、版本组成。
* **SLSA**：Supply-chain Levels for Software Artifacts，是软件供应链安全框架，用来提升构建和产物可信度。[SLSA](https://slsa.dev/)
* **OPA**：Open Policy Agent，通用策略引擎，可以把策略写成代码，用在 CI/CD、Kubernetes、API Gateway 等地方。[Open Policy Agent](https://openpolicyagent.org/docs?utm_source=chatgpt.com)

---

### 4. 语义风险门禁：LLM 做传统工具不擅长的事

LLM 适合检查这些：

| 检查项 | 例子 |
| --- | --- |
| 需求一致性 | 代码是否真的实现了 ticket / spec |
| 业务逻辑风险 | 折扣、权限、审批流、计费逻辑是否有漏洞 |
| 架构规则 | 是否绕过 BFF、是否直接访问数据库、是否违反分层 |
| 数据流风险 | 用户输入是否进入 SQL、日志、HTML、外部 API |
| 破坏性变更 | API schema、DB migration、配置变更是否兼容 |
| 测试缺口 | 改了核心逻辑但只补了 happy path |
| AI 幻觉痕迹 | 不存在的 API、错误 SDK 用法、伪造配置、无效异常处理 |

但 LLM 的结果应该是：

```
risk_signal: high
recommended_action: require_architecture_review
blocking: false
```

真正 block 的动作交给 policy engine：

```
if risk_signal == high and file_path contains /auth/:
    require security approval
```

这就是“AI 辅助判断 + 策略做最终裁决”。

---

### 5. 策略门禁：企业规则产品化

这层是产品核心。

用类似 OPA / Rego 的方式，把企业规则变成代码：

```rego
deny[msg] {
  input.ai_origin == "agent-generated"
  input.risk_level == "high"
  not input.approvals.security
  msg := "AI-generated high-risk change requires security approval"
}

deny[msg] {
  input.changed_paths[_] == "src/auth/*"
  input.test_coverage_delta < 0
  msg := "Auth-related changes cannot reduce test coverage"
}

deny[msg] {
  input.dependency_scan.high_vulns > 0
  msg := "High severity dependency vulnerabilities are not allowed"
}
```

企业真正想买的是这个：

> 不同业务、不同仓库、不同路径、不同风险等级，用不同门禁规则。

比如：

| 场景 | 门禁 |
| --- | --- |
| 文档改动 | 自动放行 |
| UI 样式改动 | 基础检查 + 一人 review |
| 普通后端逻辑 | 单测 + SAST + code owner |
| 登录 / 权限 / 支付 | 安全审批 + 测试证据 + 设计说明 |
| 数据库 migration | DBA / owner approval |
| Terraform / IAM | 平台团队审批 + OPA policy |
| Agent 自动生成大 PR | 拆分 PR，否则拒绝 |
| 无 ticket 的 AI PR | 拒绝进入主干 |

---

## 产品形态

### MVP 形态

做一个 GitHub App / GitLab App + CI Action。

最小闭环：

1. 监听 PR
2. 读取 diff、PR 描述、文件路径、测试结果、扫描结果
3. 调用现有工具：CodeQL / Semgrep / Sonar / Gitleaks / dependency scan
4. 用 LLM 做语义 review
5. 输出一个统一报告
6. 根据 policy 决定 pass / warn / block
7. 把结果写回 PR required check

PR 评论可以长这样：

```
AI Code Production Gate: BLOCKED

Risk level: High
AI origin: Agent-generated
Changed sensitive areas:
- src/auth/session.ts
- src/db/migrations/20260518_add_user_role.sql

Passed:
- Typecheck
- Unit tests
- Secret scan
- Formatting

Failed:
- Dependency scan: 1 high vulnerability
- Coverage delta: -4.8%
- Missing security approval
- Migration rollback plan missing

Required before merge:
1. Fix or justify vulnerable dependency
2. Add tests for session expiration and role downgrade
3. Add rollback plan
4. Request security reviewer approval
```

---

## 更强的版本：AI Code Passport

每个 PR、release、artifact 都生成一个“护照”。

包括：

| 维度 | 内容 |
| --- | --- |
| Origin | 人写 / AI 辅助 / Agent 生成 |
| Intent | 对应需求、ticket、设计文档 |
| Risk | 影响范围、敏感模块、风险等级 |
| Evidence | 测试、覆盖率、扫描、SBOM、审批 |
| Decision | 通过、阻断、豁免 |
| Audit | 谁批准、何时批准、为什么豁免 |
| Learning | 后续线上 bug 是否回流到规则 |

这会比普通 AI Review 更有壁垒，因为它和企业合规、审计、发布流程绑定。

---

## 竞品/已有方案怎么分层

| 类型 | 代表 | 已有能力 | 空白 |
| --- | --- | --- | --- |
| AI 编程助手 | Copilot、Cursor、CodeBuddy、Comate | 生成代码、补全、解释、局部 review | 不负责企业生产准入 |
| 代码质量平台 | SonarQube | Quality Gate、AI Code Assurance | 更偏代码质量，不一定覆盖企业审批流和多工具证据链 |
| AppSec 平台 | Semgrep、Snyk、Checkmarx、Veracode | SAST、SCA、Secrets、漏洞修复 | 不一定管理 AI 来源、需求一致性、agent 行为 |
| GitHub 安全能力 | GitHub Advanced Security | CodeQL、Secret、Dependency、Autofix | 生态内强，但跨工具、跨企业策略弱 |
| 供应链框架 | SLSA、SBOM、Sigstore | 构建产物可信、依赖透明 | 不判断业务语义和 AI 生成风险 |
| Policy-as-Code | OPA、Kyverno | 策略执行 | 需要上层产品把软件研发上下文喂进去 |

你的机会点：

> 把这些能力编排成一个“AI 时代的变更准入控制平面”。

---

## 可以切的 6 个产品方向

### 方向 1：AI PR Risk Scorer

只做 PR 风险评分。

输入：

* diff
* changed files
* ownership
* test result
* scan result
* AI origin
* ticket/spec
* historical incidents

输出：

```
risk_level: low / medium / high / critical
required_gates:
  - code owner review
  - security review
  - integration test
  - migration rollback plan
```

适合 MVP，容易做。

---

### 方向 2：Enterprise AI Coding Policy Engine

企业配置策略：

```
AI generated code cannot modify:
- authentication
- authorization
- encryption
- payment
- IAM
- production Terraform

unless:
- security approval exists
- tests passed
- threat model attached
```

这个更接近 B2B 价值。

---

### 方向 3：AI Code Evidence Ledger

每次 AI 生成、修改、上线都留下证据。

适合买给：

* 金融
* 医疗
* 政企
* 跨国企业
* 强审计行业

卖点是：

> 我不是帮你写代码，我帮你证明 AI 写的代码经过了可审计的控制。

---

### 方向 4：Agent Runtime Guardrail for Coding Agents

控制 Claude Code、Cursor、Copilot Agent、Devin、内部 Agent 的行为。

规则：

```
Agent cannot:
- modify production secrets
- edit IAM policy directly
- delete migration rollback
- change auth without approval
- commit generated code without tests
- open PR larger than 800 LOC
```

这个方向更前沿，但更难，需要 IDE / Git / CI 集成。

---

### 方向 5：AI-generated Code Quality Benchmark for Enterprise Repos

企业内部 benchmark：

```
同一个任务：
- Copilot 生成
- Claude Code 生成
- Gemini 生成
- Qwen Coder 生成
- Cursor Agent 生成

比较：
- 编译通过率
- 测试通过率
- 安全漏洞
- 代码复杂度
- 返工次数
- reviewer 耗时
```

卖点：

> 企业不用靠感觉选择 AI coding 工具，而是用自己代码库评测。

这个很适合做咨询/平台切入。

---

### 方向 6：AI Code Review Prompt + Rule Registry

企业维护一套代码审查规则库：

```
- Java Spring 安全规范
- React 前端规范
- Terraform 安全规范
- API 兼容性规范
- 日志脱敏规范
- 数据库 migration 规范
- BFF 架构规范
```

然后自动分发到：

* Copilot instructions
* Cursor rules
* Claude Code instructions
* Gemini instructions
* PR Review bot
* CI gate

GitHub 官方也提醒，不同 agent 对 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md` 等 instruction 文件支持不一致，需要测试和记录；组织级 custom instructions 应保持窄而明确，适合放安全、合规等不可谈判标准。[Well-Architected](https://wellarchitected.github.com/library/governance/recommendations/governing-agents/)

这个点很关键：**企业规则分发本身就是一个产品。**

---

## 我最建议的定位

不要叫：

> AI Code Review

太窄，且红海。

建议叫：

> **AI SDLC Control Plane**  
> 或  
> **AI Code Production Gate**

中文可以是：

> **AI 代码生产准入平台**  
> **AI 研发门禁控制台**  
> **AI 代码治理平台**

一句话 pitch：

> 企业正在大量使用 Copilot、Cursor、Claude Code 等 AI 编码工具，但缺少统一的准入控制。我们把 AI 代码来源、测试证据、安全扫描、架构规则、审批流程和发布门禁统一起来，让 AI 写的代码只有在满足企业策略后才能进入生产。

---

## 第一版不要做什么

别一上来做这些：

1. **不要做完整 IDE** Cursor、Copilot、CodeBuddy、Comate 已经太强。
2. **不要自己重做 SAST** 接 Semgrep / CodeQL / Sonar 比自己写扫描器现实。
3. **不要声称能准确识别 AI 代码百分比** 这个不可靠，也容易变成伪需求。
4. **不要让 LLM 单独决定是否阻断上线** 企业不会信。LLM 应该生成 risk signal，最终由 policy engine 和确定性证据决定。
5. **不要只做 PR 评论机器人** PR bot 容易被忽略，必须接 required check / merge rule / deployment gate。

---

## 第一版应该做什么

MVP 可以只做 4 件事：

### 1. PR Risk Classifier

根据 diff、路径、测试、扫描、AI origin 判断风险等级。

### 2. Evidence Collector

收集所有证据：

* test result
* coverage delta
* SAST result
* secret scan
* dependency scan
* code owner review
* ticket link
* design note
* AI declaration

### 3. Policy Engine

企业配置规则：

```
什么情况下 block
什么情况下 warn
什么情况下 require approval
什么情况下 allow
```

### 4. GitHub / GitLab Required Check

最终结果变成一个不可绕过的 merge gate。

---

## 一个非常具体的 MVP 场景

目标用户：

> 100-1000 人研发团队，已经开始用 Copilot / Cursor / Claude Code，但安全团队和技术管理者不放心。

MVP demo：

1. 开发者用 Claude Code 生成一个登录相关 PR
2. PR 模板声明 `AI-assisted: yes`
3. 系统检测到改了 `/auth` 目录
4. 自动提升风险等级为 High
5. 检查发现：
    * 单测通过
    * 覆盖率下降
    * 没有安全 reviewer
    * 没有 threat model
6. PR 被 block
7. 系统自动生成缺失清单
8. 开发者补测试 + 安全审批
9. 门禁变绿，允许 merge

这个 demo 很容易讲清楚价值。

---

## 商业化切入点

### 最可能买单的人

| 角色 | 他们关心什么 |
| --- | --- |
| CTO / VP Eng | AI 提效不能带来线上事故 |
| Security / AppSec | AI 生成代码绕过安全流程 |
| Platform Engineering | 统一研发门禁和开发者体验 |
| Compliance / Audit | 能否证明代码经过控制 |
| DevEx 团队 | AI coding adoption 之后如何标准化 |

### 最强卖点

不是“减少漏洞”，而是：

> **让企业敢大规模使用 AI coding。**

这个比单纯 code review 更高级。

---

## 你可以继续往下挖的几个关键词

后续搜索/验证可以围绕这些词：

```
AI Code Governance
AI-generated code security
AI Code Assurance
Agentic coding quality gates
SDLC governance for AI coding
Policy-as-code for CI/CD
Secure AI-assisted software development
AI coding compliance
AI-generated code audit trail
Software supply chain provenance
SLSA provenance
SBOM enforcement
GitHub Copilot governance
Enterprise AI coding guardrails
```

---

## 我的判断

这个 idea 值得推进，但要定位准确：

| 判断项 | 结论 |
| --- | --- |
| 需求强度 | 高 |
| 时间点 | 对 |
| 竞品压力 | 高 |
| 产品空白 | 有，但不在 AI review 本身 |
| 最佳切入 | PR / CI/CD 门禁 |
| 护城河 | 企业策略、证据链、上下文、审批流、历史风险数据 |
| 最小 MVP | GitHub App + policy engine + evidence report + required check |

一句话：

> **不要卖“AI 帮你看代码”。卖“AI 写的代码怎么被企业安全地允许上线”。**

* * *

**You:**

先不管企业，先看怎么检测AI写的代码的问题，比如已有项目测试不充足，比如已有项目有business logic，不是ai code review，它应该是rule engine做地基，而不是ai判断。

* * *

**ChatGPT:**

这次我会把重点收窄到“检测机制”。核心前提我先采纳：**AI 不做裁判，rule engine 才是地基；AI 最多辅助生成候选规则、解释风险、补测试建议。**

* * *

**ChatGPT:**

对，方向应该改成：

> **不是检测“是不是 AI 写的代码”，也不是让 AI review。**  
> 而是检测：**这次变更有没有破坏已有项目的隐含规则、业务约束、架构边界、测试可信度。**

AI 只是风险来源之一。真正的问题是：

```
已有项目测试不足 + 业务逻辑隐含在代码里 + AI/人类都可能改坏
```

所以产品地基应该是：

```
Fact Extractor → Rule Engine → Evidence Gate → Report
```

不是：

```
LLM reads diff → LLM says good/bad
```

---

# 1. 核心产品定义

可以叫：

> **Code Rule Engine for Existing Projects**  
> **Legacy-aware Code Change Gate**  
> **Business Logic Guardrail for Code Changes**

一句话：

> 从已有代码库中提取结构、依赖、测试、调用链、业务不变量，然后用规则引擎判断每次代码变更是否安全。

这里的关键词不是 AI code，而是：

```
existing project
insufficient tests
business logic
implicit rules
change safety
rule-based gate
```

---

# 2. 基本架构

## 2.1 输入

每次 PR / commit 进来，系统收集这些事实：

```YAML
change:
  changed_files:
    - src/order/OrderService.ts
    - src/payment/refund.ts
  changed_lines: 182
  deleted_tests: false
  added_tests: true

code_facts:
  touched_domains:
    - order
    - payment
  public_api_changed: true
  db_schema_changed: false
  auth_logic_changed: false
  money_logic_changed: true
  state_machine_changed: true
  external_api_changed: false

test_facts:
  changed_lines_covered: 41%
  related_tests_found: true
  mutation_score: 32%
  snapshot_changed: true
  contract_tests_affected: true

history_facts:
  files_hotspot_score: high
  recent_incidents_related: true
  last_modified_by_many_people: true

decision:
  risk_level: high
  block_reasons:
    - money logic changed but mutation score too low
    - public API changed but no contract test updated
    - order state transition changed without invariant test
```

---

## 2.2 输出

不是“代码写得不好”，而是具体门禁结果：

```
BLOCKED

原因：
1. 变更涉及 payment/refund 金额逻辑
2. 相关测试覆盖率只有 41%
3. mutation testing 显示测试无法杀死关键分支变异
4. refund 状态流从 PAID → REFUNDED 被改动，但没有状态机规则测试
5. public API response schema 发生变化，但没有 contract test 更新

需要补充：
- refund amount invariant test
- order state transition test
- API contract update
- reviewer approval from payment owner
```

这就是产品味道。

---

# 3. Rule Engine 才是核心

你说得对，地基应该是 rule engine。

LLM 可以参与：

| 环节 | LLM 是否适合 |
| --- | --- |
| 判断是否通过 | 不适合 |
| 生成候选规则 | 适合 |
| 解释失败原因 | 适合 |
| 从代码里总结业务逻辑 | 适合，但要人工确认 |
| 生成测试建议 | 适合 |
| 自动修复 | 可以，但必须再过规则 |
| 最终 block / allow | 不应该由 LLM 决定 |

真正的判断应该是：

```
确定性事实 + 显式规则 = 决策
```

比如 OPA 这类 policy engine 本来就是用“policy as code”来做统一策略决策，可以把结构化输入喂给规则引擎判断 allow / deny。OPA 官方定位就是通用策略引擎，可以用 Rego 这种声明式语言对结构化数据做策略评估，并可用于 CI/CD、Kubernetes、API Gateway 等场景。[Open Policy Agent+1](https://openpolicyagent.org/docs?utm_source=chatgpt.com)

---

# 4. 最关键的问题：已有项目测试不充足怎么办？

这是这个 idea 真正有价值的地方。

普通 code gate 默认：

```
测试通过 = 安全
```

但真实老项目是：

```
测试通过 ≠ 安全
```

因为测试可能根本没覆盖关键业务逻辑。

所以你要检测的不是“测试是否通过”，而是：

```
测试是否足以证明这次变更安全
```

---

## 4.1 Diff Coverage：改动行有没有被测试覆盖

最基础规则：

```YAML
rule: changed_code_must_be_covered
if:
  changed_lines_coverage < 80%
  touched_domain in ["payment", "auth", "order", "pricing"]
then:
  block: true
  reason: "Sensitive domain change requires changed-line coverage >= 80%"
```

传统总体覆盖率没意义。

```
项目总覆盖率 78%
```

不如：

```
这次改动的 182 行，只有 41% 被测试跑到
```

后者才是门禁信号。

---

## 4.2 Mutation Testing：测试是不是真的有断言能力

覆盖率只能说明代码被执行过，不能说明测试真的能发现错误。

所以要引入 **Mutation Testing**。

Mutation Testing 的意思是：工具故意把代码改坏，例如把 `>` 改成 `>=`、把 `&&` 改成 `||`，然后重新跑测试。如果测试没有失败，说明测试虽然跑到了代码，但没有真正验证行为。Stryker 官方文档也是这么定义 mutation testing：它会修改代码再运行单元测试，预期测试应该失败；如果不失败，可能说明测试覆盖不足。[Stryker Mutator](https://stryker-mutator.io/docs/?utm_source=chatgpt.com)

Java/JVM 生态里 PIT 是成熟方案，官方称它是 Java/JVM 的 mutation testing system，并且能和现代测试、构建工具集成。[PIT Mutation Testing](https://pitest.org/?utm_source=chatgpt.com)

门禁规则可以是：

```YAML
rule: mutation_score_for_sensitive_change
if:
  touched_domain in ["payment", "pricing", "auth"]
  mutation_score < 70
then:
  block: true
```

这比“AI review”强得多。

因为它不是问 AI：

```
你觉得 refund 逻辑有没有问题？
```

而是问：

```
如果 refund 逻辑被轻微改坏，现有测试能不能抓住？
```

---

## 4.3 Characterization Test：先锁住老系统当前行为

老项目最大问题是：业务逻辑没人讲得清楚。

这时候不能一开始就要求“正确测试”，而是先做：

> **Characterization Test / Golden Master Test**

意思是：先记录当前系统行为，把它当成基线。以后变更如果改变行为，必须明确说明。

Approval Tests 这类工具就是这种思路：它通过把代码输出和预期结果做比较来验证变化，也常被称为 Golden Master / Snapshot Testing，适合复杂对象、列表、CSV、图片等不容易用普通 assert 写清楚的结果。[GitHub+1](https://github.com/approvals/ApprovalTests.Documentation/blob/main/explanations/what_are_approvals.md?utm_source=chatgpt.com)

门禁规则：

```YAML
rule: behavior_snapshot_changed
if:
  snapshot_changed == true
  no_explicit_approval == true
then:
  block: true
  reason: "Existing behavior changed without approval"
```

这个特别适合已有项目：

```
我不知道老逻辑是不是正确，
但我至少知道这次改动有没有改变老逻辑。
```

---

# 5. 业务逻辑应该怎么检测？

业务逻辑不能靠 AI 临场猜。

要把业务逻辑变成几类可执行规则。

---

## 5.1 状态机规则

很多业务 bug 都是状态流错了。

例如订单：

```
CREATED → PAID → SHIPPED → COMPLETED
CREATED → CANCELLED
PAID → REFUNDED
```

规则：

```YAML
rule: invalid_order_state_transition
domain: order
allowed_transitions:
  CREATED:
    - PAID
    - CANCELLED
  PAID:
    - SHIPPED
    - REFUNDED
  SHIPPED:
    - COMPLETED
  COMPLETED: []
  CANCELLED: []
  REFUNDED: []
```

检测点：

```
如果代码新增了 PAID → CANCELLED
必须 block 或 require approval
```

这类规则可以从哪里来？

| 来源 | 方式 |
| --- | --- |
| 代码枚举 | 扫 enum / constants |
| switch / if 分支 | AST 分析 |
| 数据库状态字段 | schema 分析 |
| 历史生产数据 | 统计真实状态迁移 |
| 测试 fixture | 提取已有 case |
| 文档 / ticket | LLM 可以辅助提取候选，但要人工确认 |

---

## 5.2 金额规则

AI 特别容易写错 money logic。

规则类型：

```
金额不能为负
退款金额不能大于支付金额
折扣后金额不能小于 0
税费计算必须使用 Decimal，不能用 float
货币转换必须带 currency
金额入库必须是 minor unit，例如 cents/fen
```

门禁规则：

```YAML
rule: money_logic_requires_decimal
if:
  touched_domain: payment
  code_contains:
    - "number"
    - "float"
    - "parseFloat"
  no_decimal_library_used: true
then:
  block: true
```

这里可以结合 Semgrep 一类工具做规则扫描。Semgrep 支持用规则查找代码模式，也支持 taint analysis，也就是跟踪不可信输入从 source 流向 sink，中间是否经过 sanitizer。[Semgrep+1](https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/overview?utm_source=chatgpt.com)

---

## 5.3 权限规则

权限规则非常适合 rule engine。

例如：

```YAML
rule: no_direct_admin_grant
if:
  changed_code_contains:
    - "role = ADMIN"
    - "setRole('admin')"
  not_in_allowed_files:
    - "AdminRoleService"
then:
  block: true
```

更高级一点：

```YAML
rule: user_input_cannot_control_role
taint:
  source:
    - req.body.role
    - request.getParameter("role")
  sink:
    - user.setRole(...)
    - updateUserRole(...)
  sanitizer:
    - authorizeAdmin(...)
then:
  block: true
```

这不是 AI review，是数据流规则。

---

## 5.4 幂等性规则

很多业务代码必须幂等。

解释一下：**幂等性**就是同一个操作执行多次，结果应该和执行一次一样。比如支付回调可能重复来两次，但订单不能重复入账。

规则：

```YAML
rule: payment_callback_must_be_idempotent
if:
  touched_files:
    - "**/payment/**"
    - "**/callback/**"
  code_missing:
    - idempotencyKey
    - unique constraint
    - processed event check
then:
  block: true
```

---

## 5.5 时间规则

AI 常写错时间逻辑：

```
本地时间 / UTC 混用
时区丢失
用 now() 导致测试不可控
有效期边界错误
```

规则：

```YAML
rule: no_direct_now_in_business_logic
if:
  touched_domain in ["subscription", "billing", "order"]
  code_contains:
    - "new Date()"
    - "LocalDateTime.now()"
    - "Date.now()"
  not_in_allowed_wrapper: true
then:
  warn_or_block: true
```

要求统一使用：

```
Clock / TimeProvider / BusinessCalendar
```

---

## 5.6 API Contract 规则

已有项目测试不足时，API contract 很重要。

Pact 是 code-first 的 contract testing 工具，用来测试 HTTP 和消息集成；contract test 验证应用之间的消息是否符合共享契约，避免只靠昂贵且脆弱的端到端集成测试。[Pact Docs](https://docs.pact.io/?utm_source=chatgpt.com)

规则：

```YAML
rule: public_api_change_requires_contract_test
if:
  public_api_schema_changed: true
  contract_test_updated: false
then:
  block: true
```

检测：

```
OpenAPI diff
GraphQL schema diff
Pact contract diff
DTO response shape diff
```

---

# 6. 架构规则也应该纳入

很多 AI 代码的问题不是语法错，而是破坏架构。

比如：

```
Controller 直接访问 Repository
Frontend 直接拼接后端内部 API
Domain 层依赖 Infrastructure 层
业务代码直接调用外部 SDK
绕过统一 auth middleware
绕过 BFF
```

Java 里 ArchUnit 就是这种思路。它可以用普通单元测试框架检查 Java 架构规则，包括包、类、层之间的依赖、循环依赖等；它通过分析 Java bytecode 来做这些检查。[ArchUnit+1](https://www.archunit.org/?utm_source=chatgpt.com)

规则示例：

```Java
noClasses()
  .that().resideInAPackage("..domain..")
  .should().dependOnClassesThat()
  .resideInAPackage("..infrastructure..");
```

产品化以后，不应该局限 Java。可以抽象成：

```YAML
rule: domain_must_not_depend_on_infra
language: any
layers:
  domain:
    cannot_depend_on:
      - infrastructure
      - web
      - database
```

---

# 7. 检测系统应该分成 6 类规则

## 7.1 Structural Rules：结构规则

看代码结构。

```
目录边界
模块依赖
层级依赖
循环依赖
禁止 import
禁止跨 bounded context 调用
```

适合工具：

```
ArchUnit
ESLint custom rule
dependency-cruiser
madge
jQAssistant
custom AST analyzer
```

---

## 7.2 Semantic Pattern Rules：语义模式规则

看危险代码模式。

```
直接 SQL 拼接
直接使用 Date.now()
金额使用 float
catch 后吞异常
日志打印 PII
绕过统一权限检查
```

适合：

```
Semgrep
CodeQL
custom AST rules
```

CodeQL / Semgrep 这类不是“AI review”，而是明确规则。

---

## 7.3 Data Flow Rules：数据流规则

看数据从哪里来，流到哪里去。

```
用户输入 → SQL
用户输入 → HTML
用户输入 → role update
用户输入 → file path
用户输入 → command execution
```

Semgrep taint mode 就是这类能力：定义 source、sink、sanitizer，追踪数据流。[Semgrep+1](https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/overview?utm_source=chatgpt.com)

---

## 7.4 Behavior Rules：行为规则

看行为有没有变化。

```
snapshot test 是否变化
golden master 是否变化
关键 API response 是否变化
核心计算结果是否变化
```

适合老项目。

因为很多老项目没有 spec，只有当前行为。

---

## 7.5 Test Adequacy Rules：测试充分性规则

看测试够不够证明安全。

```
changed line coverage
branch coverage
mutation score
related test existence
contract test updated
snapshot updated
property test exists
```

这是这个产品最重要的差异化之一。

---

## 7.6 Domain Invariant Rules：领域不变量规则

看业务底线有没有被破坏。

```
余额不能为负
库存不能超卖
退款不能大于支付金额
订单完成后不能取消
普通用户不能授予自己 admin
订阅过期后不能访问 pro 功能
同一个事件不能重复处理
```

这部分最难，但也最值钱。

---

# 8. Rule Engine 的数据模型

可以把所有扫描结果统一成 facts。

```JSON
{
  "repo": "checkout-service",
  "change": {
    "id": "PR-481",
    "changed_files": [
      "src/payment/refund.ts",
      "src/order/state.ts"
    ],
    "changed_lines": 182
  },
  "facts": {
    "domains": ["payment", "order"],
    "sensitive_domains": ["payment"],
    "public_api_changed": true,
    "db_schema_changed": false,
    "money_logic_changed": true,
    "state_transition_changed": true,
    "changed_line_coverage": 0.41,
    "mutation_score": 0.32,
    "contract_test_updated": false,
    "snapshot_changed": true,
    "owner_approved": false
  }
}
```

然后规则：

```rego
package codegate

default allow := true

deny[msg] {
  input.facts.money_logic_changed
  input.facts.changed_line_coverage < 0.8
  msg := "Money logic changed but changed-line coverage is below 80%"
}

deny[msg] {
  input.facts.state_transition_changed
  not input.facts.domain_invariant_test_updated
  msg := "State transition changed without invariant test update"
}

deny[msg] {
  input.facts.public_api_changed
  not input.facts.contract_test_updated
  msg := "Public API changed without contract test update"
}

deny[msg] {
  input.facts.mutation_score < 0.6
  input.facts.sensitive_domains[_] == "payment"
  msg := "Sensitive payment change requires mutation score >= 60%"
}
```

这个才是你说的“rule engine 做地基”。

---

# 9. AI 在这个产品里的正确位置

AI 不做 judge，但可以做 4 件事。

## 9.1 从代码中提取候选业务规则

例如 AI 读到：

```TypeScript
if (order.status !== "PAID") {
  throw new Error("Only paid orders can be refunded");
}
```

输出候选规则：

```YAML
candidate_rule:
  type: state_transition
  domain: order
  invariant: "Only PAID orders can be refunded"
  confidence: medium
  evidence:
    - src/refund/refundService.ts:42
```

但是状态是：

```
candidate → human approved → active rule
```

不能直接进入门禁。

---

## 9.2 给规则生成测试模板

比如规则：

```
退款金额不能大于支付金额
```

AI 可以生成：

```TypeScript
it("should reject refund amount greater than paid amount", () => {
  const order = paidOrder({ paidAmount: 100 });
  expect(() => refund(order, 101)).toThrow();
});
```

但测试是否有效，还是靠 mutation testing / coverage / CI。

---

## 9.3 解释规则失败原因

规则引擎输出可能很硬：

```
RULE_PAYMENT_003 failed
```

AI 可以解释成：

```
你修改了退款金额计算，但没有新增能证明“退款金额不超过原支付金额”的测试。当前测试虽然执行到了 refund.ts，但 mutation testing 显示把 <= 改成 < 后测试仍然通过，所以测试断言不足。
```

这个很有用。

---

## 9.4 辅助生成项目初始规则库

扫描项目后，AI 输出：

```
我发现这个项目可能有这些业务规则：
1. 订单只有 PAID 状态才能退款
2. REFUNDED 状态不能再次 refund
3. discount 不能和 coupon 同时叠加
4. admin role 只能由 AdminUserService 修改
```

然后让人确认：

```
accept / reject / edit
```

这比 AI review 有价值。

---

# 10. MVP 应该怎么做

不要先做全语言、全场景。MVP 只切一个栈。

你可以选：

```
TypeScript / JavaScript monorepo
```

或者：

```
Java Spring Boot
```

我更建议先选 **TypeScript / Node / React + API 项目**，因为你自己更容易快速做出 demo。

---

## MVP 版本 1：PR Change Risk Gate

只做 GitHub PR。

### 输入

```
PR diff
changed files
test coverage report
unit test result
package diff
OpenAPI diff
Semgrep result
custom rule result
```

### 输出

```
risk score
failed rules
missing evidence
suggested tests
```

### 第一批规则

```YAML
rules:
  - changed lines coverage must be >= 70%
  - sensitive path change requires test update
  - public API change requires contract/snapshot update
  - money logic cannot use floating point
  - auth change requires owner approval
  - migration change requires rollback
  - deleted tests block merge
  - large PR requires split
```

---

# 11. MVP 版本 2：Business Rule Mining

扫描已有项目，生成候选规则。

### 从哪里挖规则？

| 来源 | 可提取信息 |
| --- | --- |
| enum | 状态机 |
| constants | 业务枚举 |
| validation schema | 字段约束 |
| if / switch | 业务分支 |
| test names | 已知业务场景 |
| fixtures | 典型输入输出 |
| database schema | 唯一约束、外键、nullable |
| API schema | public contract |
| error messages | 业务限制 |
| logs / metrics name | 关键流程 |
| comments | 隐性规则 |
| production data | 实际状态迁移 |

### 输出

```YAML
candidate_rules:
  - id: ORDER_REFUND_001
    type: invariant
    rule: "Only PAID orders can be refunded"
    evidence:
      - src/refund/refundService.ts:42
      - test/refund.test.ts:18
    status: pending_review
```

这个是高价值点。

---

# 12. MVP 版本 3：Test Adequacy Checker

专门解决“测试不充足”。

### 它不说

```
测试通过了
```

### 它说

```
测试不够证明这次变更安全
```

检测项：

```
changed-line coverage
branch coverage
mutation score
related test existence
contract test update
snapshot/golden master update
property test existence
sensitive domain minimum threshold
```

这个产品点很锋利。

因为 AI 代码最大的问题不是编译不过，而是：

```
看起来对，测试也绿，但业务错了。
```

---

# 13. 可以形成的产品闭环

## 第一步：Baseline

扫描已有项目，建立当前基线：

```
当前覆盖率低没关系
当前架构混乱没关系
当前旧漏洞没关系
```

但以后：

```
不能让风险变更更差
```

这个叫 **ratchet rule**。

解释一下：ratchet 是棘轮机制，只能往一个方向走。用于工程质量时，就是“不要求一次性变好，但不允许变差”。

规则：

```YAML
rule: no_quality_regression
if:
  new_mutation_score < baseline_mutation_score
then:
  block: true
```

这样老项目也能用。

否则一上来全量扫描，满屏红灯，没人会用。

---

## 第二步：Diff Gate

只拦这次变更造成的新风险。

```
legacy issue 不拦
new issue 拦
risk increase 拦
evidence missing 拦
```

---

## 第三步：Rule Discovery

系统持续发现新规则：

```
这个 if 分支看起来是业务规则
这个 enum 看起来是状态机
这个测试名看起来是业务约束
这个 production incident 对应某类规则
```

---

## 第四步：Human Approval

人确认后进入规则库：

```
candidate rule → approved rule → enforced rule
```

---

## 第五步：Continuous Hardening

线上事故、bugfix、hotfix 回流成规则。

```
每次线上 bug 修复后：
- 是否需要新增 invariant？
- 是否需要新增 mutation test？
- 是否需要新增 static rule？
- 是否需要新增 architecture rule？
```

这个闭环很值钱。

---

# 14. 一个具体例子

假设 AI 改了退款逻辑：

```Diff
- if (refundAmount > order.paidAmount) throw new Error(...)
+ if (refundAmount >= order.paidAmount) throw new Error(...)
```

普通测试可能仍然通过。

AI review 可能说：

```
Looks good.
```

你的系统应该做：

## 1. Fact extraction

```YAML
changed_domain: payment
money_logic_changed: true
comparison_operator_changed: true
related_tests_found: true
changed_line_coverage: 91%
mutation_score: 35%
```

## 2. Rule evaluation

```YAML
failed_rules:
  - PAYMENT_MUTATION_SCORE_MINIMUM
  - MONEY_COMPARISON_CHANGE_REQUIRES_BOUNDARY_TEST
```

## 3. Output

```
BLOCKED

原因：
这次变更修改了 payment/refund 的金额比较边界。
虽然 changed-line coverage 是 91%，但 mutation testing 显示边界条件测试不足。

必须补充：
- refundAmount == paidAmount
- refundAmount > paidAmount
- refundAmount < paidAmount
```

这就非常实用。

---

# 15. 另一个例子：状态机

AI 改了：

```Diff
- if (order.status !== "PAID") throw new Error(...)
+ if (["PAID", "SHIPPED"].includes(order.status)) ...
```

系统识别：

```YAML
state_transition_changed: true
domain: order
old_allowed_states:
  refund:
    - PAID
new_allowed_states:
  refund:
    - PAID
    - SHIPPED
```

规则：

```YAML
rule: state_machine_change_requires_invariant_update
if:
  state_transition_changed: true
  invariant_test_updated: false
then:
  block: true
```

输出：

```
BLOCKED

refund 允许状态从 PAID 扩展到 PAID, SHIPPED。
这是业务状态机变化，不允许只改实现代码。

需要：
1. 更新 order state machine spec
2. 增加 SHIPPED refund 测试
3. owner approval
```

---

# 16. 技术实现路线

## 16.1 Fact Extractor

按语言做插件。

TypeScript：

```
ts-morph
TypeScript Compiler API
ESLint custom rules
dependency-cruiser
Istanbul coverage
StrykerJS
OpenAPI diff
Semgrep
```

Java：

```
JavaParser
ArchUnit
PIT
JaCoCo
OpenRewrite
Semgrep
Maven/Gradle plugin
```

OpenRewrite 值得关注，它的核心概念是 recipes，官方 recipe catalog 覆盖 Java、JavaScript、YAML、GitHub Actions、Gradle、Maven、Docker 等多类代码和配置，也包括静态分析和自动修复相关 recipe。[OpenRewrite Docs+1](https://docs.openrewrite.org/recipes?utm_source=chatgpt.com)

---

## 16.2 Rule Engine

可以有两层：

### 简单规则 DSL

给普通团队配置：

```YAML
rules:
  - id: sensitive_path_requires_tests
    when:
      changed_path:
        - "src/payment/**"
        - "src/auth/**"
    require:
      changed_line_coverage: ">=80"
      test_files_changed: true
```

### 高级策略引擎

给平台团队用：

```
OPA / Rego
```

OPA 适合做这个底层，因为它本来就是把策略决策从应用中抽出来，对输入数据做统一 policy evaluation。[Open Policy Agent+1](https://openpolicyagent.org/docs?utm_source=chatgpt.com)

---

## 16.3 Evidence Store

每次 PR 存：

```
facts
rules evaluated
pass/fail
coverage
mutation score
snapshots
contract diffs
approvals
baseline comparison
```

这会变成后续产品护城河。

---

# 17. 这个方向的产品边界

## 不做

```
不做 IDE
不做代码生成
不做聊天式 code review
不做“识别 AI 代码百分比”
不做另一个 Sonar
不做全量 legacy 扫描吓人报表
```

## 做

```
只判断这次变更是否让项目更危险
只要求补齐跟本次变更相关的证据
只把人确认过的规则作为硬门禁
只让 AI 辅助生成候选规则和解释
```

---

# 18. 最小可卖版本

我会把第一版产品收敛成：

> **PR Test Adequacy + Business Rule Gate**

具体功能：

1. GitHub App
2. 读取 PR diff
3. 判断 touched domain
4. 跑 changed-line coverage
5. 对 changed files 跑 mutation testing
6. 检查 sensitive paths
7. 检查 API schema diff
8. 检查是否更新相关测试
9. 用 rule engine 输出 pass/block
10. AI 只负责解释失败原因和建议补什么测试

第一版规则库：

```YAML
sensitive_domains:
  payment:
    paths:
      - "src/payment/**"
    required:
      changed_line_coverage: 0.8
      mutation_score: 0.7
      tests_changed: true

  auth:
    paths:
      - "src/auth/**"
    required:
      changed_line_coverage: 0.85
      mutation_score: 0.75
      owner_approval: true

  order:
    paths:
      - "src/order/**"
    required:
      state_machine_change_requires_test: true
      snapshot_change_requires_approval: true
```

---

# 19. 真正的差异化

市面上大多数工具看的是：

```
代码有没有漏洞
代码有没有坏味道
代码有没有格式问题
```

你这个方向应该看：

```
这次变更有没有足够证据证明它没有破坏已有业务行为
```

这很不一样。

尤其是这句话可以作为核心卖点：

> **Green tests are not enough. We check whether the tests are capable of catching the kinds of mistakes this change could introduce.**

中文：

> **测试通过不代表安全。我们检查的是：这些测试有没有能力抓住这次变更可能引入的错误。**

这个定位比 “AI code review” 强很多。

* * *

**You:**

基于OPA是最好选择吗？收敛一下产品形态。深化、发散一下检测、门禁、规则的技术方向，主要考虑和目标是，帮助个人，小团队，企业，在日益泛滥的AI coding的基础上，作为最后一道关卡，让AI生成的代码尽可能缩小影响范围，提效的同时可控。主要是帮助brownfield apps.

* * *

**ChatGPT:**

## 先回答：OPA 是最好选择吗？

**不是唯一最好，但可以作为 enterprise 版的底层策略引擎。**

更准确的判断：

| 层级 | 推荐 |
| --- | --- |
| 个人 / 小团队 | 不直接暴露 OPA / Rego，先用 YAML / JSON 规则 |
| 成熟团队 | YAML DSL + 内置 rule packs |
| 企业 | DSL 编译到 OPA / Rego，或者允许高级用户写 Rego |
| 插件生态 | Semgrep / CodeQL / ArchUnit / Stryker / PIT / Pact 等作为 fact providers |
| 最终门禁 | GitHub required status check / GitLab external status check / CI job |

**OPA 的优势**是：它专门做“把结构化输入交给策略做 allow / deny 决策”，Rego 适合处理复杂层级数据，例如 PR diff、测试结果、扫描结果、审批状态、路径、风险等级等。OPA 官方也明确定位为通用 policy engine，用 Rego 对结构化数据做策略评估。[Open Policy Agent+1](https://openpolicyagent.org/docs?utm_source=chatgpt.com)

但它的问题也明显：

1. **Rego 学习成本高**  
    个人开发者和小团队不愿意为一个代码门禁学 Rego。
2. **OPA 只负责决策，不负责事实提取**  
    你最难的不是 `deny[msg]`，而是把 brownfield app 里的业务逻辑、测试弱点、架构边界提取成可信 facts。
3. **OPA 对产品体验不够友好**  
    你的产品卖点不是“我支持 Rego”，而是“我能告诉你这次 AI 改动哪里危险、缺什么证据、如何缩小影响范围”。

所以推荐架构是：

```
内置规则 DSL / UI
        ↓
Normalized Facts
        ↓
Rule Engine
        ↓
OPA / CEL / Custom evaluator
        ↓
GitHub / GitLab / CI Gate
```

OPA 可以在底层，但不要作为产品心智的第一层。

---

# 产品形态收敛

我会把它收敛成：

> **AI Code Change Firewall for Brownfield Apps**  
> 面向已有项目的 AI 代码变更防火墙。

中文可以叫：

> **AI 代码变更防火墙**  
> **Brownfield 代码门禁**  
> **AI Coding 最后一道关卡**

核心不是“检查 AI 写的代码好不好”，而是：

> **任何 AI / 人类生成的代码，只要要进入已有项目，就必须证明这次变更没有扩大不可控影响。**

这里的重点是 **brownfield apps**，也就是已经运行多年、有历史包袱、测试不足、业务逻辑隐含在代码里的老项目。

---

# 一句话定位

> 测试绿灯不代表安全。这个产品检查的是：**这次变更是否有足够证据证明它没有破坏已有业务行为。**

更锋利一点：

> **AI coding 提效负责加速写代码，我们负责限制它的破坏半径。**

---

# 不要做成什么

不要做成：

```
AI code review bot
```

也不要做成：

```
another SonarQube
```

也不要做成：

```
识别这段代码是不是 AI 写的
```

正确形态是：

```
PR / MR / CI 里的 change gate
```

它只关心四件事：

```
1. 这次改了什么？
2. 改动影响哪些业务区域？
3. 当前测试和规则能否证明它安全？
4. 如果不能，应该 block / warn / require approval / require smaller rollout？
```

---

# 产品应该分三档

## 1. Personal：本地 CLI

适合个人开发者、独立开发者。

```Bash
codegate check
codegate baseline
codegate explain
```

能力：

```
- 扫当前 diff
- 检查改动范围
- 检查敏感路径
- 检查是否改了测试
- 检查 changed-line coverage
- 给出风险报告
- 不强制 block，只提醒
```

目标：

> 个人用 Cursor / Claude Code / Copilot 写代码后，提交前先跑一次，避免 AI 一把梭改坏老项目。

---

## 2. Team：GitHub Action / GitLab CI

适合小团队。

```YAML
- uses: codegate/check@v1
  with:
    profile: brownfield-strict
    config: codegate.yml
```

能力：

```
- PR required check
- 只拦新增风险，不清算历史旧账
- 根据路径和风险等级要求测试 / owner approval
- 输出 missing evidence
```

GitHub rulesets 支持 required status checks，也就是所有必需 CI 检查通过后才允许合并；外部服务也可以通过 commit status API 回写状态。[GitHub Docs+1](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets?utm_source=chatgpt.com) GitLab 也有 merge request approval rules 和 external status checks，可以用外部系统影响 MR 是否可合并。[docs.gitlab.com+1](https://docs.gitlab.com/user/project/merge_requests/approvals/rules/?utm_source=chatgpt.com)

---

## 3. Enterprise：Policy Server + Evidence Store

适合企业。

能力：

```
- 中央规则库
- repo profile 管理
- 规则版本管理
- 豁免审批
- 审计记录
- 多工具结果聚合
- GitHub / GitLab / Jenkins / Azure DevOps 集成
- OPA / Rego 高级策略支持
```

企业版卖的不是“扫描”，而是：

> **AI coding adoption governance**  
> 企业如何放心扩大 AI coding 使用范围。

---

# 核心系统架构

```
                PR / MR / Commit / Local Diff
                         ↓
                 Change Fact Extractor
                         ↓
       ┌──────────────────────────────────┐
       │ Normalized Fact Model             │
       │ changed files / domains / tests   │
       │ API diff / coverage / mutation    │
       │ arch violations / business rules  │
       └──────────────────────────────────┘
                         ↓
                  Rule Evaluation
                         ↓
      allow / warn / block / require evidence
                         ↓
        GitHub Check / GitLab Status / CI Output
                         ↓
                 Evidence Report
```

重点：**LLM 不参与最终判定。**

LLM 的位置应该是：

```
- 解释失败原因
- 生成测试建议
- 从代码里挖候选规则
- 辅助把自然语言规范转成规则草稿
```

但最终裁决必须是：

```
facts + rules = decision
```

---

# OPA、CEL、Cedar 怎么选？

## OPA

适合：

```
- 企业级复杂策略
- 多条件组合
- 审批 / 路径 / 风险 / 测试 / 扫描结果混合判断
- 中央 policy server
```

例子：

```rego
deny[msg] {
  input.change.touched_domains[_] == "payment"
  input.tests.changed_line_coverage < 0.8
  msg := "Payment changes require changed-line coverage >= 80%"
}
```

适合 enterprise。

---

## CEL

**CEL，Common Expression Language，通用表达式语言**，适合嵌入到应用配置里做轻量判断。它的定位是 fast、portable、safe，适合 validation constraints、authorization rules 等场景。[Common Expression Language](https://cel.dev/?utm_source=chatgpt.com) Kubernetes 也用 CEL 在 API server 里声明 validation rules、policy rules 和 constraints。[Kubernetes](https://kubernetes.io/docs/reference/using-api/cel/?utm_source=chatgpt.com)

适合：

```
- 简单条件表达式
- YAML 配置里的 when 条件
- 给个人 / 小团队使用
```

例子：

```YAML
rules:
  - id: payment-coverage
    when: "'payment' in change.touched_domains"
    require: "tests.changed_line_coverage >= 0.8"
```

CEL 比 Rego 更适合产品第一层。

---

## Cedar

**Cedar** 是 AWS 开源的授权策略语言，主要用于表达“谁可以对什么资源做什么动作”的 authorization decision。官方文档也把它定位在应用权限和访问控制策略上。[Cedar Policy Language Reference Guide+1](https://docs.cedarpolicy.com/?utm_source=chatgpt.com)

它适合：

```
user can approve PR?
team can override policy?
service can deploy artifact?
```

但不太适合：

```
这次 refund 逻辑改动是否需要 mutation score >= 70%
```

所以 Cedar 可以用在权限系统，不适合作为代码质量规则核心。

---

# 我的选择

```
MVP:
  Custom YAML DSL + CEL expression evaluator

Team/Enterprise:
  DSL → internal evaluator

Enterprise advanced:
  optional OPA/Rego backend

AuthZ:
  可考虑 Cedar，但不是第一优先级
```

不要一开始就绑定 OPA。否则产品会太像 infra tool，不像一个能被个人、小团队快速用起来的 code gate。

---

# 规则系统应该长什么样

## 第一层：简单声明式规则

```YAML
profiles:
  brownfield-default:
    baseline_mode: true
    only_block_new_risk: true

domains:
  payment:
    paths:
      - "src/payment/**"
      - "src/billing/**"
    required:
      changed_line_coverage: 0.8
      mutation_score: 0.6
      tests_changed: true

  auth:
    paths:
      - "src/auth/**"
      - "src/security/**"
    required:
      changed_line_coverage: 0.85
      owner_approval: true
      no_direct_role_assignment: true

rules:
  - id: deleted-tests-block
    when:
      deleted_test_files: true
    decision: block
    reason: "Deleting tests is not allowed without explicit override"

  - id: public-api-contract
    when:
      public_api_changed: true
    require:
      contract_test_updated: true
    decision: block
```

这一层给 80% 用户。

---

## 第二层：规则包

做成内置 packs：

```
codegate init --profile brownfield
```

生成：

```
- brownfield-core
- ai-coding-risk
- test-adequacy
- api-contract
- payment-safety
- auth-safety
- frontend-risk
- db-migration
- terraform-iac
```

用户不用从零写规则。

---

## 第三层：高级策略

企业用户可以写：

```rego
package codegate

deny[msg] {
  input.change.ai_generated == true
  input.change.lines_added > 800
  input.tests.changed_line_coverage < 0.7
  msg := "Large AI-generated PR requires stronger test evidence"
}
```

但这不是 MVP 起点。

---

# 检测方向深化

## 1. Change Scope Detection：先确定影响范围

AI coding 最大风险之一是：

```
为了实现一个小需求，改了一大片代码。
```

所以第一类检测不是安全漏洞，而是 **blast radius**，即影响半径。

规则：

```YAML
- id: ai-large-diff
  when:
    lines_changed: ">800"
  decision: warn
  require:
    split_pr: true
```

检测项：

```
- 改了多少文件
- 改了多少行
- 是否跨多个 domain
- 是否同时改 frontend + backend + db + infra
- 是否修改 generated files
- 是否删除大量代码
- 是否重命名核心类
- 是否移动目录结构
- 是否 touching hot files
```

**hot files** 指历史上频繁修改、经常出 bug、多人共同维护的文件。brownfield 项目里，hot files 往往比普通文件更危险。

门禁策略：

```
小 PR：允许自动通过更多检查
大 PR：要求拆分 / owner approval / 更高测试证据
跨 domain PR：默认高风险
```

---

## 2. Sensitive Path Rules：敏感路径门禁

最简单但最有效。

```YAML
sensitive_paths:
  - path: "src/auth/**"
    risk: high
    require:
      owner_approval: true
      changed_line_coverage: 0.85

  - path: "src/payment/**"
    risk: high
    require:
      mutation_score: 0.7
      tests_changed: true

  - path: "migrations/**"
    risk: medium
    require:
      rollback_plan: true
      backward_compatible: true

  - path: "infra/terraform/**"
    risk: high
    require:
      iac_policy_check: true
```

这对个人、小团队也有用，因为配置成本低。

---

## 3. Architecture Boundary Detection：架构边界检测

AI 很容易写出“能跑但破坏架构”的代码。

比如：

```
- controller 直接访问 database
- domain 层 import infra 层
- frontend 直接调用内部 service endpoint
- BFF 被绕过
- shared module 反向依赖 app module
```

Java 可以用 ArchUnit。ArchUnit 官方定位就是用普通 Java 单元测试框架检查架构规则，例如 package/class/layer/slice 依赖和循环依赖，并通过分析 Java bytecode 来完成。[ArchUnit+1](https://www.archunit.org/?utm_source=chatgpt.com)

TypeScript 可以用：

```
- dependency-cruiser
- ESLint import rules
- ts-morph / TypeScript Compiler API
- custom AST analyzer
```

规则示例：

```YAML
- id: no-domain-to-infra
  type: dependency-boundary
  from: "src/domain/**"
  cannot_import:
    - "src/infra/**"
    - "src/db/**"
```

这类规则很适合产品化，因为它是稳定规则，不依赖 AI 判断。

---

## 4. Semantic Pattern Rules：业务危险模式检测

这里用 Semgrep / CodeQL / AST rules。

Semgrep 支持自定义规则和 taint mode。taint analysis 的意思是追踪数据从 source 到 sink 的流动，例如用户输入是否流入 SQL、HTML、命令执行、角色修改等危险位置。[Semgrep](https://semgrep.dev/docs/writing-rules/data-flow/taint-mode/overview?utm_source=chatgpt.com) CodeQL 也支持写 custom queries，用于扩展内置安全分析，检查代码库特定漏洞、编码标准和代码模式。[GitHub Docs+1](https://docs.github.com/en/code-security/concepts/code-scanning/codeql/custom-codeql-queries?utm_source=chatgpt.com)

适合检测：

```
- SQL 拼接
- command injection
- path traversal
- 用户输入控制 role
- 日志打印 PII
- 金额用 float / number
- catch 后吞异常
- 直接调用 Date.now()
- 绕过统一 auth middleware
- 直接访问 process.env
- hardcoded secret
```

产品形态：

```YAML
rules:
  - id: no-float-money
    type: semgrep
    domain: payment
    pattern: |
      $AMOUNT: number
    message: "Money amount should not use floating point number"

  - id: user-input-to-role
    type: taint
    source:
      - "req.body.role"
    sink:
      - "user.setRole(...)"
    require_sanitizer:
      - "authorizeAdmin(...)"
```

---

## 5. Test Adequacy：测试充分性检测

这是你的核心差异化。

普通 CI 只问：

```
测试是否通过？
```

你的产品要问：

```
这些测试有没有能力抓住这次变更可能引入的错误？
```

检测项：

```
- changed-line coverage
- branch coverage
- touched function coverage
- related tests changed
- mutation score
- boundary case coverage
- contract tests updated
- snapshot / golden master changed
```

**Mutation testing** 是关键能力。它会故意修改代码再运行单元测试，预期测试应该失败；如果测试没失败，说明测试可能没有真正验证行为。Stryker 官方就是这样定义 mutation testing 的。[Stryker Mutator](https://stryker-mutator.io/docs/?utm_source=chatgpt.com) Java/JVM 生态可以用 PIT，它官方定位为 Java/JVM 的 mutation testing system，并且能和现代测试、构建工具集成。[PIT Mutation Testing](https://pitest.org/?utm_source=chatgpt.com)

门禁例子：

```YAML
- id: sensitive-change-mutation
  when:
    touched_domain_in:
      - payment
      - auth
      - pricing
  require:
    changed_line_coverage: ">=0.8"
    mutation_score: ">=0.65"
```

注意：mutation testing 很慢，所以不能全量跑。

MVP 里应该做：

```
- 只对 changed files 跑
- 只对 touched functions 跑
- 只对 high-risk domain 跑
- 只在 PR ready-for-review 后跑
- 本地提醒，CI 里选择性 block
```

---

## 6. Business Invariant Detection：业务不变量检测

**业务不变量**就是不管代码怎么改，都必须始终成立的业务底线。

例如：

```
- 余额不能为负
- 退款金额不能超过支付金额
- 已完成订单不能取消
- 普通用户不能给自己 admin 权限
- 订阅过期后不能访问 pro 功能
- 同一支付回调不能重复入账
- 库存不能扣成负数
```

这部分最难，但也最有价值。

产品上不要幻想一开始自动理解全部业务，而是走这个流程：

```
候选规则发现 → 人确认 → 进入规则库 → 门禁执行
```

AI 可以参与“候选规则发现”，但不能直接变成强规则。

例子：

```YAML
candidate_rule:
  id: refund-paid-only
  type: invariant
  source:
    - src/refund/refundService.ts:42
    - tests/refund.test.ts:18
  rule: "Only PAID orders can be refunded"
  status: pending_review
```

确认后变成：

```YAML
rules:
  - id: refund-paid-only
    type: invariant
    domain: payment
    check:
      state_transition:
        action: refund
        allowed_from:
          - PAID
```

---

## 7. State Machine Detection：状态机检测

很多 brownfield 业务逻辑本质是状态机。

例如订单：

```
CREATED → PAID → SHIPPED → COMPLETED
CREATED → CANCELLED
PAID → REFUNDED
```

AI 最容易犯的错误是：

```
为了让某个 case 通过，偷偷新增一条状态迁移。
```

检测方式：

```
- 从 enum / constants 提取状态
- 从 if / switch 提取状态判断
- 从历史测试提取状态流
- 从数据库历史数据观察真实迁移
- 从代码变更 diff 检测 allowed states 是否变化
```

门禁：

```YAML
- id: state-machine-change-requires-test
  when:
    state_machine_changed: true
  require:
    invariant_test_updated: true
    owner_approval: true
```

---

## 8. API Contract Detection：接口契约检测

AI 改接口很危险，尤其是 brownfield。

检测：

```
- OpenAPI diff
- GraphQL schema diff
- DTO shape diff
- response field rename / delete
- enum value change
- status code change
- required field change
```

Pact 是 code-first contract testing 工具，用来测试 HTTP 和消息集成；contract test 的目标是验证应用之间的消息是否符合共享契约。[Pact Docs](https://docs.pact.io/?utm_source=chatgpt.com)

门禁：

```YAML
- id: public-api-change
  when:
    public_api_changed: true
  require:
    contract_test_updated: true
    consumer_impact_checked: true
```

这对小团队也很有用，因为很多 bug 都来自“后端改了字段，前端炸了”。

---

## 9. Characterization / Golden Master：锁住老行为

brownfield 最大问题：

```
没人知道正确行为是什么。
```

所以第一阶段不要强求“正确”，先锁住“当前行为”。

做法：

```
- 对关键函数生成输入输出样本
- 对关键 API 保存 response snapshot
- 对复杂计算保存 golden output
- PR 改动后比较行为差异
```

门禁不是说“不允许变”，而是：

```
行为变了必须显式确认。
```

规则：

```YAML
- id: behavior-change-requires-approval
  when:
    golden_master_changed: true
  require:
    behavior_change_reason: true
    owner_approval: true
```

这对测试不足的老项目很实用。

---

## 10. Database Migration Gate：数据库变更门禁

AI 改 migration 很危险。

检测：

```
- drop column
- rename column
- change type
- add non-null column without default
- destructive migration
- missing rollback
- backward incompatible schema change
- migration 和代码不兼容
```

规则：

```YAML
- id: destructive-migration
  when:
    db_migration_destructive: true
  decision: block
  require:
    rollback_plan: true
    manual_approval: true
```

对于 brownfield app，这类规则非常刚需。

---

## 11. Runtime Blast Radius：运行时影响半径

最后一道关卡不一定只在 merge 前，也可以在 deploy 前。

检测：

```
- 是否有 feature flag
- 是否灰度发布
- 是否只影响内部用户
- 是否有 kill switch
- 是否有 metrics / logs / alert
- 是否能 rollback
```

规则：

```YAML
- id: high-risk-change-needs-flag
  when:
    risk_level: high
  require:
    feature_flag: true
    rollback_plan: true
    alert_defined: true
```

这个方向适合企业版。

---

# 门禁决策不要只有 pass / fail

建议是五种结果：

```
ALLOW
WARN
BLOCK
REQUIRE_EVIDENCE
REQUIRE_APPROVAL
```

例子：

```YAML
decision:
  status: REQUIRE_EVIDENCE
  missing:
    - changed-line coverage >= 80%
    - mutation score >= 60%
    - owner approval
```

这样比粗暴 block 更容易被团队接受。

---

# Brownfield 的关键策略：baseline + ratchet

老项目不能一上来全量扫描。

否则结果一定是：

```
3000 个问题，全红，没人用。
```

正确策略是：

```
1. 先建立 baseline
2. 旧问题不阻断
3. 新增风险阻断
4. 风险变大阻断
5. 逐步提高门槛
```

这个叫 **ratchet**，类似棘轮机制：不要求一夜变好，但不允许继续变差。

规则：

```YAML
baseline:
  mode: enabled
  ignore_existing_violations: true

ratchet:
  block_new_violations: true
  block_coverage_regression: true
  block_mutation_score_regression: true
```

这正好适合 brownfield apps。

---

# 规则优先级设计

不要所有规则都 block。

分成四级：

| 级别 | 处理 |
| --- | --- |
| P0 | 直接 block |
| P1 | 缺证据则 block |
| P2 | warn + require reason |
| P3 | 只记录趋势 |

例子：

```
P0:
- secret leaked
- destructive migration without approval
- auth bypass
- payment negative amount risk

P1:
- payment 改动但测试不足
- public API changed but no contract test
- state machine changed but no invariant test

P2:
- PR too large
- architecture boundary warning
- snapshot changed

P3:
- complexity increased
- duplicated code increased
```

这样产品不会变成烦人的“质量洁癖工具”。

---

# 最小 MVP 应该怎么做

我建议第一版只做：

> **Brownfield PR Gate for TypeScript / JavaScript repos**

因为个人、小团队、AI coding 使用者最多，也容易 demo。

## MVP 功能

```
1. codegate init
2. 生成 codegate.yml
3. 扫 PR diff
4. 识别 sensitive paths
5. 读取 coverage report
6. 检查 changed-line coverage
7. 检查 test files 是否更新
8. 对高风险文件跑 limited mutation testing
9. 跑 Semgrep custom rules
10. 输出 GitHub Check
```

## MVP 不做

```
- 不做全语言
- 不做复杂 dashboard
- 不做自动理解全部业务逻辑
- 不做 AI 自动判定
- 不做 OPA-first
- 不做企业审批流
```

---

# MVP 的 codegate.yml

```YAML
version: 1

mode: brownfield
baseline:
  enabled: true
  block_new_violations_only: true

risk:
  large_pr_lines: 600
  large_pr_files: 20

domains:
  payment:
    paths:
      - "src/payment/**"
      - "src/billing/**"
    required:
      changed_line_coverage: 0.8
      mutation_score: 0.6
      tests_changed: true

  auth:
    paths:
      - "src/auth/**"
      - "src/security/**"
    required:
      changed_line_coverage: 0.85
      owner_approval: true
      tests_changed: true

  api:
    paths:
      - "src/api/**"
      - "src/routes/**"
    required:
      contract_or_snapshot_updated: true

rules:
  - id: deleted-tests
    when:
      deleted_test_files: true
    decision: block
    reason: "Test files were deleted"

  - id: large-ai-change
    when:
      lines_changed_gt: 800
    decision: require_approval
    reason: "Large change requires human approval"

  - id: migration-risk
    when:
      changed_paths:
        - "migrations/**"
    require:
      rollback_plan: true
```

---

# 输出报告应该这样

```
CodeGate: BLOCKED

Risk level: High

Touched domains:
- payment
- api

Failed rules:
1. payment.changed_line_coverage
   Required: >= 80%
   Actual: 42%

2. payment.mutation_score
   Required: >= 60%
   Actual: 31%

3. api.contract_or_snapshot_updated
   Public API response changed, but no contract/snapshot update found.

Why this matters:
This PR changes refund calculation and public API response shape.
Existing tests pass, but they do not sufficiently prove the changed behavior is safe.

Required actions:
- Add boundary tests for refundAmount == paidAmount
- Add test for refundAmount > paidAmount
- Update API contract or snapshot
- Request payment owner approval
```

注意这个输出没有说：

```
AI 认为你代码不好
```

它说的是：

```
规则和证据不满足
```

这就更可信。

---

# 后续可以发散的高级能力

## 1. Business Rule Mining

从代码里发现候选规则：

```
- enum 状态
- if / switch 分支
- validation schema
- error message
- test names
- fixtures
- API schema
- database constraints
```

输出：

```YAML
candidate_rules:
  - id: order-refund-paid-only
    evidence:
      - src/refund.ts:42
      - tests/refund.test.ts:18
    rule: "Only PAID orders can be refunded"
    status: pending_review
```

这是产品护城河。

---

## 2. Rule Pack Marketplace

给不同技术栈内置规则包：

```
- React app pack
- Next.js app pack
- Node API pack
- Java Spring pack
- Rails pack
- Terraform pack
- Payment domain pack
- Auth domain pack
- SaaS subscription pack
```

这比卖通用扫描器更有产品感。

---

## 3. AI Agent Sandbox Contract

对 Claude Code / Cursor Agent / Copilot Agent 生成的大改动加规则：

```YAML
ai_agent:
  max_files_changed: 15
  max_lines_changed: 600
  require_tests_for_new_code: true
  require_explanation_for_behavior_change: true
  block_sensitive_paths_without_approval: true
```

核心不是识别 AI，而是：

```
当变更呈现 agentic 特征时，自动提高证据要求。
```

---

## 4. Change Plan Enforcement

要求大改动先提交 plan：

```
plan.md
- intent
- files to modify
- expected behavior change
- test plan
- rollback plan
```

实际 PR 和 plan 对比：

```
- 是否改了计划外文件？
- 是否动了敏感路径？
- 是否遗漏 test plan？
```

这对 AI agent 很关键，因为 agent 经常越界。

---

# 我会怎么定义 V1

## 产品名暂定

```
CodeGate
```

## Tagline

```
A last-mile safety gate for AI-assisted changes in brownfield codebases.
```

中文：

```
面向老项目的 AI 代码变更最后一道安全门。
```

## V1 核心

```
不是 AI review
不是 SAST
不是覆盖率平台
不是企业治理大屏

而是：
PR 级别的变更风险门禁。
```

## V1 三个核心能力

```
1. Detect impact
   识别这次改动影响范围

2. Require evidence
   根据影响范围要求测试、契约、审批、回滚证据

3. Block unsafe change
   证据不足则阻断合并
```

---

# 最重要的产品原则

## 原则 1：只拦新增风险，不清算历史旧账

brownfield app 必须这样，否则没人接入。

---

## 原则 2：AI 不做裁判

AI 只能做：

```
- suggest
- explain
- generate candidate
- draft tests
```

不能做：

```
- final allow / deny
```

---

## 原则 3：证据优先于观点

门禁语言应该是：

```
changed_line_coverage is 42%, required 80%
mutation score is 31%, required 60%
public API changed, contract not updated
state transition changed, invariant test missing
```

不要是：

```
This looks risky.
```

---

## 原则 4：先路径规则，再业务规则

最小可行顺序：

```
sensitive paths
→ diff coverage
→ mutation testing
→ contract diff
→ architecture boundary
→ business invariant
→ rule mining
```

不要一开始就做“自动理解业务逻辑”。那会变成不可靠的 AI review。

---

# 最终判断

OPA 是好底座，但不是第一产品层。

最优收敛形态是：

```
CodeGate = Brownfield-aware PR risk gate
```

技术核心是：

```
Fact extraction + Rule engine + Evidence gate
```

产品核心是：

```
让 AI coding 可以提效，但不能无限扩大破坏半径。
```

第一版最值得做的是：

```
Personal CLI + GitHub Action
针对 TypeScript / JavaScript brownfield repo
提供 sensitive path gate + changed-line coverage + scoped mutation testing + Semgrep rules + API snapshot/contract gate
```

这个方向比“AI code review”更稳，因为它不是卖模型判断力，而是卖：

> **可执行、可审计、可逐步增强的变更安全边界。**

* * *

**You:**

我不想分层，并且不同层不同的实现，我需要的是一套实现可以覆盖不同类型的用户，不需要代码，给我技术实现的考虑，检测方向够了，实现方向呢？第一版面向JAVA，金融企业。

* * *

**ChatGPT:**

我会把“实现方向”收敛成一套架构：同一套 core，个人/小团队/企业只是部署和配置不同，不分裂实现。第一版按 **Java + 金融 brownfield + CI 门禁** 来设计。

* * *

**ChatGPT:**

## 结论：第一版不要基于 OPA 做核心

第一版面向 **Java + 金融企业 + brownfield apps**，我会选：

> **一套 Java-native 的 Rule Engine + Fact Model + Evidence Gate。**  
> OPA 可以作为未来兼容目标，但不应该是第一版的核心执行引擎。

原因很直接：

| 点 | 判断 |
| --- | --- |
| 金融企业 Java 项目 | Maven / Gradle / Jenkins / GitLab / Sonar / JaCoCo / PIT / ArchUnit 生态更现实 |
| brownfield app | 最难的是提取事实，不是写 Rego |
| 企业落地 | 越少引入新运行时越好 |
| 门禁可靠性 | 规则必须可解释、可审计、可复现 |
| 用户范围 | 个人、小团队、企业都用同一套 CLI / Runner，只是配置不同 |

OPA 的问题不是能力不够，而是它会把产品心智带偏：

> 用户真正需要的不是“写策略”，而是“这次改动到底缺什么证据，为什么不能合并”。

---

# 产品实现收敛

产品形态就一个：

> **CodeGate Runner**  
> 一个可在本地、CI、企业内部平台运行的 Java 代码变更门禁执行器。

不是三套实现。

不同用户只是用法不同：

| 用户 | 使用方式 | 核心实现 |
| --- | --- | --- |
| 个人 | 本地 CLI 跑当前 diff | 同一个 Runner |
| 小团队 | CI job 跑 PR/MR | 同一个 Runner |
| 企业 | Jenkins / GitLab / GitHub Enterprise / Bitbucket Server 集成 | 同一个 Runner |
| 金融企业 | 内网部署、无云依赖、审计存档 | 同一个 Runner + Evidence Store |

核心原则：

> **一套规则模型，一套事实模型，一套执行引擎，一套报告格式。**

部署可以不同，但实现不要分裂。

---

# 第一版产品定义

## 名字暂定

**CodeGate**

## 定位

> 面向 Java brownfield 应用的代码变更门禁。  
> 通过规则和证据判断 AI / 人类生成的代码是否可以进入主干或生产。

## 不是

```
不是 AI Code Review
不是 Sonar 替代品
不是 AI 代码识别器
不是完整 AppSec 平台
不是质量大盘
```

## 是

```
PR / MR / commit 级别的变更风险门禁
```

它只回答一个问题：

> 这次变更是否有足够证据证明它没有扩大 brownfield 系统的不可控风险？

---

# 第一版技术核心

一套实现应该拆成 6 个内部模块，但它们属于同一个 Runner，不是分层产品。

```
1. Change Collector
2. Java Fact Extractor
3. Evidence Collector
4. Rule Engine
5. Decision Engine
6. Report / CI Adapter
```

---

# 1. Change Collector：先拿到“这次改了什么”

这是入口。

必须支持：

```
Git diff
PR/MR diff
目标分支 baseline
多模块 Maven / Gradle 项目
单仓多服务
Jenkins workspace
GitLab CI workspace
GitHub Actions workspace
```

第一版重点不是全量扫描，而是：

```
只看本次变更
```

因为 brownfield 项目历史问题太多，全量扫会不可用。

## 需要提取的变更事实

| 事实 | 用途 |
| --- | --- |
| 改了哪些文件 | 判断 domain / sensitive path |
| 改了哪些 Java class / method | 关联测试、覆盖率、mutation |
| 改了多少行 | 判断 blast radius |
| 是否删除测试 | 高风险 |
| 是否修改 pom.xml / build.gradle | 依赖和构建风险 |
| 是否修改 SQL / migration | 数据库风险 |
| 是否修改配置 | 环境和安全风险 |
| 是否修改 Controller / API DTO | 契约风险 |
| 是否修改 Service / Domain | 业务逻辑风险 |
| 是否修改 Security / Auth | 权限风险 |

---

# 2. Java Fact Extractor：第一版最重要

这个产品的难点不在 Rule Engine，而在 **Fact Extractor**。

Rule Engine 只会判断：

```
if facts match rules → decision
```

但你要先把 Java brownfield 项目的真实结构提取出来。

## 第一版建议只支持 Java / Spring Boot / Maven & Gradle

金融企业里这条线最现实。

应该重点识别：

```
Spring MVC / Spring Boot
Spring Security
Spring Transaction
JPA / MyBatis
Controller / Service / Repository 分层
DTO / Entity / Mapper
Maven / Gradle 多模块
JUnit / Mockito / SpringBootTest
```

---

# 3. Java fact 应该提取哪些

## 3.1 Class / Method 级别事实

不仅要知道文件变了，还要知道：

```
哪个 class 变了
哪个 method 变了
method signature 是否变了
public method 是否变了
annotation 是否变了
transaction boundary 是否变了
exception handling 是否变了
返回类型是否变了
```

金融 Java 项目里，很多风险藏在这些地方：

```
@Transactional
@PreAuthorize
@PostAuthorize
@Secured
@RequestMapping
@GetMapping
@PostMapping
@Async
@Scheduled
@Cacheable
@CacheEvict
```

比如：

| 变更 | 风险 |
| --- | --- |
| 去掉 `@PreAuthorize` | 权限绕过 |
| 改 `@Transactional` propagation | 交易一致性风险 |
| 改 public method signature | 调用方兼容风险 |
| 改 Controller response DTO | API 契约风险 |
| 改 exception handling | 错误处理语义变化 |
| 改 BigDecimal 计算 | 金额逻辑风险 |

---

## 3.2 Spring 语义事实

第一版应该强绑定 Spring，而不是做泛 Java。

需要识别：

```
Controller endpoint
Service method
Repository method
Security annotation
Transaction annotation
Validation annotation
Request / Response DTO
Configuration Bean
Scheduled Job
Async Job
External Client
Message Listener
```

因为金融企业 Java brownfield 大概率不是纯 Java，而是 Spring 系。

---

## 3.3 金融业务敏感事实

第一版 rule pack 应该内置金融敏感词和敏感结构。

例如路径、类名、方法名、字段名中包含：

```
payment
pay
refund
settlement
ledger
account
balance
fee
interest
rate
limit
quota
transfer
withdraw
deposit
reconcile
invoice
order
risk
kyc
aml
auth
permission
role
entitlement
```

中文项目还要支持：

```
支付
退款
清算
结算
账务
账户
余额
费率
利息
额度
转账
提现
充值
对账
发票
权限
角色
风控
```

这些不是最终判断，但可以用于 risk scoring。

---

# 4. Evidence Collector：收集“能不能证明安全”的证据

第一版不要自己重做所有检查。应该集成 Java 生态已有工具，然后把结果统一成 facts。

## 4.1 JaCoCo：changed-line coverage

JaCoCo 是 Java 代码覆盖率生态里的常用工具，官方提供 Maven plugin、Java agent、CLI、XML report 等集成方式；Gradle 的 JaCoCo 插件也支持生成 XML、CSV、HTML 等报告。[jacoco.org+1](https://www.jacoco.org/jacoco/trunk/doc/?utm_source=chatgpt.com)

关键点：不要看整体覆盖率。

要看：

```
本次改动行覆盖率
本次改动 method 覆盖率
本次改动 class 覆盖率
敏感 domain 的变更覆盖率
```

例子：

| 指标 | 价值 |
| --- | --- |
| project line coverage | 参考价值低 |
| changed-line coverage | 门禁价值高 |
| changed-method coverage | 很高 |
| sensitive-domain changed coverage | 很高 |

第一版必须实现：

> diff line → JaCoCo XML line hit → changed-line coverage。

这会是核心能力。

---

## 4.2 PIT：mutation testing

PIT 是 Java/JVM 生态成熟的 mutation testing 工具；它的基本原理是自动向代码注入 mutation，然后跑测试，如果测试失败则 mutation 被 killed，如果测试仍通过则 mutation survived，mutation killed 的比例可用于衡量测试质量。[PIT Mutation Testing+1](https://pitest.org/?utm_source=chatgpt.com)

第一版不要全量跑 PIT。

金融 brownfield 项目全量 mutation 会非常慢。

应该做：

```
只对 changed classes 跑
只对 high-risk changed classes 跑
只对 payment/auth/account/risk 等 domain 跑
只在 PR ready 后跑
支持 nightly 全量 baseline
```

门禁不要要求一开始 80% mutation score。brownfield 项目会死。

更现实：

```
敏感变更：mutation score 不能低于 baseline
新增逻辑：mutation score 必须达到最低门槛
核心金额逻辑：survived mutant 必须解释或修复
```

---

## 4.3 ArchUnit：架构边界

ArchUnit 官方定位是用普通 Java 单元测试框架检查 Java 架构规则，可以检查包、类、层、slice 之间的依赖和循环依赖，并通过分析 Java bytecode 完成。[ArchUnit](https://www.archunit.org/?utm_source=chatgpt.com)

第一版不要要求用户手写 ArchUnit 测试。

你的产品应该自己内置一组架构 fact：

```
Controller 是否直接依赖 Repository
Domain 是否依赖 Infrastructure
Service 是否跨 bounded context 调用
Repository 是否被非 Service 直接调用
Web 层是否直接操作 Entity
```

也可以兼容用户已有 ArchUnit 测试结果。

但产品输出应该统一：

```
architecture_boundary_violated: true
violated_rule: controller-direct-repository
```

而不是把 ArchUnit 原始输出直接扔给用户。

---

## 4.4 Semgrep / CodeQL：静态语义规则

Semgrep 支持 Java，并且 CE 版本支持单文件、跨函数常量传播、单函数 taint analysis 和语义分析；Semgrep 的 taint mode 用于表达 source → sink → sanitizer 这类数据流规则。[Semgrep+1](https://semgrep.dev/docs/languages/java?utm_source=chatgpt.com)

CodeQL 支持自定义查询，用来扩展内置安全分析，检测代码库特定漏洞、编码标准和模式。[GitHub Docs](https://docs.github.com/en/code-security/concepts/code-scanning/codeql/custom-codeql-queries?utm_source=chatgpt.com)

第一版策略：

```
Semgrep 用于快速自定义规则
CodeQL 作为可选高级集成
不要强依赖 CodeQL
```

原因：

| 工具 | 第一版角色 |
| --- | --- |
| Semgrep | 内置 rule pack，速度快，规则容易写 |
| CodeQL | 企业已有则接入，作为 external fact provider |
| Sonar | 企业已有则接入，不替代 |
| Checkmarx / Fortify | 企业已有则接入，不替代 |

---

## 4.5 SARIF：统一外部工具结果

SARIF 是 OASIS 标准，目标是为静态分析工具定义通用输出格式，让团队可以聚合和理解不同工具产生的结果；SARIF 2.1.0 已在 2020 年批准为标准。[OASIS Open+1](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=sarif&utm_source=chatgpt.com)

第一版应该支持导入 SARIF。

因为金融企业里可能已经有：

```
SonarQube
Fortify
Checkmarx
CodeQL
Semgrep
Snyk
内部扫描器
```

你不可能替代它们。

你应该做：

```
SARIF in → normalized findings → facts → rule decision
```

这样 CodeGate 是门禁编排层，不是扫描器竞争者。

---

# 5. Rule Engine：不要 OPA-first，做 typed rule engine

第一版推荐：

> 自研一个受限 DSL 的 typed rule engine。

不是为了炫技，而是为了控制复杂度。

## 为什么不要 Rego-first

| 问题 | 影响 |
| --- | --- |
| Rego 学习成本高 | 个人、小团队用不起来 |
| OPA 是外部 runtime | 金融企业内网、审计、部署更复杂 |
| OPA 不懂 Java 语义 | 仍然要自己做 Fact Extractor |
| Rego 报错体验不适合开发者 | 难解释“缺什么证据” |
| 产品需要强约束 DSL | 不是通用策略平台 |

## Rule Engine 应该长什么样

不是代码，而是产品语义：

```
当某类变更发生时，
要求某些证据存在；
如果证据不足，
输出 block / require evidence / require approval。
```

规则实体应该包括：

| 字段 | 说明 |
| --- | --- |
| trigger | 什么变更触发 |
| scope | 作用于哪些路径 / package / annotation / domain |
| required evidence | 需要哪些证据 |
| threshold | 覆盖率、mutation、风险分数门槛 |
| decision | block / warn / require evidence / require approval |
| reason template | 给用户看的解释 |
| remediation template | 建议补什么 |
| severity | P0 / P1 / P2 / P3 |
| owner | 规则归属团队 |
| baseline behavior | 是否只拦新增问题 |

---

# 6. Rule 设计：金融 Java 第一版规则包

第一版可以叫：

```
java-financial-brownfield-pack
```

## 6.1 金额规则

金融项目第一优先级。

检测：

```
BigDecimal 使用方式
double / float 用于金额
BigDecimal equals 比较
BigDecimal 构造方式
scale / rounding mode 变化
金额字段类型变化
金额计算边界条件变化
```

门禁：

```
金额逻辑变更必须有 changed-line coverage
金额逻辑变更必须有边界测试
高风险金额逻辑需要 mutation testing
rounding mode 变化必须 require approval
```

---

## 6.2 权限规则

检测：

```
Spring Security annotation 删除或修改
Controller 新增 endpoint 没有权限约束
角色判断逻辑变化
用户输入流入 role / permission update
admin / operator / maker-checker 相关代码变化
```

金融企业里特别要关注：

```
maker-checker
four-eyes principle
approval workflow
segregation of duties
```

解释一下：

* **maker-checker**：一个人发起操作，另一个人复核批准。
* **four-eyes principle**：四眼原则，关键操作至少两个人参与。
* **segregation of duties**：职责分离，避免一个角色完成所有高风险步骤。

这些在金融系统很常见。

---

## 6.3 交易一致性规则

检测：

```
@Transactional 增删改
propagation 变化
isolation 变化
rollbackFor 变化
readOnly 变化
事务方法调用链变化
异步方法进入事务逻辑
```

门禁：

```
交易边界变化必须 require evidence
核心账务服务事务变化必须 owner approval
rollbackFor 变化必须有异常路径测试
```

---

## 6.4 数据库变更规则

检测：

```
DDL migration
drop column/table
rename column
change column type
add non-null without default
index 删除
unique constraint 删除
foreign key 删除
liquibase / flyway 变更
```

门禁：

```
破坏性 migration 默认 block
需要 rollback plan
需要 backward compatibility note
需要 DBA / owner approval
```

---

## 6.5 API 契约规则

检测：

```
Controller mapping 变化
Request DTO 字段变化
Response DTO 字段变化
字段 nullable 变化
枚举值变化
HTTP status 变化
异常映射变化
```

门禁：

```
public API changed → contract/snapshot updated
字段删除 / 重命名 → require approval
response shape changed → require consumer impact note
```

---

## 6.6 架构边界规则

检测：

```
Controller 直接访问 Repository
Service 跨 domain 直接调用
Domain 依赖 infrastructure
Web 层返回 Entity
Repository 暴露给外部 package
循环依赖
```

门禁：

```
新增架构违规 block
历史架构违规 baseline ignore
敏感 domain 架构违规 require approval
```

---

## 6.7 测试充分性规则

检测：

```
changed-line coverage
changed-method coverage
mutation score
新增 public method 是否有测试
删除测试文件
修改测试但删除断言
只改 mock 不改 assertion
```

门禁：

```
敏感 domain 覆盖率不足 → require evidence
mutation survived in changed class → block / require explanation
删除测试 → block
新增 public business method 无测试 → block
```

---

# 7. 一套实现的关键：统一 Fact Model

不要让每个工具的输出各管各的。

必须统一成一个内部事实模型。

## 事实模型要围绕 change，而不是 repo

核心实体：

```
Change
ChangedFile
ChangedClass
ChangedMethod
Domain
Evidence
Finding
RuleResult
Decision
```

每个事实都要有：

| 字段 | 目的 |
| --- | --- |
| source | 来自 git / JaCoCo / PIT / Semgrep / ArchUnit / SARIF |
| confidence | 事实可靠性 |
| location | 文件、行号、class、method |
| related rule | 触发了哪些规则 |
| baseline status | 是历史问题还是新增问题 |
| severity | 风险等级 |
| explainability | 能否解释给用户 |

第一版最重要的是：

```
所有判断都能回到具体文件、具体行、具体证据。
```

否则金融企业不会信。

---

# 8. Baseline 是 brownfield 能否落地的关键

第一版必须内置 baseline 机制。

否则金融老项目一接入，必然全红。

## Baseline 做什么

第一次接入时：

```
记录当前已有违规
记录当前覆盖率
记录当前 mutation score
记录当前架构边界状态
记录当前静态扫描问题
记录当前 sensitive domain 风险水平
```

之后 PR 只判断：

```
新增问题
风险变差
敏感变更证据不足
```

不要清算历史旧账。

## Baseline 的实现考虑

| 点 | 建议 |
| --- | --- |
| baseline 粒度 | repo + module + rule + location |
| location 稳定性 | 不要只靠行号，要结合 class / method / fingerprint |
| baseline 存储 | 文件模式 + 企业 Evidence Store 都支持 |
| 过期机制 | 长期未触发可清理 |
| diff 判断 | 新增 finding 才阻断 |
| 审计 | baseline 生成和更新要留记录 |

---

# 9. Decision Engine：不要只有 pass/fail

第一版建议五种结果：

```
ALLOW
WARN
REQUIRE_EVIDENCE
REQUIRE_APPROVAL
BLOCK
```

金融企业尤其需要中间态。

因为很多东西不是“代码一定错”，而是：

```
你动了高风险业务，但证据不够。
```

比如：

| 场景 | 决策 |
| --- | --- |
| secret 泄露 | BLOCK |
| 删除权限注解 | BLOCK |
| 金额逻辑变更但测试不足 | REQUIRE_EVIDENCE |
| 事务边界变化 | REQUIRE_APPROVAL |
| 大 PR 跨多个 domain | WARN / REQUIRE_APPROVAL |
| 历史旧问题未变 | ALLOW |

这比简单质量门禁更适合 brownfield。

---

# 10. Report 设计：金融企业要“审计友好”

报告不能像普通 CI log。

它应该是一份 evidence report。

## 报告结构

```
1. Decision
2. Risk summary
3. Touched domains
4. Failed rules
5. Missing evidence
6. Evidence collected
7. Baseline comparison
8. Required actions
9. Audit metadata
```

## 每条失败规则必须说明

```
规则是什么
为什么触发
证据是什么
缺什么
怎么补
是否可以豁免
谁可以豁免
```

金融企业里，“可解释”和“可审计”比炫酷更重要。

---

# 11. 第一版部署形态：一个 Runner，三种入口

你不想分层实现，这对。

所以做一个 Runner，提供三种入口即可：

```
CLI
Maven / Gradle wrapper
CI adapter
```

但底层都是同一个 Runner。

## 入口 1：CLI

给个人和本地开发。

```
运行当前 diff
输出报告
不强依赖服务端
```

## 入口 2：Maven / Gradle

给 Java 项目自然接入。

重点不是把逻辑写进 Maven plugin，而是：

```
Maven / Gradle 只负责调用 Runner、定位报告、传递 classpath / module 信息。
```

## 入口 3：CI Adapter

给企业。

支持：

```
Jenkins
GitLab CI
GitHub Enterprise
Bitbucket Server
Azure DevOps
```

CI adapter 不做判断，只把结果回写成：

```
status check
MR comment
artifact report
SARIF report
JSON report
HTML report
```

---

# 12. 金融企业第一版应优先支持 Jenkins / GitLab

虽然 GitHub 生态体验好，但金融企业常见现实是：

```
Jenkins + GitLab
Jenkins + Bitbucket
SonarQube
Maven 私服
内网制品库
不能访问公网
```

所以第一版实现必须满足：

```
完全离线运行
规则包离线安装
工具版本固定
无云依赖
可输出审计报告
可对接已有 Sonar / Fortify / Checkmarx
```

这比 SaaS dashboard 更重要。

---

# 13. LLM 在第一版的位置

第一版最好 **不依赖 LLM**。

因为金融企业落地会问：

```
模型在哪里？
代码会不会出内网？
结果是否稳定？
能否复现？
为什么这次 block 上次不 block？
```

所以第一版判断链路必须做到：

```
无 LLM 也能完整工作。
```

LLM 只能作为可选增强：

```
解释失败规则
生成测试建议
从代码挖候选业务规则
把自然语言规范转成待审核规则
```

但不能参与：

```
最终 allow / block
```

这也是产品可信度的关键。

---

# 14. 第一版真正要做的最小闭环

如果收敛到能落地，我会做这些：

## V1 范围

```
Java 8/11/17
Maven 优先，Gradle 次之
Spring Boot / Spring MVC / Spring Security
JUnit + JaCoCo
PIT mutation testing
ArchUnit-compatible architecture checks
Semgrep Java rule pack
SARIF import
Jenkins / GitLab CI 输出
```

## V1 不做

```
不做自动修复
不做跨语言
不做 IDE
不做企业大屏
不做在线 SaaS
不做 AI 自动判定
不做全量业务理解
不做复杂审批流
```

---

# 15. V1 的默认执行流程

一次 PR 进来：

```
1. 读取目标分支和当前分支 diff
2. 识别 changed files/classes/methods
3. 识别 touched domains
4. 运行或读取 JaCoCo 报告
5. 计算 changed-line coverage
6. 对高风险 changed classes 运行 PIT
7. 运行 Java/Spring/Semgrep 规则包
8. 读取已有 SARIF / Sonar / SAST 结果
9. 与 baseline 对比，过滤历史旧问题
10. Rule Engine 判断缺失证据
11. 输出 decision 和 evidence report
12. CI 根据 decision 允许或阻断合并
```

这就是完整一套实现。

---

# 16. Rule Engine 的核心不是“规则多”，而是“证据缺口”

每条规则最终都应该落到：

```
你改了 X
X 属于 Y 风险
因此你必须提供 Z 证据
现在 Z 不存在或不达标
所以不能合并
```

例子：

| 改动 | 要求证据 |
| --- | --- |
| 金额计算 | 边界测试 + mutation score |
| 权限控制 | 安全测试 + owner approval |
| 事务边界 | 回滚路径测试 + owner approval |
| API DTO | contract/snapshot |
| DB migration | rollback plan + backward compatibility |
| 架构边界 | 无新增违规 |
| 大 PR | 拆分说明 / approval |
| 删除测试 | 明确豁免 |

---

# 17. 第一版最有杀伤力的 5 个门禁

## 1. Sensitive Domain Changed-Line Coverage Gate

> 敏感业务改动必须被测试覆盖。

尤其是：

```
payment
account
balance
settlement
auth
risk
```

---

## 2. Mutation Gate for Money/Auth Logic

> 金额 / 权限逻辑不是覆盖到就够了，测试必须能杀死错误变体。

---

## 3. Spring Security Annotation Gate

> 权限注解变化默认高风险。

检测：

```
@PreAuthorize
@PostAuthorize
@Secured
@RolesAllowed
SecurityFilterChain
AuthenticationProvider
AccessDecisionManager
```

---

## 4. Transaction Boundary Gate

> 事务边界变化需要证据。

检测：

```
@Transactional 新增、删除、属性变化
```

---

## 5. DB Migration Gate

> 破坏性数据库变更必须有回滚和兼容说明。

这五个足够支撑第一版 demo。

---

# 18. 技术选型建议

## 核心语言

建议：

```
Java 或 Kotlin
```

不要用 Node/Python 做第一版 Java 金融门禁核心。

原因：

```
更容易进入 Java 企业环境
更容易集成 Maven / Gradle
更容易处理 classpath / bytecode
更容易离线部署
更容易被金融企业接受
```

Kotlin 写内部实现会舒服，但对外保持 Java 生态兼容。

---

## Java 代码分析

建议组合：

```
JavaParser / Spoon / OpenRewrite
ASM / ClassGraph
JGit
```

选择逻辑：

| 工具方向 | 用途 |
| --- | --- |
| AST parser | 源码级 method / annotation / signature diff |
| bytecode scanner | 架构依赖、package/class dependency |
| build model reader | Maven / Gradle module 和 dependency |
| git library | diff、baseline、fingerprint |

OpenRewrite 有大量 recipe，官方 recipe catalog 覆盖 Java、Maven、YAML、Gradle、Docker、GitHub Actions 等，并包含 static analysis remediation 方向。[docs.openrewrite.org+1](https://docs.openrewrite.org/recipes/staticanalysis?utm_source=chatgpt.com)

但第一版不要过度依赖 OpenRewrite 做门禁核心。它更适合：

```
代码结构分析
未来自动修复
规则发现
迁移 recipe
```

---

## 外部工具集成

建议：

| 能力 | 工具 |
| --- | --- |
| 覆盖率 | JaCoCo |
| Mutation | PIT |
| 架构边界 | ArchUnit 思路 / bytecode analysis |
| 静态规则 | Semgrep |
| 高级安全 | CodeQL / Fortify / Checkmarx 作为 SARIF 输入 |
| 依赖规则 | Maven Enforcer / OWASP Dependency-Check / 企业现有 SCA |
| 报告统一 | JSON + HTML + SARIF |

Maven Enforcer Plugin 本身提供很多内置规则，例如依赖收敛、重复依赖版本、禁止某些依赖管理作用域等，适合接入依赖和构建一致性方向。[maven.apache.org](https://maven.apache.org/enforcer/enforcer-rules/index.html?utm_source=chatgpt.com)

---

# 19. 第一版数据存储

一开始不需要复杂数据库。

## Local / CI 模式

```
.codegate/
  baseline
  rules
  evidence
  reports
```

## 企业模式

同一套 Runner 可以把结果上传到内部 Evidence Store。

Evidence Store 存：

```
repo
branch
commit
PR/MR
rules version
tool versions
baseline version
facts
decisions
waivers
reports
```

注意：这不是另一套实现，只是同一 Runner 的结果存储方式不同。

---

# 20. Waiver / 豁免机制必须第一版就考虑

金融企业没有豁免机制，门禁一定会被绕过。

豁免不能是“随便点通过”。

需要：

```
豁免规则 ID
豁免范围
豁免原因
豁免人
有效期
是否一次性
是否需要二级审批
审计记录
```

第一版可以简单做：

```
repo 内 waiver 文件 + CI 审计输出
```

企业以后再接审批系统。

---

# 21. 成功标准

第一版不需要证明“能发现所有 AI 代码问题”。

它要证明：

```
在测试不足的 Java brownfield 项目里，
AI / 人类生成的高风险变更，
不能在缺少证据的情况下悄悄进入主干。
```

最好的 demo 场景：

| 场景 | 结果 |
| --- | --- |
| AI 修改退款金额边界，测试绿 | mutation gate block |
| AI 删除权限注解，测试绿 | security annotation gate block |
| AI 改事务 propagation，测试绿 | transaction boundary gate require approval |
| AI 改 API DTO 字段，测试绿 | contract gate require evidence |
| AI 改 migration drop column，测试绿 | DB migration gate block |
| AI 大范围重构跨 5 个 domain | blast radius gate require split/approval |

---

# 最终建议

第一版就做一个东西：

> **Java Financial Brownfield CodeGate Runner**

核心是：

```
Git diff → Java facts → evidence collection → typed rule engine → decision report
```

不要 OPA-first。

不要 AI-first。

不要 dashboard-first。

不要全语言。

先把 Java 金融 brownfield 的 5 个高价值门禁打穿：

```
1. changed-line coverage
2. mutation testing
3. Spring Security annotation
4. transaction boundary
5. database migration
```

这套东西个人可以本地跑，小团队可以 CI 跑，企业可以内网 Jenkins / GitLab 跑。

实现是一套，只是运行入口和规则配置不同。

* * *

**You:**

别给我看规则方向了， 我现在需要技术实现方案，目标检测语言是Java，不代表这个产品本身也需要用Java，golang?rust?python?

* * *

**ChatGPT:**

我会只讲实现，不再展开规则内容。核心问题是：**目标分析 Java，不等于实现语言必须是 Java；但要看你要做的是“源码分析器”“CI Runner”“企业内网工具”还是“高性能多语言引擎”。**

* * *

**ChatGPT:**

## 直接结论

第一版我会选：

> **Kotlin / JVM 做核心实现。**  
> 不是 Go，不是 Rust，不是 Python。

更具体：

```
CodeGate Runner = Kotlin/JVM CLI + Java analysis engine + report adapters
```

目标项目是 Java，尤其是金融 Java brownfield app，这意味着最难的问题不是 CLI 写得漂不漂亮，而是：

```
能不能准确理解 Maven / Gradle / Spring / annotation / classpath / bytecode / JaCoCo / PIT / 企业已有扫描结果。
```

这些东西天然在 JVM 生态里。用 Go / Rust / Python 做核心，最后大概率还是要不断 shell out 到 Java 工具，复杂度会上升。

---

# 语言选择判断

| 实现语言 | 适合作为什么 | 不适合作为什么 | 结论 |
| --- | --- | --- | --- |
| Kotlin / JVM | Java 语义分析、规则引擎、CI Runner、企业内网工具 | 启动速度不如 Go/Rust | **V1 最优** |
| Java | 同上，更保守 | 表达力弱一些 | 可选，但我更偏 Kotlin |
| Go | CLI wrapper、轻量 server、分发体验 | 深度 Java 分析核心 | 不适合 V1 core |
| Rust | 高性能 parser、tree-sitter 多语言索引 | Java 语义、classpath、Spring 语义 | 后期可做性能模块 |
| Python | 原型、实验、规则生成脚本 | 金融企业生产门禁 | 不建议做 core |

---

# 为什么不是 Go

Go 的优点很明显：

```
单文件分发舒服
启动快
CI 工具体验好
写 daemon / server 简单
```

但对这个产品来说，核心矛盾不是这些。

你的核心矛盾是：

```
Java 项目语义理解。
```

尤其是金融 Java 项目：

```
Maven 多模块
Gradle 多模块
Spring Boot
Spring Security
@Transactional
JPA / MyBatis
Lombok
MapStruct
JaCoCo XML
PIT mutation report
Sonar / Fortify / Checkmarx / Semgrep / CodeQL 结果
```

Go 自己没有成熟的 Java semantic ecosystem。你可以用 Tree-sitter 解析 Java，但 Tree-sitter 主要提供 concrete syntax tree 和增量解析能力，不负责 Java classpath、类型解析、Spring annotation 语义、Maven/Gradle build model。Tree-sitter 本身适合做快速语法索引，不适合做第一版的 Java 金融门禁核心。[tree-sitter.github.io+1](https://tree-sitter.github.io/?utm_source=chatgpt.com)

所以如果用 Go，真实架构会变成：

```
Go CLI
  → 调 JVM 分析器
  → 调 Maven / Gradle
  → 调 JaCoCo
  → 调 PIT
  → 调 Semgrep
  → 读一堆报告
```

这不是一套更简单的实现，而是多一层 orchestration。

---

# 为什么不是 Rust

Rust 的优势是：

```
高性能
内存安全
单 binary
适合做 parser/indexer
```

如果未来你要做多语言高速 pre-scan，Rust + Tree-sitter 是合理路线。Tree-sitter 有 Java grammar，也有 Rust crate 可以提供 Java language support。[docs.rs](https://docs.rs/tree-sitter-java?utm_source=chatgpt.com)

但第一版不应该走 Rust core。

原因：

```
Rust 可以很好地知道“这里有个 method declaration”
但很难自然知道：
- 这个 method 的真实类型是什么
- 这个 annotation 来自哪个包
- 这个 class 在 Maven reactor 哪个 module
- 这个 Spring endpoint 是否继承了 class-level mapping
- 这个 Controller 返回 DTO 是否破坏了契约
- Lombok / MapStruct / annotation processor 生成了什么
```

这些都不是纯 parser 能解决的。

Rust 可以是后期组件，不是 V1 主体。

---

# 为什么不是 Python

Python 可以做 POC，但不适合金融企业生产门禁。

原因：

```
依赖环境难控制
venv / pip / 内网镜像麻烦
启动和性能一般
企业 CI 中可复现性差
安全审计成本高
最终仍然要调用 JVM 工具
```

金融企业会关心：

```
这个门禁是不是 deterministic？
依赖怎么锁？
能不能离线？
能不能审计？
版本怎么冻结？
结果能不能复现？
```

Python 不是不能做，但会增加销售和落地摩擦。

---

# 为什么选 Kotlin / JVM

因为第一版最重要的是：

```
Java-aware
build-aware
CI-friendly
offline-friendly
auditable
```

Kotlin/JVM 能直接复用 Java 生态：

| 能力 | JVM 生态优势 |
| --- | --- |
| Git diff | JGit 是纯 Java Git 实现，可以读写 Git repository 和 working directory。[GitHub](https://github.com/eclipse-jgit/jgit?utm_source=chatgpt.com) |
| Maven 模型 | Maven Model Builder 可以构建 effective model，处理 profile、继承、插值等。[maven.apache.org](https://maven.apache.org/ref/current/maven-model-builder/?utm_source=chatgpt.com) |
| Gradle 模型 | Gradle Tooling API 允许外部软件执行、监控 Gradle build，并查询 build details。[docs.gradle.org](https://docs.gradle.org/current/userguide/tooling_api.html?utm_source=chatgpt.com) |
| Java AST / type analysis | OpenRewrite 的 LST 支持 type attribution，用来做准确匹配和转换。[docs.openrewrite.org+1](https://docs.openrewrite.org/concepts-and-explanations/lossless-semantic-trees?utm_source=chatgpt.com) |
| Java source analysis | Spoon 是开源 Java source code analysis / transformation 库，支持现代 Java 版本。[spoon.gforge.inria.fr](https://spoon.gforge.inria.fr/?utm_source=chatgpt.com) |
| 覆盖率 | JaCoCo Maven plugin 提供 runtime agent 和 report creation，JaCoCo 也有 XML report 能力。[eclemma.org+1](https://www.eclemma.org/jacoco/trunk/doc/maven.html?utm_source=chatgpt.com) |
| mutation testing | PIT 是 Java/JVM mutation testing 工具。[eow.alc.co.jp](https://eow.alc.co.jp/search?q=pit&utm_source=chatgpt.com) |
| classpath / bytecode | ClassGraph 是 JVM classpath/module scanner。[GitHub](https://github.com/classgraph/classgraph?utm_source=chatgpt.com) |
| 外部结果统一 | SARIF 是 OASIS 定义的静态分析结果交换格式，适合聚合多工具结果。[OASIS Open](https://www.oasis-open.org/standard/sarifv2-1-os/?utm_source=chatgpt.com) |

Kotlin 的优势是工程实现更舒服：

```
data class 做 Fact Model
sealed class 做 Decision / Finding / Evidence 类型
DSL parser 更好写
null safety 对复杂分析结果友好
JVM 生态无缝集成
```

对外不需要宣传 Kotlin。对客户来说它就是：

```
一个可离线运行的 CodeGate Runner。
```

---

# 推荐实现形态

不要做 server-first。

第一版应该是：

```
一个本地优先的 Runner。
```

也就是：

```
codegate-runner
```

它可以运行在：

```
开发者本机
Jenkins
GitLab CI
GitHub Enterprise
Bitbucket Server
企业内部构建机
```

同一个 binary / jar，同一套逻辑。

---

# 推荐技术架构

```
CodeGate Runner
│
├─ Workspace Loader
├─ Git Diff Engine
├─ Build Model Resolver
├─ Java Semantic Analyzer
├─ Bytecode / Classpath Analyzer
├─ Evidence Importer
├─ Rule Evaluator
├─ Baseline Manager
├─ Decision Engine
└─ Report Adapter
```

重点是：**所有输入都转成统一 Fact Model**。

不要让 JaCoCo、PIT、Semgrep、Sonar、Git diff 各说各话。

---

# 1. Workspace Loader

负责识别当前项目结构。

需要支持：

```
单模块 Maven
多模块 Maven
单模块 Gradle
多模块 Gradle
混合历史项目
非标准 src 路径
生成代码目录
测试目录
target / build 输出目录
```

实现重点：

```
不要自己猜 classpath。
```

应该通过 Maven / Gradle 生态拿 build model：

```
Maven → effective POM / reactor modules
Gradle → Tooling API / project model
```

这也是为什么 JVM 实现天然更合适。

---

# 2. Git Diff Engine

目标不是简单知道文件变了，而是要把 diff 映射到 Java 语义对象：

```
changed file
changed line
changed class
changed method
changed annotation
changed method signature
changed endpoint
changed transaction boundary
changed DTO field
```

实现上要做几件事：

```
1. 获取 base commit 和 head commit
2. 识别 added / modified / deleted / renamed files
3. 提取 changed line ranges
4. 把 changed line ranges 映射到 AST method ranges
5. 得到 changed methods / changed classes
```

这里建议用：

```
JGit + 必要时 fallback 到 native git
```

JGit 好处是纯 JVM、可嵌入、离线友好。native git 好处是和 CI 行为一致，尤其是 rename detection、submodule、浅克隆场景。第一版可以优先 JGit，但保留 native git fallback。

---

# 3. Java Semantic Analyzer

这是核心。

不要只用文本扫描。

需要能理解：

```
class
method
field
annotation
import
inheritance
interface implementation
method call
DTO structure
Spring mapping
transaction annotation
security annotation
exception handler
```

## 推荐路线

第一选择：

```
OpenRewrite LST + type attribution
```

OpenRewrite 的 LST 是 lossless semantic tree，不只是传统 AST；官方文档也强调 type attribution 对准确匹配很重要，因为没有类型信息时无法区分语法类似但语义不同的代码。[docs.openrewrite.org+1](https://docs.openrewrite.org/concepts-and-explanations/lossless-semantic-trees?utm_source=chatgpt.com)

这对你很关键。

比如你不能只看：

```
logger.info(...)
```

你要知道它到底是哪个 logger 类型。

同理，你不能只看：

```
@PreAuthorize
```

你要知道它是不是：

```
org.springframework.security.access.prepost.PreAuthorize
```

## 备用路线

```
Spoon
JavaParser + Symbol Solver
```

Spoon 是成熟的 Java source analysis / transformation 库。JavaParser 可以生成 Java AST，Java Symbol Solver 可以把符号关联到 declaration、计算表达式类型。[spoon.gforge.inria.fr+2JavaParser+2](https://spoon.gforge.inria.fr/?utm_source=chatgpt.com)

我的建议：

```
V1：OpenRewrite 优先
保留 Spoon / JavaParser 作为 fallback 或特定任务工具
```

不要一开始自己写 Java parser。

---

# 4. Bytecode / Classpath Analyzer

Java brownfield 不能只看源码。

原因：

```
Lombok 生成代码
MapStruct 生成 mapper
annotation processor 生成类
多模块依赖
第三方 jar
编译后 annotation 信息
真实方法签名
```

所以需要一个 bytecode / classpath analyzer。

用途：

```
解析 target/classes
解析依赖 jar
确认 annotation 是否存在
确认 class dependency
确认 method signature
确认 package dependency
识别循环依赖
识别 Controller → Repository 这类依赖路径
```

这里可以用：

```
ClassGraph / ASM
```

ClassGraph 本身就是 JVM classpath / module scanner。[GitHub](https://github.com/classgraph/classgraph?utm_source=chatgpt.com)

设计上应该是：

```
source facts + bytecode facts 合并
```

如果两者冲突：

```
bytecode facts 优先用于实际构建结果
source facts 优先用于定位和解释
```

---

# 5. Evidence Importer

不要自己重做所有工具。

第一版应该是：

```
consume existing reports first
run tools second
```

也就是：

```
如果 CI 已经生成 JaCoCo XML，就读取
如果 CI 已经跑 PIT，就读取
如果已有 Sonar / Fortify / Checkmarx / Semgrep，就读取 SARIF 或可导出的报告
如果没有，CodeGate 再按配置触发
```

这对金融企业很重要，因为他们通常已有一套安全扫描和质量工具。

## Evidence Importer 要支持

```
JaCoCo XML
PIT report
SARIF
JUnit XML
Maven Surefire / Failsafe report
Gradle test report
Semgrep JSON / SARIF
Sonar issue export，视企业可用 API 而定
```

Semgrep CLI 支持 SARIF 输出，SARIF 本身就是为聚合不同静态分析工具结果设计的标准格式。[Semgrep+1](https://semgrep.dev/docs/customize-semgrep-ce?utm_source=chatgpt.com)

---

# 6. Changed-line Coverage 计算

这是 V1 的核心能力之一。

实现路径：

```
Git diff changed lines
        ↓
Java source file path normalization
        ↓
JaCoCo XML line coverage
        ↓
changed line hit / missed
        ↓
method-level aggregation
        ↓
domain-level aggregation
```

注意几个坑：

```
路径不一致：CI workspace path vs repo relative path
多模块 report：每个 module 一个 JaCoCo XML
聚合 report：有些企业用 aggregate report
生成代码：需要排除 generated sources
重命名文件：要处理 old path / new path
行号漂移：baseline 不能只靠 line number
```

所以内部不要只存：

```
file + line
```

要存：

```
file + class + method signature + line range + source hash
```

---

# 7. Mutation Testing 集成

不要让 CodeGate 自己实现 mutation testing。

直接集成 PIT。

实现策略：

```
1. 识别 high-risk changed classes
2. 生成 PIT targetClasses / targetTests 范围
3. 调用 Maven / Gradle / PIT
4. 读取 PIT report
5. 把 survived mutants 映射回 changed methods
6. 生成 evidence facts
```

关键是控制成本。

不能全量跑 mutation。

第一版要做到：

```
只对 changed classes 跑
只对敏感 package 跑
只对方法级别变更跑
支持超时
支持 fallback 为 warn / require evidence
支持 nightly full baseline
```

PIT 结果不要原样扔给用户。要归一化成：

```
class
method
mutator type
line
killed / survived
related changed line?
related sensitive domain?
```

---

# 8. Rule Evaluator：自研 typed evaluator，不要 OPA-first

这里不是讲规则方向，而是讲实现方式。

我建议：

```
YAML / JSON config
        ↓
parse
        ↓
typed Rule IR
        ↓
evaluate against Fact Model
        ↓
Decision
```

不要在 V1 引入通用脚本语言。

不要让用户写任意表达式。

更不要把 Rego 当第一界面。

原因：

```
金融企业要可审计
规则要可解释
失败原因要结构化
IDE / CI 输出要稳定
同一输入必须同一输出
```

## Rule Engine 内部应该是强类型的

例如内部对象是：

```
Trigger
Scope
EvidenceRequirement
Threshold
DecisionAction
ExplanationTemplate
RemediationTemplate
WaiverPolicy
```

而不是：

```
一坨字符串表达式。
```

好处：

```
可以做规则校验
可以做版本迁移
可以生成文档
可以生成 UI
可以解释为什么触发
可以做审计
```

## OPA 放在哪里

OPA 可以以后作为：

```
enterprise advanced policy backend
```

但不是 V1 core。

V1 你需要的是：

```
Java change facts → typed rule evaluation → evidence report
```

不是：

```
先把所有东西塞进 input.json，再让用户写 Rego。
```

---

# 9. Baseline Manager

brownfield app 没有 baseline 就落不了地。

Baseline 实现不要复杂，但要稳定。

## Baseline 存什么

```
rule id
finding fingerprint
file path
class name
method signature
source range
evidence value
first seen commit
last seen commit
status
```

## Fingerprint 不能只靠行号

行号会变。

更稳的是：

```
repo + module + rule id + class + method signature + normalized location + code snippet hash
```

## Baseline 存储

V1 不需要数据库。

直接：

```
.codegate/baseline.json
.codegate/evidence/*.json
.codegate/reports/*.html
```

企业场景可以把同样 JSON 上传内部 artifact store 或数据库。

这仍然是一套实现。

---

# 10. Report Adapter

金融企业里报告比 dashboard 更重要。

V1 输出必须包括：

```
machine-readable JSON
human-readable HTML
CI summary markdown
JUnit XML，方便 Jenkins/GitLab 展示
SARIF，可接代码扫描展示系统
```

SARIF 适合承载 findings，但不适合承载所有业务门禁证据；所以你需要自己的 JSON evidence report，同时可导出 SARIF 给平台消费。

报告要保证：

```
每个 decision 都能回溯到 facts
每个 fact 都能回溯到 source
每个 source 都能回溯到工具版本和 commit
```

---

# 推荐 V1 技术栈

## 实现语言

```
Kotlin / JVM
```

## 运行要求

```
Runner 自己要求 JDK 17+
目标项目可分析 Java 8 / 11 / 17，后续支持 21
```

解释一下：Runner 用较新的 JDK 跑，不代表目标项目必须升级。目标项目还是用自己的 Maven / Gradle toolchain 编译。

## 核心依赖方向

| 模块 | 技术选择 |
| --- | --- |
| CLI | picocli 或类似 JVM CLI 框架 |
| Git | JGit + native git fallback |
| Maven | Maven Model Builder |
| Gradle | Gradle Tooling API |
| Source analysis | OpenRewrite LST/type attribution |
| Fallback analysis | Spoon / JavaParser |
| Bytecode/classpath | ClassGraph / ASM |
| Coverage | JaCoCo XML importer |
| Mutation | PIT runner/report importer |
| Static findings | SARIF importer |
| External scan | Semgrep CLI optional |
| Config | YAML + JSON schema |
| Report | JSON + HTML + Markdown + JUnit XML + SARIF |
| Packaging | fat JAR first，Docker image second，native image later |

---

# Packaging：不要第一版追求单 binary

Go/Rust 单 binary 很诱人，但这不是 V1 最重要的问题。

我建议顺序：

## V1

```
fat JAR
Docker image
CI template
offline bundle
```

## V1.5

```
jlink custom runtime
```

## V2

```
GraalVM Native Image
```

GraalVM Native Image 可以把 Java 代码 ahead-of-time 编译成 native executable。[GraalVM](https://www.graalvm.org/latest/reference-manual/native-image/?utm_source=chatgpt.com)

但它对反射、动态 classloading、复杂 JVM 工具集成可能带来额外配置成本。你的 V1 会大量碰到 parser、classpath、external process、report importer，不值得一开始就追 native binary。

---

# 一套实现怎么覆盖不同用户

不是做三套产品，而是同一个 Runner 支持三种运行上下文：

```
local
ci
enterprise
```

但内部流程完全一样：

```
load workspace
load config
collect facts
import evidence
evaluate rules
write report
exit with decision code
```

区别只是输入输出：

| 场景 | 输入 | 输出 |
| --- | --- | --- |
| 本地 | 当前 git diff | terminal + HTML report |
| 小团队 CI | PR diff + CI reports | CI status + markdown summary |
| 金融企业 | MR diff + enterprise reports + baseline | audit report + artifact + status check |

不要分实现。只分 adapter。

---

# 最关键的实现原则

## 1. Build-aware，不要 source-only

金融 Java 项目里，source-only 一定不够。

必须理解：

```
classpath
module
generated source
annotation processor
compiled class
test report
coverage report
```

---

## 2. Consume-first，不要 replace-first

不要替代企业已有工具。

先读：

```
JaCoCo
PIT
Sonar
Fortify
Checkmarx
Semgrep
CodeQL
JUnit
Maven / Gradle reports
```

然后统一成 CodeGate facts。

---

## 3. Deterministic

同一个 commit、同一个配置、同一组报告，必须得到同一个 decision。

这就是为什么 LLM、Python 脚本、动态策略语言都不适合做 V1 核心。

---

## 4. Offline-first

金融企业默认：

```
不能依赖公网
不能上传代码
不能远程调用模型
不能动态下载规则
```

所以：

```
规则包离线安装
工具版本固定
报告本地生成
依赖来源可审计
```

---

## 5. Stable Fact Schema

这个产品长期护城河不是 parser，而是 fact schema。

必须从第一版就定义清楚：

```
ChangeFact
JavaSymbolFact
CoverageFact
MutationFact
StaticFindingFact
ArchitectureFact
EvidenceFact
DecisionFact
BaselineFact
WaiverFact
```

未来你支持 Go、JS、Python，也只是增加 analyzer，把结果映射到同一套 fact schema。

---

# 如果你硬要非 JVM 实现

那我只接受一种：

```
Go outer runner + JVM Java analyzer sidecar
```

结构：

```
Go CLI / Server
        ↓
JVM Analyzer
        ↓
Fact JSON / Protobuf
        ↓
Go Rule Evaluator / Reporter
```

但我不建议 V1 这么做。

因为你会立刻多出这些问题：

```
两个运行时
两个依赖体系
进程通信
版本兼容
日志归并
错误处理
本地调试复杂
离线包更复杂
```

除非你的第一目标是做一个漂亮的跨平台 CLI 工具，否则没必要。

---

# 最终建议

第一版就这样定：

```
Language: Kotlin/JVM
Runtime: JDK 17+
Distribution: fat JAR + Docker image + offline bundle
Core analyzer: OpenRewrite-first Java semantic analyzer
Build integration: Maven Model Builder + Gradle Tooling API
Coverage: JaCoCo XML importer
Mutation: PIT report importer / scoped runner
Findings: SARIF importer
Git: JGit + native git fallback
Rule engine: custom typed evaluator
Storage: local JSON baseline + evidence reports
Output: JSON / HTML / Markdown / JUnit XML / SARIF
```

一句话：

> **不要为了 CLI 分发体验选 Go/Rust。这个产品的第一性原理是 Java brownfield 语义可信度，所以 core 应该站在 JVM 生态里。**

* * *

**You:**

不太合理，如果按你这样的逻辑，这个应用如果要新增语言支持，就得重写一套对应语言咯？

* * *

**ChatGPT:**

你这个反驳成立。上一版把“Java 第一版”推成了“JVM core”，这会导致未来多语言扩展很重；更合理的是：**核心产品必须语言无关，Java 只是第一批 analyzer。**

* * *

**ChatGPT:**

对，这个质疑是对的。

我前面把 **“第一版目标语言是 Java”** 和 **“产品 core 也应该 JVM 化”** 混在一起了。  
如果真按那个方案走，未来新增 TypeScript、Python、Go、C# 时，确实会变成：

```
每支持一种语言，就要重新做一套分析、规则、报告、门禁逻辑
```

这不合理。

正确架构应该是：

```
Core 是语言无关的
Analyzer 是语言相关的
Evidence / Rule / Decision / Report 是统一的
```

也就是说：

> 新增语言时，只新增 analyzer，不重写产品 core。

---

# 修正后的技术方向

我会改成：

```
CodeGate Core：Go
Java Analyzer：JVM / Kotlin / Java
通信协议：JSON / Protobuf Fact Bundle
规则引擎：Go 内置 typed evaluator
报告与门禁：Go core 统一输出
```

产品形态：

```
codegate
  ├─ core runner
  ├─ analyzer protocol
  ├─ java-analyzer
  ├─ evidence importers
  ├─ rule evaluator
  └─ report adapters
```

用户看到的是一个工具：

```
codegate check
```

不是一堆工具。

内部是否调用 Java analyzer，对用户不暴露成“另一套产品”。

---

# 关键设计：Analyzer 只是事实生产者

Java analyzer 不做最终判断。

它只输出事实：

```
这个 PR 改了哪些 class
哪些 method 变了
哪些 annotation 变了
哪些 endpoint 变了
哪些 transaction boundary 变了
哪些 dependency edge 变了
哪些 method 有覆盖率
哪些 mutation survived
```

然后统一交给 Core。

Core 才做：

```
baseline comparison
evidence aggregation
rule evaluation
decision
report
CI status
audit output
```

这样未来新增语言时，新增的是：

```
python-analyzer
typescript-analyzer
go-analyzer
csharp-analyzer
```

但不动：

```
rule engine
baseline
decision
report
CI adapter
waiver
evidence store
```

---

# 推荐实现语言

## Core：Go

我现在会选 **Go 做产品 core**。

原因：

| 维度 | Go 的优势 |
| --- | --- |
| CLI 分发 | 单 binary，CI 里好用 |
| 企业内网 | 离线部署简单 |
| 进程编排 | 适合调用 analyzer、Maven、Gradle、PIT、Semgrep |
| 配置处理 | YAML / JSON / schema 很成熟 |
| 报告输出 | JSON / HTML / Markdown / JUnit XML / SARIF 都好做 |
| 可维护性 | 比 Rust 快，团队门槛低 |
| 长期多语言 | 不被 JVM 生态绑死 |

所以：

```
CodeGate Core = Go
```

---

## Java Analyzer：JVM

Java analyzer 仍然建议 JVM 实现。

但它只是插件，不是产品 core。

原因：

```
Java 语义分析离不开 JVM 生态
```

尤其是：

```
Maven / Gradle
Spring annotation
classpath
bytecode
Lombok / MapStruct
JaCoCo
PIT
OpenRewrite / Spoon / JavaParser
ClassGraph / ASM
```

这些让 Java analyzer 用 JVM 写更合理。

所以：

```
Java support = JVM analyzer plugin
```

不是：

```
整个产品 = JVM 应用
```

这个区别很重要。

---

# 不可能完全“一套 analyzer 支持所有语言”

这里要说清楚。

如果你想做深度门禁，不可能完全用一套 analyzer 支持所有语言。

因为不同语言的语义差异很大：

| 语言 | 特有问题 |
| --- | --- |
| Java | classpath、annotation、Spring、bytecode、Maven、Gradle |
| TypeScript | type checker、tsconfig、path alias、React、AST transform |
| Python | 动态类型、import resolution、runtime monkey patch |
| Go | module、interface、package-level visibility |
| C# | Roslyn、attribute、project/solution model |
| Kotlin | compiler plugin、nullability、JVM interop |

所以合理边界是：

```
每种语言需要自己的 analyzer
但所有 analyzer 输出统一 facts
```

类比一下：

```
compiler frontend per language
shared IR
shared backend
```

这里的 shared IR 就是你的 **Fact Model**。

---

# 最核心的设计：Fact Model

这个产品的长期核心不是 Java analyzer，而是：

> **语言无关的变更事实模型。**

只要 Fact Model 稳，语言扩展就不会推倒重来。

---

## Fact Model 应该长这样

### 1. Change Fact

描述这次改了什么。

```
Change
- repo
- base commit
- head commit
- changed files
- changed ranges
- added / modified / deleted / renamed
- generated files
- test files changed
```

---

### 2. Symbol Fact

描述代码符号。

```
Symbol
- language
- module
- file
- kind: class / method / field / function / endpoint / config
- qualified name
- signature
- visibility
- range
- annotations / decorators / attributes
```

Java 里是：

```
class / method / field / annotation
```

TypeScript 里可以是：

```
function / class / component / hook / route handler
```

Go 里可以是：

```
package / function / method / interface
```

Core 不关心语言细节，只看统一 Symbol。

---

### 3. Semantic Change Fact

描述语义变化。

```
SemanticChange
- symbol changed
- signature changed
- annotation changed
- visibility changed
- return type changed
- dependency changed
- entrypoint changed
- config changed
```

Java analyzer 可以输出：

```
@Transactional changed
@PreAuthorize removed
@RequestMapping path changed
public method signature changed
```

TypeScript analyzer 以后可以输出：

```
API route changed
exported function signature changed
React prop type changed
```

Core 只看：

```
entrypoint_changed
public_contract_changed
security_boundary_changed
transaction_boundary_changed
```

---

### 4. Dependency Fact

描述依赖关系。

```
DependencyEdge
- from symbol
- to symbol
- edge type: import / call / extends / implements / injects / configures
```

这个用于架构边界、影响范围、blast radius。

---

### 5. Entry Point Fact

描述外部入口。

```
EntryPoint
- HTTP endpoint
- message consumer
- scheduled job
- CLI command
- public API
- event handler
```

Java 里来自：

```
Spring Controller
Kafka Listener
Scheduled Job
REST endpoint
```

未来 TypeScript 里来自：

```
Next.js route
Express handler
React server action
```

Core 统一处理 entrypoint，不关心来源语言。

---

### 6. Evidence Fact

描述证据。

```
Evidence
- test passed
- changed-line coverage
- mutation result
- static finding
- contract diff
- snapshot diff
- dependency scan
- approval
- waiver
```

这个完全语言无关。

Java coverage 来自 JaCoCo。

TypeScript coverage 以后可能来自 Istanbul。

Core 只看：

```
changed-line coverage = 42%
```

不关心它来自 JaCoCo 还是 Istanbul。

---

### 7. Decision Fact

描述最终结果。

```
Decision
- allow
- warn
- require evidence
- require approval
- block
- failed rules
- missing evidence
- audit metadata
```

这个永远由 Core 产生。

---

# 推荐系统架构

```
               ┌────────────────────┐
               │     Git Diff        │
               └─────────┬──────────┘
                         ↓
               ┌────────────────────┐
               │   CodeGate Core     │
               │        Go           │
               └─────────┬──────────┘
                         ↓
          ┌──────────────────────────────┐
          │ Analyzer Protocol             │
          │ input: workspace + diff       │
          │ output: Fact Bundle           │
          └─────────┬────────────────────┘
                    ↓
      ┌─────────────┬─────────────┬─────────────┐
      │ Java Analyzer│ TS Analyzer │ Go Analyzer │
      │ JVM          │ Node/TS     │ Go          │
      └─────────────┴─────────────┴─────────────┘
                    ↓
          ┌──────────────────────────────┐
          │ Normalized Fact Store         │
          └─────────┬────────────────────┘
                    ↓
          ┌──────────────────────────────┐
          │ Evidence Importers            │
          │ JaCoCo / PIT / SARIF / JUnit  │
          └─────────┬────────────────────┘
                    ↓
          ┌──────────────────────────────┐
          │ Rule Evaluator                │
          └─────────┬────────────────────┘
                    ↓
          ┌──────────────────────────────┐
          │ Decision + Report             │
          └──────────────────────────────┘
```

这才是可扩展方向。

---

# Analyzer Protocol 设计

Analyzer 不应该和 Core 强耦合。

它应该像一个独立 worker。

## Analyzer 输入

```
- workspace path
- base commit
- head commit
- changed files
- changed ranges
- build metadata
- config
- cache dir
```

## Analyzer 输出

```
Fact Bundle
- analyzer name
- analyzer version
- language
- capabilities
- symbols
- changed symbols
- semantic changes
- dependency edges
- entrypoints
- diagnostics
```

## Analyzer 运行方式

第一版用：

```
subprocess + JSON / NDJSON
```

不要一开始上 gRPC。

理由：

```
CI 环境简单
日志好收集
debug 方便
离线部署容易
跨语言容易
```

后面需要常驻服务时，再加：

```
gRPC / local daemon
```

---

# Analyzer Capability 很重要

每个 analyzer 启动时声明自己能做什么。

例如 Java analyzer：

```
capabilities:
- syntax
- symbols
- type attribution
- annotations
- method signatures
- dependency graph
- spring entrypoints
- bytecode scan
- coverage mapping
```

早期 TypeScript analyzer 可能只有：

```
capabilities:
- syntax
- symbols
- exports
- route handlers
- coverage mapping
```

Core 根据 capability 决定哪些规则可执行。

如果某个 capability 缺失，输出应该是：

```
missing capability
```

而不是假装检查通过。

这对产品可信度很重要。

---

# V1 技术选型

## 整体推荐

```
Core: Go
Java Analyzer: Kotlin / Java
Protocol: JSON first, Protobuf later
Config: YAML + JSON Schema
Storage: local JSON / SQLite optional
Reports: JSON + HTML + Markdown + JUnit XML + SARIF
Packaging: Go binary + bundled Java analyzer jar + Docker image
```

---

# 为什么 Core 选 Go，而不是 Rust

Rust 也可以，但我不建议 V1 用 Rust 做 core。

Rust 适合：

```
高性能 indexing
tree-sitter 多语言 parser
大规模代码搜索
本地 daemon
```

但你的 V1 更需要：

```
CI 集成
进程编排
报告生成
配置管理
企业离线部署
快速产品迭代
```

Go 更合适。

Rust 后面可以做：

```
generic syntax indexer
high-performance diff mapper
tree-sitter analyzer runtime
```

但不是 V1 必需。

---

# 为什么 Core 不选 Python

Python 适合原型，不适合这个产品的核心。

问题：

```
依赖部署麻烦
企业内网 pip 镜像麻烦
运行环境不可控
性能一般
长期维护和审计压力更大
```

可以用 Python 做实验性 analyzer，但不要做 production core。

---

# Java Analyzer 具体怎么做

Java Analyzer 是 V1 的第一批 analyzer。

它应该是 JVM 实现，但只负责输出 facts。

## 内部组件

```
Java Analyzer
├─ Build Model Resolver
├─ Source Parser
├─ Type Resolver
├─ Spring Analyzer
├─ Bytecode Scanner
├─ Coverage Mapper
├─ Mutation Result Mapper
└─ Fact Bundle Exporter
```

---

## Build Model Resolver

负责理解：

```
Maven multi-module
Gradle multi-module
source sets
test source sets
generated source
classpath
target/classes
```

Java 项目不理解 build model，后面所有分析都会漂。

---

## Source Parser

负责：

```
class / method / field
annotation
method range
signature
visibility
import
throws
return type
```

建议：

```
OpenRewrite / Spoon / JavaParser 选一个主工具
```

我的偏向：

```
OpenRewrite 优先
Spoon 备选
JavaParser 适合轻量 fallback
```

---

## Spring Analyzer

负责提取 Spring 语义 facts：

```
Controller endpoint
Request mapping
Security annotation
Transaction annotation
Scheduled job
Async method
Configuration bean
Repository
Service
Component
```

这些 facts 统一输出给 Core。

---

## Bytecode Scanner

负责补足源码分析不足：

```
compiled class
actual annotation
dependency edge
package dependency
Lombok / MapStruct 后的结果
```

工具方向：

```
ClassGraph / ASM
```

---

## Coverage Mapper

负责：

```
JaCoCo XML → changed line coverage → changed method coverage
```

它输出的是统一 Evidence Fact。

---

## Mutation Mapper

负责：

```
PIT report → mutation facts
```

不要让 Core 直接理解 PIT 原始报告。

Java Analyzer 或 Evidence Importer 把它归一化即可。

---

# Core 具体负责什么

Core 绝不能懂太多 Java 细节。

它只懂统一 facts。

## Core 组件

```
CodeGate Core
├─ Workspace Loader
├─ Git Diff Engine
├─ Analyzer Manager
├─ Fact Store
├─ Evidence Importer
├─ Baseline Manager
├─ Rule Evaluator
├─ Decision Engine
├─ Waiver Manager
└─ Report Adapter
```

---

## Analyzer Manager

负责：

```
识别 repo 语言
选择 analyzer
启动 analyzer
传入 diff
收集 fact bundle
处理 timeout
处理 analyzer crash
缓存结果
```

Analyzer crash 不能让结果变成 allow。

应该是：

```
required analyzer failed → require evidence / block
optional analyzer failed → warn
```

---

## Fact Store

V1 可以不用数据库。

本地存：

```
.codegate/cache/
.codegate/facts/
.codegate/baseline/
.codegate/reports/
```

后面企业版再把同样 facts 上传内部服务。

---

## Rule Evaluator

Rule Evaluator 不关心语言。

它只看 facts。

例如：

```
changed symbol has public contract change
changed symbol has security boundary change
changed-line coverage below threshold
mutation evidence missing
```

注意：这里不是在讲规则方向，而是讲实现边界。

Rule Evaluator 的输入必须是标准 facts，不应该直接读 Java AST。

---

# 多语言扩展方式

未来新增 TypeScript 时，不重写 core。

只做：

```
typescript-analyzer
```

它输出同一套 facts：

```
Symbol
SemanticChange
DependencyEdge
EntryPoint
Evidence
```

Core 不变。

未来新增 Python，也一样：

```
python-analyzer → Fact Bundle → Core
```

新增语言的成本应该是：

```
新增 analyzer + mapping
```

不是：

```
重写 rule engine + report + baseline + CI adapter
```

---

# 可以做一个 Generic Analyzer

为了避免每种语言一开始都要深度实现，可以内置一个通用 analyzer。

## Generic Analyzer 能做什么

基于：

```
git diff
file path
Tree-sitter syntax tree
basic symbol ranges
import graph
test file detection
SARIF import
coverage import，若存在
```

它能提供浅层 facts：

```
changed functions
changed classes
changed files
basic imports
large diff
test changed
public-ish symbol changed
```

这个 analyzer 可以覆盖很多语言的 60 分能力。

## Deep Analyzer 做什么

Java Analyzer 是 deep analyzer。

它提供：

```
type attribution
framework semantics
Spring facts
bytecode facts
coverage method mapping
mutation mapping
```

所以长期架构是：

```
Generic Analyzer：所有语言都有基础能力
Deep Analyzer：重点语言逐步增强
```

这可以避免每支持一个语言都从零开始。

---

# 一套实现的真实含义

这里要避免一个误区。

“一套实现”不应该理解成：

```
所有语言都用同一个 parser
所有 analyzer 都用同一种编程语言
所有语义都用一套代码硬解析
```

那不现实。

更合理的一套实现是：

```
一套 Core
一套 Fact Model
一套 Analyzer Protocol
一套 Rule Engine
一套 Baseline
一套 Report
一套 CI Gate
```

语言差异被限制在：

```
Analyzer
```

这才可维护。

---

# 第一版我会这样落地

## V1 目标

```
金融企业 Java brownfield app
```

## V1 实现

```
Core:
- Go CLI
- analyzer manager
- fact model
- baseline
- rule evaluator
- report output
- CI adapter

Java Analyzer:
- JVM/Kotlin
- Maven/Gradle awareness
- Java semantic facts
- Spring facts
- bytecode facts
- JaCoCo/PIT mapping

Packaging:
- codegate binary
- bundled java-analyzer.jar
- Docker image
- offline bundle
```

## V1 用户体验

用户只运行：

```
codegate check
```

内部流程：

```
Go core 读取 diff
Go core 调用 java-analyzer.jar
Java analyzer 输出 facts
Go core 收集 JaCoCo / PIT / SARIF evidence
Go core 做 baseline / rule / decision
Go core 输出报告和 CI status
```

---

# 这样未来新增语言不会推倒重来

## 新增 TypeScript

只加：

```
typescript-analyzer
```

复用：

```
Core
Fact Model
Rule Engine
Baseline
Report
CI
Evidence format
```

## 新增 Go

只加：

```
go-analyzer
```

复用同一套 core。

## 新增 Python

只加：

```
python-analyzer
```

复用同一套 core。

---

# 最终建议

我会把技术路线改成：

```
Core 用 Go
Analyzer 插件化
Java Analyzer 用 JVM
Fact Model 语言无关
Rule Engine 语言无关
Evidence 语言无关
Report 语言无关
```

一句话：

> **不要做 JVM 产品；做一个 Go core 的代码变更门禁系统，Java 只是第一套 deep analyzer。**

这比“全 JVM”更合理，也比“纯 Go/Rust 强行分析 Java”更现实。
