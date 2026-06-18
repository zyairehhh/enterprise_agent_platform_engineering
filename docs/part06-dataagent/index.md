# Part VI DataAgent 主线深潜

## 本部分章节

- [Ch.32 DataAgent 产品形态](ch32-dataagent.md) — 产品边界；四种形态；问题理解与任务规划
- [Ch.33 语义层工程](ch33.md) — 指标口径、Schema Linking、可信上下文
- [Ch.34 NL2SQL 工程化](ch34-nl2sql.md) — 分步生成与大库剪枝；安全执行与业务解释
- [Ch.35 Text-to-Pandas / Text-to-Python](ch35-text-to-pandas-text-to-python.md) — Python 沙箱；SQL + Python 协同
- [Ch.36 数据分析、可视化与报告](ch36.md) — 洞察、图表、报告；输出质量 Eval
- [Ch.37 DataAgent 对标与生态](ch37-dataagent.md) — 开源/商业对标；选型；Part VI 总结

**统一案例**：山岚集团华东区 GMV 下滑问数 → 分析 → 报告 Run 链（自 Ch.32 起贯穿六章）。

以下模块为 mini-platform 中 DataAgent 能力在 Part VI 各章的展开顺序。

## Part VI mini-platform 模块地图

| 模块路径 | 职责 | 主要章节 |
| --- | --- | --- |
| `agents/data_agent/` | AgentSpec、Question Frame | Ch.32–33 |
| `infra/semantic_layer/` | 指标、View、口径解析 | Ch.33 |
| `tools/sql_executor/` | 只读 SQL、校验与执行 | Ch.34 |
| `tools/python_sandbox/` | 分析沙箱 | Ch.35 |
| `tools/chart_renderer/` | 图表 spec | Ch.36 |
| `agents/data_agent/templates/` | 报告模板 | Ch.36 |

平台 Run / Registry / HITL 底座见 **Part V**（`core/runtime/` 等；对照 [Ch.22–30](../part05-agent-capabilities/ch22-agent-runtime.md)）。

## Part VI 能力体系（一览）

| 章 | 一句话 |
| --- | --- |
| **32** | DataAgent ≠ NL2SQL；四种产品形态 |
| **33** | 口径与 Linking 地基 |
| **34** | 安全 SQL 与自修复 |
| **35** | 沙箱 Python 分析 |
| **36** | 洞察、图表、报告与输出 Eval |
| **37** | 生态对标、选型、持续改进 |
