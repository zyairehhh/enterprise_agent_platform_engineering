# Ch.37 DataAgent 对标与生态

> **本章目标**：读者学完能说明 DataAgent **生态分化的原因**，用能力矩阵对比 **DB-GPT、Vanna、WrenAI、Defog、Sherlock** 等开源方案与 ChatBI/BI Copilot，并在 **自研、采购、混合** 路线下完成 **企业选型清单**，理解 Part VI 六章如何组成 **可实施的 DataAgent 能力体系**。  
> **关键议题**：开源对标、商业 ChatBI、选型、Eval 与持续改进、Part VI 总结  
> **前置阅读**：[Ch.32–36](ch32-dataagent.md) 全章 · [Ch.31 框架对标](../part05-agent-capabilities/ch31.md) · [Ch.39 离线评估](../part07-observability-eval/ch39-dataagent-eval-benchmark.md)  
> **估计阅读**：约 70 min  
> **mini-platform 关联**：对照全书 `mini-platform/` 模块（本章无新增模块）  
> **按角色推荐阅读**：CTO / 采购 ⇒ 全章 ｜ 架构师 ⇒ §2–§5 ｜ 数据负责人 ⇒ §3 + §6

Part VI 前五章分别定义 **产品边界**（Ch.32）、**语义层**（Ch.33）、**NL2SQL**（Ch.34）、**Python 分析**（Ch.35）与 **表达与输出 Eval**（Ch.36）。读者若已跟随 [Ch.32 §4 华东下滑案例](ch32-dataagent.md) 走完一条 Run 链，会自然产生最后一个问题：**业界已有大量 DataAgent / Text-to-SQL 产品，山岚集团该自建、采购还是混合？**

答案是：**没有单一产品能替代 Part V 平台 + Part VI 六章的组合**——但企业也不必从零造每一个轮子。本章说明生态为何高度分化、主流方案各覆盖哪些能力、ChatBI 与 DataAgent 的边界在哪里，以及如何用 **Eval 与业务金标准集** 证明选型后的持续改进。

本章依次介绍：生态为何分化（§1）；开源与商业分类（§2）；主流开源对比（§3）；ChatBI、BI Copilot、DataAgent 差异（§4）；自研/采购/混合（§5）；评估与持续改进（§6）；企业选型清单与 **Part VI 总结**（§7）。

---

### DataAgent 生态为什么分化

为什么没有「一个标准 DataAgent 产品」能直接采购上线？

根本原因在于：**DataAgent 要同时满足四类不同来源的需求**——对话入口、Text-to-SQL 技术、企业部署合规、组织级数据治理——而市场上多数产品只从其中 **一条轴线** 起步，其余能力靠集成或外接补齐。四类分化轴如下：

| 分化轴 | 典型产物 | 能力缺口 |
| --- | --- | --- |
| **入口** | ChatBI 对话框 | 缺平台治理（Runtime、Registry、Trace） |
| **技术路线** | Text-to-SQL 库（如 Vanna，见 §2 说明） | 缺语义层与 HITL |
| **部署** | SaaS Copilot | 缺私有化与多租户 |
| **组织** | 数据中台项目 | 缺 Agent Runtime 与 Run 六态 |

以山岚为例：运营总监一句「上周华东 GMV 下滑 Top SKU」，背后至少需要 **Question Frame 解析**（Ch.32）、**Metric 绑定与消歧**（Ch.33）、**只读 SQL 执行**（Ch.34）、**品类贡献度 Python**（Ch.35）、**图表与报告审批**（Ch.36）——以及贯穿全程的 **Run 审计与 Eval**（Ch.37、Ch.39）。采购一个「对话查数」SaaS，通常只覆盖入口与 NL2SQL 演示；**语义层口径、沙箱分析、HITL 发布** 仍须企业自建或二次集成。

LLM/Agent-as-Data-Analyst 综述将分析 Agent 所需能力归纳为 **语义感知、工具链编排、自主流水线** 等多维组合 [2]——**极少有单一产品一次覆盖**。企业落地的常见形态是：**Part V 平台（Ch.22–30）+ 语义层（Ch.33）+ 专用 Tool（Ch.34–36）**，通过 Registry 统一审计，而非指望一个 ChatBI 包打天下。

