# 金融服务领域的 Claude

针对金融服务行业最常见的工作流程提供的参考代理、技能和数据连接器——投资银行、股票研究、私募股权和财富管理。

这里的所有内容都通过**同一来源的两种方式**提供：作为 [Claude Cowork](https://claude.com/product/cowork) 插件安装，或通过 [Claude Managed Agents API](https://docs.claude.com/en/api/managed-agents) 在您自己的工作流引擎后端部署。相同的系统提示、相同的技能——由您选择运行位置。

> [!IMPORTANT]
> 本仓库中的任何内容都不构成投资、法律、税务或会计建议。这些代理起草的是分析师工作成果——模型、备忘录、研究笔记、对账——供合格专业人员审阅。它们不做出投资建议、执行交易、承担风险、记账或批准入职；每个输出都是为人工审批而暂存的。您有责任验证输出，并遵守适用于您公司的法律法规。

## 仓库内容：

- **[代理](#agents)** — 命名的端到端工作流代理（Pitch Agent、Market Researcher、GL Reconciler 等）。每个都作为 Cowork 插件**以及**通过 `/v1/agents` 部署的 [Claude Managed Agent 模板](./managed-agent-cookbooks) 一起提供。
- **[垂直插件](#vertical-plugins)** — 底层技能、斜杠命令和数据连接器，按 FSI 垂直领域打包。如果您只需要 `/comps`、`/dcf`、`/earnings` 和连接器而不需要完整代理，可以单独安装这些。

## 代理

每个代理以其运行的工作流命名。它们是起点：安装与您工作匹配的代理，然后根据您公司的方式来调整提示、技能和连接器。

每个代理插件都是**独立的**——它绑定了它使用的技能，所以只需安装代理即可。

| 功能 | 代理 | 功能说明 |
|---|---|---|
| **覆盖与咨询** | **[Pitch Agent](./plugins/agent-plugins/pitch-agent)** | 可比公司分析、先例、LBO → 品牌推介材料，端到端 |
| | **[Meeting Prep Agent](./plugins/agent-plugins/meeting-prep-agent)** | 每次客户会议前的简报包 |
| **研究与建模** | **[Market Researcher](./plugins/agent-plugins/market-researcher)** | 行业或主题 → 行业概览、竞争格局、可比公司、创意清单 |
| | **[Earnings Reviewer](./plugins/agent-plugins/earnings-reviewer)** | 财报电话会议 + 文件 → 模型更新 → 笔记草稿 |
| | **[Model Builder](./plugins/agent-plugins/model-builder)** | DCF、LBO、三张报表、可比公司分析 — 实时在 Excel 中 |
| **基金行政与财务运营** | **[Valuation Reviewer](./plugins/agent-plugins/valuation-reviewer)** | 摄入 GP 包、运行估值模板、暂存 LP 报告 |
| | **[GL Reconciler](./plugins/agent-plugins/gl-reconciler)** | 查找差异、追溯根本原因、提交审批 |
| | **[Month-End Closer](./plugins/agent-plugins/month-end-closer)** | 预提、滚动、差异说明 |
| | **[Statement Auditor](./plugins/agent-plugins/statement-auditor)** | 审计分发前的 LP 报表 |
| **运营与入职** | **[KYC Screener](./plugins/agent-plugins/kyc-screener)** | 解析入职文档、运行规则引擎、标记缺失项 |

关于 Managed Agent 部署——`agent.yaml`、叶工作器子代理、转向事件示例和每个代理的安全说明——请参阅 **[managed-agent-cookbooks/](./managed-agent-cookbooks)**。

## 仓库结构

```
plugins/
  agent-plugins/               # 命名代理 — 每个都是独立的插件
  vertical-plugins/            # 按 FSI 垂直领域打包的技能 + 命令，以及 MCP 连接器
  partner-built/               # 合作伙伴编写的插件（LSEG、S&P Global）
managed-agent-cookbooks/       # Claude Managed Agent 指南 — 每个代理一个目录
claude-for-msft-365-install/   # 管理工具用于配置 Claude Microsoft 365 插件
scripts/                       # deploy-managed-agent.sh · check.py · validate.py · orchestrate.py · sync-agent-skills.py
```

## 快速开始

### Cowork

在 Cowork 中，打开 **Settings → Plugins → Add plugin**，然后：

- **粘贴此仓库 URL** — `https://github.com/anthropics/financial-services` — 然后从市场列表中选择您想要的代理和垂直领域，或
- **上传 zip 文件** — 将 `plugins/` 下的任何目录（例如 `plugins/agent-plugins/pitch-agent/`）压缩并拖入。

### Claude Code

```bash
# 添加市场
claude plugin marketplace add anthropics/financial-services

# 核心技能 + 连接器（首先安装）
claude plugin install financial-analysis@claude-for-financial-services

# 命名代理 — 选择您想要的
claude plugin install pitch-agent@claude-for-financial-services
claude plugin install gl-reconciler@claude-for-financial-services
claude plugin install market-researcher@claude-for-financial-services

# 垂直技能包
claude plugin install investment-banking@claude-for-financial-services
claude plugin install equity-research@claude-for-financial-services
```

安装后，代理会出现在 Cowork 调度中，技能会在相关时自动触发，斜杠命令可在会话中使用（`/comps`、`/dcf`、`/earnings`、`/ic-memo` 等）。

### Claude Managed Agents

```bash
export ANTHROPIC_API_KEY=sk-ant-...
scripts/deploy-managed-agent.sh gl-reconciler
```

[`managed-agent-cookbooks/`](./managed-agent-cookbooks) 下的每个模板引用与其插件对应物相同的系统提示和技能。部署脚本解析文件引用、上传技能、创建叶工作器子代理，并将编排器 POST 到 `/v1/agents`。请参阅 [`scripts/orchestrate.py`](./scripts/orchestrate.py) 了解参考事件循环，通过您自己的编排层在代理之间路由 `handoff_request` 事件。

> **研究预览：** 子代理委托（`callable_agents`）是预览功能。请参阅各代理的 README 了解安全性和交接指导。

## 各部分如何协同工作

| | 是什么 | 位置 |
|---|---|---|
| **代理** | 端到端拥有工作流的独立插件 — 系统提示加上它使用的技能。 Cowork 和 Managed Agent 包装器都引用同一目录。 | `plugins/agent-plugins/<slug>/` |
| **技能** | Claude 在相关时自动调用的领域专业知识、约定和分步方法。在垂直领域编写一次；每个代理捆绑它需要的同步副本。 | `plugins/vertical-plugins/<vertical>/skills/`（源）· `plugins/agent-plugins/<slug>/skills/`（捆绑） |
| **命令** | 您明确触发的斜杠操作（`/comps`、`/earnings`、`/ic-memo`）。 | `plugins/vertical-plugins/<vertical>/commands/` |
| **连接器** | 将 Claude 连接到您的数据的 [MCP 服务器](https://modelcontextprotocol.io/) — 终端、研究平台、文档存储。 | `plugins/vertical-plugins/financial-analysis/.mcp.json` |
| **Managed-agent 包装器** | `agent.yaml` + 深度为 1 的子代理 + 无人值守部署的转向示例。 | `managed-agent-cookbooks/<slug>/` |

一切都是基于文件的 — markdown 和 JSON，无需构建步骤。

## 垂直插件

从 **financial-analysis** 开始 — 它包含共享的建模技能和所有数据连接器。为您需要的工作流添加垂直领域。

| 插件 | 功能 |
|---|---|
| **[financial-analysis](./plugins/vertical-plugins/financial-analysis)** *(核心)* | 可比公司分析、DCF、LBO、三张报表、演示材料 QC、Excel 审计。全部 11 个数据连接器。 |
| **[investment-banking](./plugins/vertical-plugins/investment-banking)** | CIM、 teaser、流程函、买家名单、并购模型、交易跟踪。 |
| **[equity-research](./plugins/vertical-plugins/equity-research)** | 财报笔记、首次覆盖、模型更新、主题和催化剂跟踪。 |
| **[private-equity](./plugins/vertical-plugins/private-equity)** | 筛选、筛选、尽职调查清单、IC 备忘录、组合监控。 |
| **[wealth-management](./plugins/vertical-plugins/wealth-management)** | 客户回顾、财务计划、再平衡、报告、税务亏损收割。 |
| **[fund-admin](./plugins/vertical-plugins/fund-admin)** | 总账对账、差异追溯、预提、滚动、差异说明、NAV 核对。 |
| **[operations](./plugins/vertical-plugins/operations)** | KYC 文档解析和规则网格评估。 |
| **[lseg](./plugins/partner-built/lseg)** *(合作伙伴)* | 债券 RV、互换曲线、FX carry、期权波动率、宏观利率监控，基于 LSEG 数据。 |
| **[sp-global](./plugins/partner-built/spglobal)** *(合作伙伴)* |  Tear sheets、财报预览、资金摘要，基于 S&P Capital IQ。 |

## MCP 集成

所有连接器都集中在 **financial-analysis** 核心插件中，并在其他插件之间共享。

| 提供商 | URL |
|---|---|
| [Daloopa](https://www.daloopa.com/) | `https://mcp.daloopa.com/server/mcp` |
| [Morningstar](https://www.morningstar.com/) | `https://mcp.morningstar.com/mcp` |
| [S&P Global](https://www.spglobal.com/) | `https://kfinance.kensho.com/integrations/mcp` |
| [FactSet](https://www.factset.com/) | `https://mcp.factset.com/mcp` |
| [Moody's](https://www.moodys.com/) | `https://api.moodys.com/genai-ready-data/m1/mcp` |
| [MT Newswires](https://www.mtnewswires.com/) | `https://vast-mcp.blueskyapi.com/mtnewswires` |
| [Aiera](https://www.aiera.com/) | `https://mcp-pub.aiera.com` |
| [LSEG](https://www.lseg.com/) | `https://api.analytics.lseg.com/lfa/mcp` |
| [PitchBook](https://pitchbook.com/) | `https://premium.mcp.pitchbook.com/mcp` |
| [Chronograph](https://www.chronograph.pe/) | `https://ai.chronograph.pe/mcp` |
| [Egnyte](https://www.egnyte.com/) | `https://mcp-server.egnyte.com/mcp` |
| [Box](https://www.box.com/home) | `https://mcp.box.com` |

> MCP 访问可能需要提供商订阅或 API 密钥。

## Claude for Microsoft 365 — 安装工具

如果您的公司通过 Microsoft 365 插件在 Excel、PowerPoint、Word 和 Outlook 中运行 Claude，[`claude-for-msft-365-install/`](./claude-for-msft-365-install) 是管理工具，用于针对**您自己的云**配置它——Vertex AI、Bedrock 或内部 LLM 网关——而不是 Anthropic 的 API。

它是一个 Claude Code 插件（不是 Cowork 插件），引导 IT 管理员生成自定义插件清单、授予 Azure 管理员同意，并通过 Microsoft Graph 编写每用户路由配置。安装方式：

```bash
claude plugin install claude-for-msft-365-install@claude-for-financial-services
/claude-for-msft-365-install:setup
```

这与上述代理和垂直插件是分开的——它是让插件在租户中部署的入口，之后这里的代理和技能在其中运行。

## 定制化

这些是参考模板——根据您公司的工作方式进行调整会让它们更好。

- **更换连接器** — 将 `.mcp.json` 指向您的数据提供商和内部系统。
- **添加公司背景** — 将您的术语、流程和格式标准放入技能文件。
- **使用您的模板** — `/ppt-template` 教 Claude 您的品牌 PowerPoint 布局。
- **调整代理范围** — 编辑 `agents/<slug>.md` 以匹配您团队实际运行工作流的方式。
- **添加您自己的** — 为我们未涵盖的工作流复制结构。

## 技能和命令参考

<details>
<summary><b>financial-analysis</b> — 核心建模、Excel、演示材料 QC</summary>

| 技能 | 命令 | 说明 |
|---|---|---|
| comps-analysis | `/comps` | 使用交易倍数的可比公司分析 |
| dcf-model | `/dcf` | 带 WACC 和敏感性分析的 DCF 估值 |
| lbo-model | `/lbo` | 杠杆收购模型 |
| 3-statement-model | `/3-statement-model` | 填充三张报表财务模型模板 |
| audit-xls | `/debug-model` | Excel 模型审计 — 公式追踪、硬编码检测、平衡检查 |
| clean-data-xls | — | 在 Excel 中规范化和清理表格数据 |
| deck-refresh | — | 重新链接和刷新演示材料中嵌入的图表/表格 |
| competitive-analysis | `/competitive-analysis` | 竞争格局和市场定位 |
| ib-check-deck | — | 检查演示材料的错误和一致性 |
| pptx-author | — | 无头生成 `.pptx` 文件（Managed Agent 模式） |
| xlsx-author | — | 无头生成 `.xlsx` 文件（Managed Agent 模式） |
| ppt-template-creator | `/ppt-template` | 创建可重用的 PPT 模板技能 |
| skill-creator | — | 创建新技能的指南 |

</details>

<details>
<summary><b>investment-banking</b> — 交易材料和执行</summary>

| 技能 | 命令 | 说明 |
|---|---|---|
| strip-profile | `/one-pager` | 推介材料的公司简介单页 |
| pitch-deck | — | 用数据填充推介材料模板 |
| datapack-builder | — | 从 CIM 和文件构建数据包 |
| cim-builder | `/cim` | 起草保密信息备忘录 |
| teaser | `/teaser` | 匿名公司 teaser 单页 |
| buyer-list | `/buyer-list` | 战略和财务买家范围 |
| merger-model | `/merger-model` | 增厚/稀释并购分析 |
| process-letter | `/process-letter` | 投标说明和流程函 |
| deal-tracker | `/deal-tracker` | 跟踪实时交易、里程碑和待办事项 |

</details>

<details>
<summary><b>equity-research</b> — 覆盖和发布</summary>

| 技能 | 命令 | 说明 |
|---|---|---|
| earnings-analysis | `/earnings` | 财报后季度更新报告 |
| earnings-preview | `/earnings-preview` | 财报前情景分析和关键指标 |
| initiating-coverage | `/initiate` | 机构级首次覆盖报告 |
| model-update | `/model-update` | 用新数据更新财务模型 |
| morning-note | `/morning-note` | 早会笔记和交易想法 |
| sector-overview | `/sector` | 行业格局和主题报告 |
| thesis-tracker | `/thesis` | 维护和更新投资论点 |
| catalyst-calendar | `/catalysts` | 跟踪覆盖范围内的即将到来的催化剂 |
| idea-generation | `/screen` | 股票筛选和想法来源 |

</details>

<details>
<summary><b>private-equity</b> — 从筛选到组合运营</summary>

| 技能 | 命令 | 说明 |
|---|---|---|
| deal-sourcing | `/source` | 发现公司、检查 CRM、起草创始人联系 |
| deal-screening | `/screen-deal` | 对收到的 CIM 和 teaser 快速通过/拒绝 |
| dd-checklist | `/dd-checklist` | 按工作流的尽职调查清单 |
| dd-meeting-prep | `/dd-prep` | 为管理层演示和专家电话做准备 |
| unit-economics | `/unit-economics` | ARR 队列、LTV/CAC、净留存率、收入质量 |
| returns-analysis | `/returns` | IRR/MOIC 敏感性表 |
| ic-memo | `/ic-memo` | 投资委员会备忘录起草 |
| portfolio-monitoring | `/portfolio` | 跟踪组合公司 KPI 和差异 |
| value-creation-plan | `/value-creation` | 交割后 100 天计划和 EBITDA 桥接 |
| ai-readiness | `/ai-readiness` | 评估组合公司的人工智能就绪程度 |

</details>

<details>
<summary><b>wealth-management</b> — 顾问工作流</summary>

| 技能 | 命令 | 说明 |
|---|---|---|
| client-review | `/client-review` | 准备客户会议，包含业绩和讨论要点 |
| financial-plan | `/financial-plan` | 退休、教育、遗产和现金流预测 |
| portfolio-rebalance | `/rebalance` | 分配漂移分析和税务感知再平衡 |
| client-report | `/client-report` | 面向客户的业绩报告 |
| investment-proposal | `/proposal` | 潜在客户提案 |
| tax-loss-harvesting | `/tlh` | 识别税务亏损收割机会并管理洗售 |

</details>

## 贡献

这里的一切都是 markdown 和 YAML。Fork、编辑、PR。对于新内容：

- 新技能 → 在 `plugins/vertical-plugins/<vertical>/skills/` 下添加，然后运行 `python3 scripts/sync-agent-skills.py` 以同步到任何捆绑它的代理。
- 新代理 → `plugins/agent-plugins/<slug>/`（包含 `agents/<slug>.md` + `skills/`）以及匹配的 `managed-agent-cookbooks/<slug>/`。
- 推送前运行 `python3 scripts/check.py` — 它会检查每个清单、验证所有跨文件引用，并在任何捆绑技能与其垂直源有漂移时失败。

## 许可证

[Apache License 2.0](./LICENSE)
