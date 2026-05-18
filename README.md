# CodeGate

CodeGate 是一个面向 brownfield codebase 的代码变更门禁系统。它的目标不是判断代码是不是 AI 写的，也不是再做一个 AI code review bot，而是在 AI coding 日益普及后，作为合并前的最后一道关卡，限制高风险变更的影响范围，并要求变更提供足够证据。

核心判断是：

> 测试绿灯不代表安全。CodeGate 检查的是：这次变更是否有足够证据证明它没有破坏已有业务行为。

## 产品定位

CodeGate 关注 PR / MR / commit 级别的变更风险。它从 diff、代码语义、测试覆盖、mutation 结果、静态扫描、baseline 和已有企业工具报告中提取事实，再用规则引擎做确定性决策。

它不是：

- AI code review
- SAST / SonarQube 替代品
- AI 代码识别器
- 企业治理大屏
- 自动修复工具

它是：

- brownfield 项目的变更风险门禁
- AI-assisted change 的破坏半径控制系统
- 基于 facts + rules + evidence 的合并前判断
- 可离线运行、可审计、可逐步增强的 CI gate

## V1 范围

第一版面向 Java 金融企业 brownfield 应用，重点支持：

- Java / Spring 项目变更分析
- Maven / Gradle workspace
- Git diff 到 changed class / method / line 的映射
- JaCoCo changed-line coverage
- PIT mutation evidence
- Spring Security annotation、transaction boundary、API entrypoint 等 Java/Spring 语义 facts
- SARIF / JUnit / 外部扫描报告导入
- baseline + ratchet，避免清算历史旧账
- JSON / HTML / Markdown / JUnit XML / SARIF 报告
- Jenkins / GitLab CI 等内网环境接入

V1 不做：

- 跨语言完整支持
- IDE 插件
- SaaS dashboard
- AI 自动判定
- 自动修复
- 全量业务理解
- 复杂审批流

## 技术方向

CodeGate 的核心实现应语言无关。Java 是第一套 deep analyzer，不是整个产品的技术边界。

推荐架构：

```text
CodeGate Core: Go
Java Analyzer: JVM / Kotlin / Java
Protocol: JSON first, Protobuf later
Rule Engine: Go typed evaluator
Report / Baseline / CI Gate: shared core
```

一套实现覆盖不同用户的方式不是拆成个人版、小团队版、企业版三套产品，而是保持同一个 core runner：

- 本地开发者运行 `codegate check`
- 小团队在 CI 中运行同一个 runner
- 企业在 Jenkins / GitLab / Bitbucket / 内网构建机运行同一个 runner，并保留审计报告

## 文档

- [产品设计](docs/product-design.md)
- [技术实现方案](docs/implementation.md)
- [原始讨论记录](docs/AI%20代码质量控制.md)