公开 benchmark 也在推动这一认知转变。**Spider 2.0** [3] 与 **BIRD-INTERACT** [4] 把评测从「单句翻译 SQL」推向 **企业 workflow、多轮澄清与交互式纠错**——这与 Ch.32 定义的 **诊断 + 对比 + 报告** 全链路一致（Spider/BIRD 含义见 [Ch.32 §1](ch32-dataagent.md)）。产品若仍停留在「把自然语言变成一条 SELECT」，在华东下滑这类 **多步分析 Run** 上必然力不从心。

#### 常见误区

**误区 1：部署一个 ChatBI 就等于部署了 DataAgent 平台。**  
ChatBI 往往是 **问数形态的子集**（Ch.32 §2），缺少 `waiting_human` 审批链、Handoff 与跨 Agent 编排；山岚月报 Run 链仍须 Part V Runtime。

**误区 2：引入 DB-GPT 就无需 Part V。**  
**DB-GPT** [5] 是开源 **Agent 应用框架**（自带 Runtime 壳与数据插件）；若企业已建设 `core/runtime/` 与 `core/registry/`，再整包引入会形成 **双 Runtime**（见 Ch.31 常见误区）——宜 **接组件、不接平台壳**。

**误区 3：NL2SQL demo 准确率够高即可上线。**  
华东案例在 Linking 阶段就存在 `gmv_tax_excluded` 与 `gmv_ops` 歧义（Ch.33 §4）；无 **业务金标准集** 与 **口径脚注 Eval**，上线后口径投诉率会掩盖 SQL 语法正确率。

---

### 开源框架与商业产品分类

下表列出的 **Vanna、WrenAI、DB-GPT** 等均为 **GitHub 开源项目或商业 Copilot 产品名**——不是本书 invented 的术语。读表前先记住各名 **一句话定位**：

| 名称 | 一句话 | 典型用途 |
| --- | --- | --- |
| **Vanna** [7] | 用向量库检索 **历史 Question-SQL**，RAG 增强生成 | 快速 POC、私有 schema 适配 |
| **WrenAI** [6] | **语义层 + 对话式 BI** | Metric 建模与问数一体 |
| **DB-GPT** [5] | **Agent 应用框架** + 数据插件模板 | 从零搭数据 App（易与 Part V 双 Runtime） |
| **Defog** [8] | 偏 **Text-to-Python 与自动报告** | 分析 + 叙事 |
| **Sherlock** [9] | **研究型**深度分析 Agent 原型 | 推理链参考，少企业治理 |
| **Power BI Copilot** [10] | BI 产品 **内置 Copilot** | 改图表/筛选，非平台化 DataAgent |

#### 生态地图

按 **偏库/偏平台** 与 **偏 NL2SQL/偏全链路** 两个维度，主流方案可粗略定位如下：

| 产品 | 偏库/SDK ↔ 偏平台 | 偏 NL2SQL ↔ 偏全链路 | 定位简述 |
| --- | --- | --- | --- |
| Vanna | 偏库 | 偏 NL2SQL | 向量检索历史 SQL + schema（RAG） |
| Sherlock | 偏库 | 偏全链路 | 研究型深度分析 |
| WrenAI | 中间 | 中间 | 语义层 + 对话式 BI |
| Defog | 中间 | 偏全链路 | Python 分析与报告 |
| DB-GPT | 偏平台 | 偏全链路 | Agent 应用框架 |
| Power BI Copilot | 偏平台 | 偏 NL2SQL | BI 内嵌 Copilot |
| mini-platform | 偏平台 | 全链路 | 与 Part V 同源参照实现 |

