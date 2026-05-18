# CodeGate 技术实现方案

## 实现结论

CodeGate 不应该做成 JVM-only 产品，也不应该用 Go / Rust / Python 强行直接深度分析 Java。

推荐实现：

```text
Core: Go
Java Analyzer: JVM / Kotlin / Java
Protocol: JSON first, Protobuf later
Rule Engine: Go typed evaluator
Packaging: Go binary + bundled java-analyzer.jar + Docker image + offline bundle
```

这个方案把语言相关复杂度限制在 analyzer，把产品核心保持语言无关。

## 架构边界

```text
codegate
├─ Core Runner
│  ├─ workspace loader
│  ├─ git diff engine
│  ├─ analyzer manager
│  ├─ fact store
│  ├─ evidence importer
│  ├─ baseline manager
│  ├─ rule evaluator
│  ├─ waiver manager
│  └─ report adapter
│
└─ Analyzers
   ├─ java-analyzer
   ├─ generic-analyzer
   └─ future analyzers
```

Core 只理解统一 facts，不直接读取 Java AST，也不直接理解 Spring 细节。

Java Analyzer 只生产 facts，不做最终 allow / block。

## 为什么 Core 用 Go

Go 适合 CodeGate 的产品核心：

- 单 binary 分发，适合 CI 和内网环境
- 进程编排简单，适合调用 analyzer、Maven、Gradle、PIT、Semgrep
- 配置、报告、文件处理、并发和超时控制成熟
- 团队维护门槛低于 Rust
- 不绑定 JVM，便于后续多语言 analyzer 扩展

Go core 负责产品稳定面：

- diff
- analyzer orchestration
- fact bundle ingestion
- baseline
- rule evaluation
- decision
- report
- CI exit code
- audit metadata

## 为什么 Java Analyzer 用 JVM

Java 深度分析应使用 JVM 生态，因为 V1 目标是 Java 金融 brownfield 项目。

Java analyzer 需要理解：

- Maven / Gradle 多模块
- classpath
- Spring annotations
- bytecode
- Lombok / MapStruct 等生成影响
- JaCoCo XML
- PIT report
- JUnit report
- SARIF / Semgrep / Sonar 等外部报告

这些能力用 JVM 实现更自然。Java analyzer 可以用 Kotlin / Java 编写，对外只暴露 analyzer protocol。

## Analyzer Protocol

Analyzer 是独立 worker，由 Core 启动。

V1 使用 subprocess + JSON / NDJSON，避免过早引入 gRPC。

### 输入

```json
{
  "workspace": "/path/to/repo",
  "baseCommit": "base",
  "headCommit": "head",
  "changedFiles": [],
  "changedRanges": [],
  "configPath": "codegate.yml",
  "cacheDir": ".codegate/cache"
}
```

### 输出

```json
{
  "analyzer": {
    "name": "java-analyzer",
    "version": "0.1.0",
    "language": "java",
    "capabilities": [
      "symbols",
      "type_attribution",
      "spring_entrypoints",
      "bytecode_scan",
      "coverage_mapping",
      "mutation_mapping"
    ]
  },
  "facts": []
}
```

Analyzer crash 或 capability 缺失不能被当作通过。Core 应根据规则需要把它转成 `REQUIRE_EVIDENCE`、`WARN` 或 `BLOCK`。

## Fact Model

Fact Model 是 CodeGate 的长期核心。它必须语言无关。

### Change Fact

描述这次改了什么：

- repo
- base commit
- head commit
- changed files
- changed ranges
- added / modified / deleted / renamed
- generated files
- test files changed

### Symbol Fact

描述代码符号：

- language
- module
- file
- kind
- qualified name
- signature
- visibility
- range
- annotations / decorators / attributes

### Semantic Change Fact

描述语义变化：

- signature changed
- annotation changed
- visibility changed
- return type changed
- dependency changed
- entrypoint changed
- security boundary changed
- transaction boundary changed
- public contract changed

### Dependency Fact

描述依赖关系：

- from symbol
- to symbol
- edge type
- source location

### Entry Point Fact

描述外部入口：

- HTTP endpoint
- message consumer
- scheduled job
- CLI command
- public API
- event handler

### Evidence Fact

描述证据：

- test result
- changed-line coverage
- mutation result
- static finding
- contract diff
- snapshot diff
- dependency scan
- approval
- waiver

### Decision Fact

由 Core 生成：

- decision status
- failed rules
- missing evidence
- required actions
- audit metadata

## Java Analyzer 设计

```text
Java Analyzer
├─ build model resolver
├─ source parser
├─ type resolver
├─ spring analyzer
├─ bytecode scanner
├─ coverage mapper
├─ mutation mapper
└─ fact bundle exporter
```

