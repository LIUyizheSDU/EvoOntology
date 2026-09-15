<p align="center">
  <img src="assets/logo.png" alt="EvoOntology" width="68%">
</p>

<h1 align="center">EvoOntology：面向 Data Agent 的自进化 Ontology Layer</h1>

> **作者：** [Meiduo Chong](https://github.com/MeiduoChong)<sup>1</sup>、[Shaolei Zhang](https://zhangshaolei1998.github.io/)<sup>1*</sup>、[Ju Fan](https://iir.ruc.edu.cn/~fanj/)<sup>1</sup>、[Xiaoyong Du](https://info.ruc.edu.cn/jsky/szdw/ajxjgcx/jsjkxyjsx1/js2/7374b0a3f58045fc9543703ccea2eb9c.htm)<sup>1</sup><br>
> **机构：** <sup>1</sup> 中国人民大学（Renmin University of China）<br>
> **联系方式：** zhongmeiduo210@ruc.edu.cn、zhangshaolei98@ruc.edu.cn

<p align="center"><strong>面向 Data Agent 的 Agent-first、自进化 Ontology Layer。</strong></p>

<p align="center"><a href="README.md">English</a> | <a href="README.zh-CN.md">简体中文</a></p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="MIT License"></a>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.10%2B-blue.svg" alt="Python 3.10+"></a>
  <a href="https://modelcontextprotocol.io/"><img src="https://img.shields.io/badge/MCP-compatible-7c3aed.svg" alt="MCP compatible"></a>
</p>

EvoOntology 用于弥合异构表格、文件和数据库上的 **agent-data gap**。它通过 MCP 工具提供版本化的 **Ontology Layer**，基于真实 workload 和数据证据完成构建，并根据实际执行轨迹持续适配。

## 为什么需要 EvoOntology

- **原始数据缺少显式语义。** 表名、字段、文件路径和零散观测通常无法完整说明指标口径、实体关系与业务约束，Agent 需要反复推断，容易产生语义错误。
- **静态语义层难以持续扩展。** 人工构建和维护依赖专家投入；数据与 workload 变化后容易过时，完整注入 prompt 的方式也会带来不断增长的上下文开销。
- **Agent 需要能够持续适配的语义。** EvoOntology 围绕真实 workload 构建 Ontology Layer，由 Agent 按需查询，并依据执行行为在受控评估下持续进化。

<p align="center">
  <img src="assets/evoontology-overview.png" alt="使用与不使用 EvoOntology 的 Data Agent 对比" width="92%">
</p>

## 核心亮点

### 我们解决的问题

- **降低语义不确定性。** 显式组织领域概念、数据映射、关系与约束，避免 Agent 仅凭原始数据猜测业务含义。
- **减少重复的数据探索。** 在任务间复用经过数据验证的知识，使 Agent 能聚焦相关数据，而不必每次重新理解整个环境。
- **降低语义维护成本。** 根据 workload 和 Agent 行为持续适配 Ontology Layer，同时保证更新可检查、可比较、可回滚。

### 设计亮点

- **主动、按需访问。** Agent 通过 MCP 工具仅获取当前步骤所需的语义，无需在每次请求中注入完整 Ontology Layer。
- **基于证据构建。** 初始 ontology 围绕 workload 构建，并通过底层数据源中的实际观测完成验证。
- **基于轨迹进化。** 从历史交互中识别不足，并在相互关联的 Content、Schema 和 Tool Layer 中进行局部更新。
- **受控版本演进。** Candidate 只有在配对评估中可复现地优于 Parent，才会发布为下一 ontology 版本。
- **直接接入 Agent。** 安装插件并重启会话后，即可将 ontology workspace 和 MCP runtime 接入受支持的 Agent。

## Demo

Formula 1 演示展示了同一个 Data Agent 在构建和进化 Ontology Layer 前后的完整过程：baseline 执行、构建 `ontology_v0`、交互式查看、Candidate 评估、版本对比，以及使用优化后 `ontology_v1` 的结果。

https://github.com/user-attachments/assets/98ae3e38-73d0-4a1a-9ac4-d140ad09bfbb

## 工作原理

EvoOntology 把 Ontology Layer 视为可训练的 Agent 状态，而不是模型权重。Builder 根据 workload 和底层数据构建有证据支撑的语义对象；Evolution Agent 再利用历史交互提出局部更新，并将每个 Candidate 与 Parent 配对评估。

<p align="center">
  <img src="assets/evoontology-framework.png" alt="EvoOntology 构建与进化框架" width="100%">
</p>

### Ontology Layer

Ontology Layer 由三个相互关联的层组成，分别定义语义知识、表示规则和运行时访问方式：

| 层 | 作用 |
| --- | --- |
| **Content Layer** | 类型化语义图，包含 Term、Mapping、Constraint 和 Evidence 四类节点。Semantic Relation 连接 Term；Structural Reference 连接 Term 与 Mapping，并将 Constraint 或 Evidence 挂接到其约束或支撑的对象上。 |
| **Schema Layer** | 定义四类节点的字段、允许使用的 Semantic Relation 类型，以及合法的 Structural Reference 模式，从而确定 Ontology Layer 的表达边界。 |
| **Tool Layer** | 通过 `browse_semantics`、`resolve_semantics` 和简洁的 session manifest 向 Agent 暴露 Ontology Layer。会话初始化时只注入 manifest；具体记录及其关联对象均按需检索。 |

### 生命周期

1. **Build** — 从 workload 提取候选概念，在原始数据源中验证，并发布 `ontology_v0`。
2. **Use** — Data Agent 按需查询 Ontology Layer，同时记录工具交互和任务结果。
3. **Evolve** — 诊断重复出现的行为，将问题归因到 Content、Tool 或 Schema，并生成局部 Candidate 补丁。
4. **Evaluate** — 在相同数据、Agent、解码配置和交互预算下比较 Parent 与 Candidate。
5. **Publish or reject** — 通过门控的 Candidate 发布为 `ontology_vN+1`；否则保留 Parent，并把结果用于下一轮。

## 快速开始

直接从 GitHub Marketplace 安装插件，无需 clone 仓库、创建虚拟环境或单独执行 `pip install`。

### Claude Code

```bash
claude plugin marketplace add MeiduoChong/EvoOntology
claude plugin install evoontology@evoontology
claude plugin list
```

新建会话后运行：

```text
/evo-build
/evo-evolve
/evo-visualize
```

### Codex

```bash
codex plugin marketplace add MeiduoChong/EvoOntology
codex plugin add evoontology-codex@evoontology
codex plugin list
```

新建 thread 后，让 Codex 使用：

```text
$evo-build
$evo-evolve
$evo-visualize
```

构建完成后，Data Agent 可以直接调用 `browse_semantics` 和 `resolve_semantics`，无需额外配置 Ontology Layer。完整流程和数据边界请见[使用指南](USAGE.md)。

## 使用模式

| 模式 | 适用场景 | 评估边界 |
| --- | --- | --- |
| `fixed_split` | 具有固定问题集和 Ground Truth 的 benchmark | Construction 数据用于 Build 和诊断；Validation Reserve 只用于最终 gate。 |
| `rolling_trajectory` | 没有固定测试集的生产 workload 或冷启动项目 | 每个 checkpoint 后持续积累新轨迹，再用独立抽样任务或 LLM Judge 对 Candidate 进行门控。 |

两种模式共用同一套 `.evoontology/` workspace、`ontology_vN` 版本、checkpoint 和 Parent/Candidate 生命周期。

## 支持的 Benchmark

EvoOntology 包含三个互补的、自包含的 Data Agent 评估环境：

| Benchmark | 任务 | 目录 |
| --- | --- | --- |
| BIRD | 面向真实数据库的 text-to-SQL | [`benchmarks/bird/`](benchmarks/bird/) |
| DDR-10K | 面向异构金融数据的开放式研究 | [`benchmarks/ddr_10k/`](benchmarks/ddr_10k/) |
| InsightBench | 迭代式业务分析和洞察生成 | [`benchmarks/insightbench/`](benchmarks/insightbench/) |

每个环境都实现 `EvolutionAdapter`，并保留原生 rollout 和评估协议。运行 `python -m benchmarks list` 可查看已注册环境；接入新环境请见[新增 Benchmark](docs/guide/new-benchmark.md)。

## 仓库结构

| 路径 | 职责 |
| --- | --- |
| [`evoontology/`](evoontology/) | 确定性核心：ontology store、runtime/MCP、trajectory、trigger、evaluation、evolution state、validation 和 visualization。 |
| [`plugins/`](plugins/) | 自包含的 Claude Code 与 Codex 插件，包括 Build、Evolve 和 Visualize skills。 |
| [`benchmarks/`](benchmarks/) | BIRD、DDR-10K 和 InsightBench 评估环境。 |
| [`docs/`](docs/) | 架构与 benchmark 接入文档。 |
| [`scripts/`](scripts/) | 将 core 同步到插件的工具。 |

## 文档

- [使用指南](USAGE.md) — 安装、workspace、生命周期、配置和端到端流程。
- [架构说明](docs/architecture.md) — 模块边界、进化状态机和评估模式。
- [新增 Benchmark](docs/guide/new-benchmark.md) — adapter、data loader、rollout、配置和 seed skill 契约。
- [Claude Code 插件](plugins/claude-code/README.md)与 [Codex 插件](plugins/evoontology-codex/README.md) — 各客户端的安装与使用方式。

## License

本项目使用 [MIT License](LICENSE)。Copyright © Meiduo Chong。