| 类别 | 代表 | 典型强项 | 与 mini-platform 的关系 |
| --- | --- | --- | --- |
| **开源平台型** | DB-GPT [5] | Agent 框架 + 数据 App 模板 | 可对标 `core/runtime/` + 数据插件，不宜重复造 Runtime |
| **语义层 + NL2SQL** | WrenAI [6] | Metric 建模 + 对话式 BI | 可对标 `infra/semantic_layer/`，经 Registry HTTP 代理接入 |
| **Question-SQL 检索增强** | Vanna [7] | 向量检索 **历史 Question-SQL** 与 schema 片段 | 可包装为 `tools/sql_executor/` 后端训练管线 |
| **Python 分析 Agent** | Defog [8] | Text-to-Python、报告 | 可对标 `tools/python_sandbox/` + `tools/chart_renderer/` |
| **研究/Deep Agent** | Sherlock [9] | 复杂推理实验 | 仅作 Planner 推理链参考，无企业 Registry |
| **商业 BI Copilot** | Power BI Copilot [10] | 嵌入 BI、低门槛 | 与 DataAgent **并存**，口径经语义层对齐 |
| **本书参照** | mini-platform | 与 Part V 平台同源 | Part VI 各章模块路径见 §7 |

---

### 主流开源方案对比（DB-GPT、Vanna、WrenAI、Defog、Sherlock）

§2 已说明各开源项目 **是什么**。本节回答：**它们分别覆盖 Part VI 哪几章的能力？** 与 mini-platform 模块如何对照？

| 能力 / 章节 | DB-GPT | Vanna | WrenAI | Defog | Sherlock | mini-platform（书中模块） |
| --- | --- | --- | --- | --- | --- | --- |
| Agent Runtime (Ch.22) | 自有 | 无 | 部分 | 部分 | 实验 | ✓ `core/runtime/` |
| Tool Registry (Ch.23) | 部分 | 无 | 部分 | 部分 | 无 | ✓ `core/registry/` |
| 语义层 (Ch.33) | 可接 | 弱 | **强** | 中 | 弱 | `infra/semantic_layer/` · `agents/data_agent/linker.py` |
| NL2SQL (Ch.34) | ✓ | **强** | ✓ | 中 | 中 | `tools/sql_executor/` |
| Python 沙箱 (Ch.35) | ✓ | 弱 | 弱 | **强** | 中 | `tools/python_sandbox/` |
| 报告/图表 (Ch.36) | 部分 | 弱 | 中 | **强** | 中 | `tools/chart_renderer/` · `agents/data_agent/templates/` |
| HITL / 多租户 | 弱 | 弱 | 中 | 中 | 弱 | ✓ Part V Run 链 · `core/policy/` |
| 企业 Eval (Ch.36–39) | 部分 | 弱 | 中 | 中 | 研究 | `core/eval/` · Ch.39 |

!!! note "mini-platform 落地状态"
    表中 **Part V** 模块（`core/runtime/`、`core/registry/` 等）与 `mini-platform/projects/multi-agent-workflow/` **已在仓库中存在**。  
    **Part VI** 列（`tools/sql_executor/`、`tools/python_sandbox/`、`infra/semantic_layer/client.py` 等）为 **书中目标契约**，随 Part VI 工程迭代合入；选型评估请以概念验证为准，勿假定仓库已包含全部目录。

*能力评分为方向性对照，非版本打分（2025-06 核对）。*

#### 各方案选型要点

**Vanna** [7] 以 **向量检索历史 SQL + 库表 schema 片段（RAG，检索增强生成）** 见长，适合 **POC 与私有 schema 快速适配**。华东案例若只用 Vanna，可较快生成 Top SKU 查询，但 **GMV 歧义消歧**（`gmv_ops` vs `gmv_tax_excluded`）与 **View 级权限** 须外接 `infra/semantic_layer/` 与 `core/policy/`，否则难进生产。

**WrenAI** [6] 强调 **语义层 + 对话式 BI（GenBI，即自然语言驱动的图表/问数）**，与 Ch.33 路线最接近——山岚的 `sales_ops` View 与 Metric 版本策略可直接类比。WrenAI 仍须接企业 `core/runtime/` 与 Ch.30 HITL；多 Agent 治理不宜与 Part V 双轨并行。

**DB-GPT** [5] 提供 **Agent 应用壳与数据插件**。若企业已有 Ch.22–30 平台，更稳妥的做法是：**NL2SQL 训练或插件逻辑经 Registry 注册为 Tool**，而非引入第二套 Runtime（与 [Ch.31](../part05-agent-capabilities/ch31.md) 框架对标结论一致）。

**Defog** [8] 偏 **Text-to-Python 与自动报告**，与 Ch.35–36 的 `python_sandbox` + `chart_renderer` 组合高度重叠。山岚华东案例的 **品类贡献度** 步骤可对标 Defog 强项；取数仍应走 `sql_executor` 只读链路。

