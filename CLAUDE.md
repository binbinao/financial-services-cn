# 金融服务业插件

金融服务的 Cowork 插件和 Claude 托管代理模板。每个命名代理都一源两用。

## 仓库结构

```
├── plugins/
│   ├── agent-plugins/               #   命名代理 — 每个都是一个自包含插件
│   │   └── <slug>/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── agents/<slug>.md     #   ← 规范系统提示词（一源两用）
│   │       └── skills/              #   ← 打包副本，从 vertical-plugins/ 同步
│   ├── vertical-plugins/            #   FSI 垂直领域 — 技能源、命令、MCP
│   │   └── <vertical>/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── commands/
│   │       ├── skills/
│   │       └── .mcp.json
│   └── partner-built/               #   合作伙伴插件（LSEG、S&P Global）
├── managed-agent-cookbooks/         #   CMA 食谱（每个命名代理一个目录）
│   └── <slug>/
│       ├── agent.yaml               #   系统 + 技能 → ../../plugins/agent-plugins/<slug>/...
│       ├── subagents/*.yaml         #   深度1叶工作器
│       ├── steering_examples.json
│       └── README.md                #   安全等级 + 交接说明
├── claude-for-msft-365-install/     # Microsoft 365 插件的管理工具（独立于 FSI 插件）
└── scripts/                         # deploy-managed-agent.sh, check.py, validate.py, orchestrate.py, sync-agent-skills.py
```

提交前运行 `python3 scripts/check.py` — 它会检查每个清单、验证所有 `system.file` / `skills.path` / `callable_agents.manifest` 引用是否可解析，并失败如果任何 `agent-plugins/<slug>/skills/` 副本与其 `vertical-plugins/` 源不同步。**在 `vertical-plugins/` 中编辑技能**，然后运行 `python3 scripts/sync-agent-skills.py` 同步到代理包。

`check.py` 还会自动安装一个 `pre-commit` 钩子（`git config core.hooksPath .githooks` — 无需 Husky/Node）。该钩子会 patch-bump 任何插件的 `.claude-plugin/plugin.json` `version`，使分支恰好比 `main` 领先一个 patch（一次 bump，而非每次提交 — 插件的 `version` 控制已安装用户的更新推送）。`version-bump` GitHub Action 在 PR 上执行相同规则作为后盾。使用 `git commit --no-verify` 绕过单次提交；bump 逻辑位于 `scripts/version_bump.py`。

## 关键文件

- `marketplace.json`：市场清单 — 注册所有插件及其源路径
- `plugin.json`：插件元数据 — 名称、描述、版本和组件发现设置
- `commands/*.md`：斜杠命令，以 `/plugin:command-name` 语法调用
- `skills/*/SKILL.md`：特定任务的详细知识和工作流程
- `*.local.md`：用户特定配置（git 忽略）
- `mcp-categories.json`：插件共享的规范 MCP 类别定义

## 开发工作流程

1. 直接编辑 markdown 文件 — 更改立即生效
2. 使用 `/plugin:command-name` 语法测试命令
3. 技能在触发条件匹配时自动调用