### Build Model Resolver

支持：

- Maven multi-module
- Gradle multi-module
- source sets
- test source sets
- generated source
- classpath
- target / build output

Java 项目不理解 build model，后续分析都会漂。

### Source Parser

提取：

- class / method / field
- annotation
- method range
- signature
- visibility
- import
- throws
- return type

候选技术：

- OpenRewrite LST / type attribution
- Spoon
- JavaParser + Symbol Solver

V1 建议 OpenRewrite 优先，Spoon 或 JavaParser 作为 fallback。

### Spring Analyzer

提取 Spring 语义 facts：

- Controller endpoint
- request mapping
- security annotation
- transaction annotation
- scheduled job
- async method
- configuration bean
- repository / service / component

### Bytecode Scanner

补足源码分析不足：

- compiled class
- actual annotation
- class dependency
- method signature
- Lombok / MapStruct 生成结果
- package dependency

候选技术：

- ClassGraph
- ASM

### Coverage Mapper

把 JaCoCo XML 映射到 changed lines / changed methods：

```text
git diff changed lines
→ repo-relative file path
→ JaCoCo XML package/class/sourcefile
→ line hit/miss
→ method/class/domain aggregation
```

### Mutation Mapper

读取 PIT report，归一化为 mutation evidence：

- class
- method
- line
- mutator
- killed / survived
- related changed line
- related sensitive domain

## Evidence Importer

V1 应该 consume-first，不要 replace-first。

优先导入已有报告：

- JaCoCo XML
- PIT report
- JUnit XML
- Maven Surefire / Failsafe reports
- Gradle test reports
- SARIF
- Semgrep JSON / SARIF
- Sonar / Fortify / Checkmarx 可导出的企业报告

CodeGate 不替代这些工具，而是把结果标准化成 evidence facts。

## Rule Evaluator

V1 不做 OPA-first。

Rule flow：

```text
YAML / JSON config
→ parse
→ typed Rule IR
→ evaluate against Fact Model
→ Decision
```

Rule Engine 内部应强类型建模：

- trigger
- scope
- evidence requirement
- threshold
- decision action
- explanation template
- remediation template
- waiver policy
- severity

避免让用户直接写任意脚本或复杂表达式。这样更可审计、可解释、可迁移。

OPA / Rego 可以作为未来 enterprise advanced policy backend，但不是 V1 core。

## Baseline Manager

Baseline 是 brownfield 能否落地的关键。

V1 存储：

```text
.codegate/
  baseline/
  cache/
  facts/
  reports/
```

Baseline fingerprint 不应只靠行号，应组合：

- repo
- module
- rule id
- class
- method signature
- normalized location
- code snippet hash

判断策略：

- existing finding: ignore for current PR
- new finding: evaluate and possibly block
- worsened evidence: evaluate and possibly block
- removed finding: mark resolved

## Report Adapter

V1 输出：

- machine-readable JSON
- human-readable HTML
- CI summary markdown
- JUnit XML
- SARIF

每个 decision 都必须能回溯：

```text
decision → failed rule → fact → source report / analyzer → file / line / symbol → tool version / commit
```

金融企业场景里，报告的可审计性比 dashboard 更重要。

## Packaging

V1：

- Go binary
- bundled `java-analyzer.jar`
- Docker image
- offline bundle
- CI templates

V1 不追求单一 native artifact 覆盖所有内容。Java analyzer 需要 JVM 是合理成本。

## 多语言扩展

新增语言时，只新增 analyzer：

```text
typescript-analyzer → Fact Bundle → Core
python-analyzer     → Fact Bundle → Core
go-analyzer         → Fact Bundle → Core
```

Core 不重写：

- rule evaluator
- baseline
- decision
- report
- CI adapter
- evidence schema

## Generic Analyzer

为了降低新增语言成本，可以提供 generic analyzer：

- git diff
- file path classification
- Tree-sitter syntax ranges
- basic symbols
- import graph
- test file detection
- SARIF import
- coverage import if available

Generic analyzer 提供浅层 facts；重点语言再做 deep analyzer。

## V1 成功标准

V1 不需要发现所有代码问题。它要证明：

> 在 Java 金融 brownfield 项目中，高风险变更不能在缺少证据的情况下悄悄进入主干。

Demo 场景：

- AI 修改退款金额边界，测试绿，但 mutation survived，CodeGate 阻断
- AI 删除 `@PreAuthorize`，测试绿，CodeGate 阻断
- AI 改 `@Transactional` propagation，CodeGate 要求审批或证据
- AI 改 public API DTO，CodeGate 要求 contract / snapshot evidence
- AI 修改 destructive migration，CodeGate 阻断并要求 rollback plan