**Sherlock** [9] 属 **研究型深度分析 Agent** 原型，在复杂推理链设计上有参考价值，但 **多数缺少企业级 Runtime、行级权限与 Eval 流水线**。不建议整包替换 Part V 平台；Planner 多步推理策略可借鉴，工程落地仍回 `core/planner/`。

!!! note "对标不等于采购建议"
    上表随社区版本变化；选型时须 **概念验证** 与 **安全审计**，本章仅提供 **能力映射** 与 mini-platform 模块对照。

---

### ChatBI、BI Copilot、DataAgent 的产品差异

三类产品名称相近，职责边界不同：

| 维度 | ChatBI | BI Copilot [10] | DataAgent（本书） |
| --- | --- | --- | --- |
| 定位 | 对话查数 | BI 内嵌助手 | 平台托管数据任务 Agent |
| 语义层 | 不定 | 依赖 BI 数据集 | **强制** Ch.33 · `infra/semantic_layer/` |
| 多步分析 | 弱 | 中 | Planner 链 Ch.34–36 · `sql_executor` → `python_sandbox` → `chart_renderer` |
| 审批 | 通常不支持 | 通常不支持 | HITL Ch.30 · 报告级 `waiting_human` |
| 与 ERP/Agent 编排 | 弱 | 弱 | Handoff Ch.28 · `agents/data_agent/` |
| 评测 | 依厂商 | 依厂商 | Spider 2.0 / BIRD-INTERACT + 业务金标准集 · `core/eval/` |

**ChatBI** 适合「单轮问数、用户规模小、合规要求低」的场景；一旦需要 **多轮澄清、Python 分析、报告审批与 Run 审计**，即进入 DataAgent 范畴（Ch.32 四种产品形态）。

**BI Copilot** 降低已有 BI 用户的操作门槛，但 **口径绑定在 BI 数据集内**，难以成为集团级 **Metric 权威源**。山岚策略是：**Tableau Copilot 做分析师辅助**（库存、固定看板）；**DataAgent 做经营问数与月报 Run 链**（华东下滑诊断、Controller 审批发布）——二者 **并存**，口径 **统一到语义层** `infra/semantic_layer/models/`（Ch.33 山岚样例），避免 Copilot 与 Agent 各说各的 GMV。

---

### 自研、采购与混合路线

#### 四条路线与适用场景

| 路线 | 适用 | 风险 |
| --- | --- | --- |
| **采购 SaaS ChatBI** | 要快、用户少、可接受数据出境 | 口径不可控、难接 HITL 与 Eval |
| **采购 + 自建语义层** | 有中台与 Cube/dbt 基础 | 两套平台集成成本高 |
| **混合：平台自研 + 组件** | 有 Part V 团队 ⭐ | 需架构纪律，禁止双 Runtime |
| **全自研** | 强合规、长期 ROI、定制深 | 初期交付慢 |

与 [Ch.31](../part05-agent-capabilities/ch31.md) 框架对标结论一致：**Runtime / Registry / Obs 宜自研或统一于 Part V**；**NL2SQL 可接 Vanna 训练管线**，包装为 `tools/sql_executor/` 的后端能力；**语义层可用 Cube 或 Wren 引擎**，由 `infra/semantic_layer/client.py` 统一 `resolve_metric()` 与 `compile_query()` 接口——外部组件 **经 Registry 的 HTTP 代理调用**（统一审计、统一 Trace），而不是业务代码直连第三方 SDK。

山岚属于 **混合路线**：Part V 与 DataAgent 应用（`agents/data_agent/`）自研；语义层基于 Cube 风格 YAML 托管在 `infra/semantic_layer/models/`；NL2SQL 可借鉴 Vanna 的 question-SQL 检索增强 `sql_executor` 生成阶段，但 **执行与 Policy 不外包**。

#### 设计取舍

