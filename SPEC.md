# Financial Services 中文版项目规范

## 1. 项目目标

**项目名称**: financial-services-cn  
**目标**: 将 financial-services 项目完全汉化，创建中文本地化版本  
**目标用户**: 中文金融从业者（投资银行、私募股权、财富管理、运营等）  
**同步策略**: 保持与英文版同步更新

---

## 2. 翻译范围

### 需要翻译的内容

| 类别 | 文件类型 | 说明 |
|------|----------|------|
| 根目录文档 | README.md, CLAUDE.md | 项目主文档 |
| agent-plugins | plugin.json, agents/*.md, skills/*.md | 命名代理插件 |
| vertical-plugins | .mcp.json, README.md, commands/*.md, skills/*/SKILL.md | 垂直领域插件 |
| partner-built | README.md, .mcp.json | 合作伙伴插件 (LSEG, S&P Global) |
| managed-agent-cookbooks | README.md, steering-examples.json, agent.yaml | 托管代理食谱 |
| marketplace.json | .claude-plugin/marketplace.json | 市场清单 |

### 翻译对象

- 插件名称 (name)
- 描述 (description)
- 系统提示词 (system prompts)
- 技能文档 (skills/SKILL.md)
- 命令文档 (commands/*.md)
- README 文档
- JSON 配置中的描述字段

---

## 3. 项目结构

保持与英文版完全一致的结构：

```
~/Documents/GitHub/financial-services-cn/
├── README.md                    # 中文版自述文件
├── CLAUDE.md                    # 中文版开发指南
├── marketplace.json             # 中文版市场清单
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── agent-plugins/           # 命名代理 (9个)
│   ├── vertical-plugins/        # 垂直领域 (7个)
│   └── partner-built/           # 合作伙伴 (2个)
├── managed-agent-cookbooks/     # 托管代理食谱 (9个)
├── claude-for-msft-365-install/ # Microsoft 365 安装
└── scripts/                     # 脚本工具
```

---

## 4. 命名规范

### 目录与文件
- 目录名保持英文不变（如 `kyc-screener`, `financial-analysis`）
- 文件名保持英文不变（如 `plugin.json`, `SKILL.md`）

### 翻译原则
- **技术术语**: 使用中文行业内常用术语
- **命令名称**: 保持英文不变（如 `/equity-research:screen`）
- **变量/函数名**: 保持英文不变
- **文件路径**: 保持英文不变

### 术语对照表

| 英文 | 中文 |
|------|------|
| plugin | 插件 |
| agent | 代理 |
| skill | 技能 |
| command | 命令 |
| vertical | 垂直领域 |
| private equity | 私募股权 |
| investment banking | 投资银行 |
| wealth management | 财富管理 |
| equity research | 股票研究 |
| financial analysis | 财务分析 |
| fund admin | 基金行政 |
| operations | 运营 |
| KYC (Know Your Customer) | 了解你的客户 |
| earnings | 财报 |
| reconciliation | 对账 |
| valuation | 估值 |
| pitch | 推介 |
| closing | 结账 |
| auditor | 审计 |

---

## 5. 代码风格

### JSON 文件 (plugin.json, .mcp.json)
```json
{
  "name": "kyc-screener",
  "displayName": "KYC 筛查代理",
  "description": "筛选潜在客户并识别潜在风险",
  ...
}
```

### Markdown 文档
- 标题使用中文
- 代码块、命令保持英文
- 术语首次出现时提供英文原文

---

## 6. 测试策略

### 验证清单
- [ ] 所有 plugin.json 格式正确
- [ ] 所有 .mcp.json 格式正确
- [ ] marketplace.json 包含所有插件
- [ ] 命令路径引用正确
- [ ] 技能路径引用正确
- [ ] JSON 文件无语法错误

### 验证方法
```bash
# 运行英文版的检查脚本（可能需要修改路径）
python3 scripts/check.py
```

---

## 7. 同步机制

### 同步策略
1. **手动同步**: 英文版更新后，人工同步翻译
2. **版本对应**: 保持与英文版相同版本号
3. **差异化**: 使用 git 分支或独立仓库管理

### 同步频率
- 英文版发布新版本后一周内完成同步

---

## 8. 边界与约束

### 必须做
- 保持项目结构与英文版一致
- 翻译所有用户可见的文本
- 保持技术术语英文原文
- 测试所有 JSON 文件格式

### 禁止做
- 不翻译目录名和文件名
- 不翻译命令名称
- 不翻译变量名/函数名/路径
- 不修改技术实现逻辑

### 需要确认
- 特定术语的翻译偏好
- 是否有遗漏的翻译内容
- 与其他中文资源的术语一致性