| 决策 | 选 A | 选 B |
| --- | --- | --- |
| NL2SQL 引擎 | 自研 Planner + Gateway + `sql_executor` | Vanna 训练 + 包装为 Registry Tool |
| 语义层 | Cube / Wren 开源引擎 | 完全自研 YAML · `infra/semantic_layer/models/` |
| 前端 | 自研 Generative UI（Ch.48） | 嵌入 Power BI Copilot，口径回写语义层 |
| Python 分析 | `tools/python_sandbox/` 自研沙箱 | 参考 Defog 报告模板，沙箱仍自建 |

**采购决策的底线**：任何引入方案不得绕过 **`tenant_id` 注入、只读执行、`metric_id@version` 审计** 三件套（Ch.34 §5）；否则华东案例可在演示环境跑通，生产环境却无法通过安全评审。

---

### 评估与持续改进

选型回答「买什么」；Eval 回答「买/建之后有没有变好」。DataAgent 的 Eval 须 **公开 benchmark + 业务金标准集** 双轨并行——前者保证技术回归，后者保证 **口径与叙事** 贴合山岚真实问法。

#### 离线 Eval

公开集 **Spider 2.0**、**BIRD**、**BIRD-INTERACT** 用于技术回归；含义见 [Ch.32 §1](ch32-dataagent.md)。**山岚业务金标准集** 用于口径与叙事——二者不可互相替代。

| 层级 | 数据集 | 章节 | mini-platform |
| --- | --- | --- | --- |
| SQL 正确性 | BIRD、Spider 2.0 [3] | Ch.39 | `core/eval/` SQL 子集 |
| 多轮交互 | BIRD-INTERACT [4] | Ch.39 | 澄清 / ASK 场景回放 |
| 洞察与报告 | 山岚业务金标准集（≥50 条） | Ch.36 §6 | 口径脚注、EvidenceRef 覆盖率 |

山岚金标准集应包含华东下滑 **变体问法**（如「销售额」vs「GMV」、「华东」vs「苏皖大区」），每条标注期望 `metric_id@version` 与是否触发 HITL——Eval 失败样本直接回流 `infra/semantic_layer/` Glossary 与 Prompt 版本。

#### 在线指标

| 指标 | 用途 | 关联模块 |
| --- | --- | --- |
| 首问解决率 | 产品可用性 | Run Trace · Ch.38 |
| 口径投诉率 | 语义层质量 | `infra/semantic_layer/` 变更记录 |
| 审批通过率 | 报告 / HITL 质量 | Ch.30 · `agents/data_agent/templates/` |
| Run 成本 | 产能与模型选型 | Ch.41 |

**持续改进闭环**：Eval 失败样本 → 语义层 / Glossary 修订 → Prompt / Tool 版本 bump → 回归 [1]。与 Ch.31 的「框架对标后迭代」相同，DataAgent 迭代 **以业务样本为主、公开榜为辅**——Spider 2.0 高分但华东案例口径脚注缺失，仍视为发布阻塞项。

[Ch.39](../part07-observability-eval/ch39-dataagent-eval-benchmark.md) 与 **Ch.50** 提供 **平台级** Eval 与 Policy 自动化；Part VI 强调：**业务样本不可只用公开 benchmark 替代**。

---

### 企业选型清单与 Part VI 总结

#### 选型 checklist（CTO / 数据负责人）

**能力与架构**

- [ ] 是否 **强制语义层**，禁止 Agent 长期直连物理表？（`infra/semantic_layer/`）  
- [ ] 是否有 **Agent Runtime + Registry + Trace**（Part V），而非仅 Chat UI？（`core/runtime/` · `core/registry/`）  
- [ ] NL2SQL 是否 **只读 + tenant 注入 + 审计**？（`tools/sql_executor/` · `core/policy/`）  
- [ ] 复杂分析是否走 **沙箱 Python**，而非非常规 SQL 变通？（`tools/python_sandbox/`）  
- [ ] 对外报告是否有 **HITL + evidence**？（Ch.30 · `agents/data_agent/templates/`）

**Eval 与运营**

- [ ] 是否在 **Spider 2.0 / BIRD-INTERACT 类** 场景抽测 [3][4]？  
- [ ] 是否有 **业务金标准问数集**（≥50 条，含口径歧义变体）？  
- [ ] 是否跟踪 **采纳率、口径投诉与用户负面反馈**？（`core/eval/`）

**供应商 / 开源**

- [ ] 私有化与数据驻留是否满足合规？  
- [ ] 与现有 Cube / dbt / BI 集成成本？  
- [ ] 是否会造成 **与 Part V 双 Runtime**？（见 Ch.31）

#### 走读：山岚「华东下滑」案例贯穿 Part VI 六章

以下沿用 [Ch.32 §4](ch32-dataagent.md) 运营总监原话：「上周华东区销售相对前周明显下滑，主要 SKU 是哪些？和品类结构有没有关系？」

| 章 | 本步做什么（白话） | mini-platform 模块 |
| --- | --- | --- |
| **Ch.32** | 把原话解析成 Question Frame：诊断任务、华东、上周 vs 前周、按 SKU 看 | `agents/data_agent/` |
| **Ch.33** | 把「GMV」绑定为 `gmv_ops@2025Q1`，「华东」展开为 `EAST`，输出可编译的 Linked Schema | `infra/semantic_layer/` · `linker.py` |
| **Ch.34** | 编译 Semantic SQL，服务端加 `tenant_id`，只读执行，取 Top SKU 宽表 | `tools/sql_executor/` |
| **Ch.35** | 读 SQL 结果文件，算各品类对下滑差额的贡献度 | `tools/python_sandbox/` |
| **Ch.36** | 画 SKU 贡献条形图，写经营会报告初稿，等人审批后发布 | `chart_renderer/` · `templates/` |
| **Ch.37** | 用业务金标准集与开源对标做 Eval，驱动下一版改 Glossary / Prompt | `core/eval/` |

六章串联的是 **同一条 Run**（如 `run-8f3a`）：Ch.32–33 在 Planner 启动前完成理解与 Linking；Ch.34–36 在 Planner 循环内按序调用 Tool；Ch.37 定义 **上线后如何用 Eval 证明较上季度改进**，并约束下一版是否引入 Vanna / Wren 等组件。

#### Part VI 模块与章节对照

| 模块路径 | 职责 | 主要章节 |
| --- | --- | --- |
| `agents/data_agent/` | AgentSpec、Question Frame、Linker | Ch.32–33 |
| `infra/semantic_layer/` | 指标、View、口径解析、`trusted_context()` | Ch.33 |
| `tools/sql_executor/` | 只读 SQL、校验与执行 | Ch.34 |
| `tools/python_sandbox/` | 分析沙箱、贡献度计算 | Ch.35 |
| `tools/chart_renderer/` | 图表 spec 与渲染 | Ch.36 |
| `agents/data_agent/templates/` | 报告模板、EvidenceRef | Ch.36 |
| `core/policy/` | 行级权限、脱敏（执行侧基线在 `sql_executor`） | Ch.34 + Ch.50 |
| `core/eval/` | 输出质量 Eval 流水线 | Ch.36 + Ch.39 |

**建议阅读顺序**：Ch.32（产品边界）→ Ch.33（语义层）→ Ch.34（NL2SQL）→ Ch.35（Python）→ Ch.36（表达与 Eval）→ Ch.37（本章）。Run / Registry / HITL 底座见 Part V（[Ch.22–30](../part05-agent-capabilities/ch22-agent-runtime.md)）。

#### Part VI 六章能力体系（总结）

Part VI 六章按 **Ch.32 → Ch.33 → Ch.34 → Ch.35 → Ch.36 → Ch.37** 递进：先定产品边界与 Question Frame，再语义层与 NL2SQL，继而 Python 分析与表达层 Eval，最后生态选型。Ch.34–36 的执行与审批依赖 Part V 的 Runtime、Registry 与 HITL（见上节建议阅读顺序）。

| 章 | 一句话 |
| --- | --- |
| **32** | DataAgent ≠ NL2SQL；四种产品形态与 Question Frame |
| **33** | 口径与 Schema Linking 地基 |
| **34** | 安全 SQL、校验与自修复 |
| **35** | 沙箱 Python 与 SQL 协同 |
| **36** | 洞察、图表、报告与输出 Eval |
| **37** | 生态对标、选型、持续改进 |

**下一步**：Part VII [Ch.39](../part07-observability-eval/ch39-dataagent-eval-benchmark.md) 搭建 **评测流水线**（`core/eval/`）；Part IX [Ch.48](../part09-frontend-multimodal/ch48-generative-ui.md) 完善 **Generative UI**，承接 Ch.36 图表与报告的前端渲染。

---

## 本章小结

### 关键结论

1. DataAgent 生态 **按入口、技术路线、部署形态、组织治理四维分化**；少有单品覆盖 Part VI 全链路，山岚华东案例即需六章协同。  
2. **Vanna / WrenAI / DB-GPT / Defog / Sherlock** 各有强项，企业宜 **混合集成**（组件进 Registry），而非重复造 Part V 平台或整包引入双 Runtime。  
3. **ChatBI 是 DataAgent 的早期子集**；BI Copilot 与 DataAgent **并存**，口径须经 `infra/semantic_layer/` 统一。  
4. 选型须同时看 **架构（Part V）、语义层、Eval、HITL**——不单看 NL2SQL demo；采购底线是 tenant 注入、只读执行与 `metric_id@version` 审计。  
5. Part VI 六章 + Part V 平台 = 本书 **DataAgent 主线** 完整地图；§7 走读表将华东下滑案例逐步对应到 `mini-platform/` 模块路径。

### 上线检查清单

- [ ] 是否完成 §7 选型 checklist？  
- [ ] 概念验证是否覆盖 **多轮澄清 + Python 分析 + 报告审批**（华东下滑全链路）？  
- [ ] 是否明确 **12 个月 Eval 与语义层治理** 负责人？  
- [ ] 引入的开源组件是否已映射到 Registry Tool，且无第二套 Runtime？

### 本书延伸阅读

- [Ch.32](ch32-dataagent.md) – [Ch.36](ch36.md) Part VI 全书  
- [Ch.31 框架对标](../part05-agent-capabilities/ch31.md) · [Ch.39 评估](../part07-observability-eval/ch39-dataagent-eval-benchmark.md)  
- [Ch.53 组织演进](../part10-security-org/ch53.md)

---

## 参考文献

[1] Liu, X., et al. (2025). NL2SQL survey. *IEEE TKDE*. [https://doi.org/10.1109/TKDE.2025.3592032](https://doi.org/10.1109/TKDE.2025.3592032)

[2] Tang, Z., et al. (2025). LLM/Agent-as-Data-Analyst: A survey. arXiv:2509.23988. [https://arxiv.org/abs/2509.23988](https://arxiv.org/abs/2509.23988)

[3] Lei, F., et al. (2024). Spider 2.0. *ICLR 2025*. arXiv:2411.07763. [https://arxiv.org/abs/2411.07763](https://arxiv.org/abs/2411.07763)

[4] Huo, N., et al. (2026). BIRD-INTERACT. *ICLR 2026*. arXiv:2510.05318. [https://arxiv.org/abs/2510.05318](https://arxiv.org/abs/2510.05318)

[5] eosphoros-ai. (2024). *DB-GPT*. GitHub. [https://github.com/eosphoros-ai/DB-GPT](https://github.com/eosphoros-ai/DB-GPT)

[6] Canner. (2024). *WrenAI*. GitHub. [https://github.com/Canner/WrenAI](https://github.com/Canner/WrenAI)

[7] vanna-ai. (2024). *Vanna*. GitHub. [https://github.com/vanna-ai/vanna](https://github.com/vanna-ai/vanna)

[8] Defog.ai. (2024). *Defog*. [https://github.com/defog-ai/defog](https://github.com/defog-ai/defog)

[9] 研究型数据 Agent 原型（如社区实验项目 Sherlock 等）在 **复杂推理链** 上常有启发，但 **多数缺少企业级 Runtime、权限与 Eval**；选型时应核验开源仓库的维护频率、许可证与数据出站策略，**不建议整包替换 Part V 平台**。若无法提供稳定开源地址，则仅作架构参考，不纳入采购短名单。

[10] Microsoft. (2024). *Copilot in Power BI*. [https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-introduction](https://learn.microsoft.com/en-us/power-bi/create-reports/copilot-introduction)

[11] Ch.31–Ch.32 本书框架与产品边界章节。

[12] Cube. (2025). Semantic layer docs. [https://cube.dev/docs/product/introduction](https://cube.dev/docs/product/introduction)
