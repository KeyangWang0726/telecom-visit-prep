---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: 'f86c3c4f-658b-4a80-a49d-225e83cb3a64'
  PropagateID: 'f86c3c4f-658b-4a80-a49d-225e83cb3a64'
  ReservedCode1: '4d2c8bee-01ce-461a-8b8a-8e9bab985c31'
  ReservedCode2: '4d2c8bee-01ce-461a-8b8a-8e9bab985c31'
---

# 电信走访通 (telecom-visit-prep)

为中国电信客户经理提供一键式企业走访准备服务：输入企业名称，自动完成企业信息搜索、商机智能推荐、拜访话术生成，输出完整走访准备报告。

## 快速开始

### 星辰超级智能体（原生平台）

```
/install-skill telecom-visit-prep
```

或对话中输入：`@电信走访通 安装技能`

安装后直接输入企业名称即可触发。

### 其他智能体平台

本技能采用纯 Markdown 编写，工作流由 LLM 推理驱动，具备良好的跨平台迁移基础。但由于不同平台的内置工具存在差异，需要进行适配。详见下方 [跨平台适配指南](#跨平台适配指南)。

---

## 技能结构

```
telecom-visit-prep/
├── SKILL.md                        # 技能主文件（工作流定义）
├── README.md                       # 本文件
└── references/                     # 知识库与模板
    ├── telecom-products.md         # 中国电信全产品线知识库
    ├── speech-scripts.md           # 拜访话术模板（双风格）
    └── report-template.md          # 走访报告输出模板
```

---

## 跨平台适配指南

### 依赖分析总览

本技能的运行依赖以下外部能力，不同平台的覆盖情况不同：

| 依赖项 | 依赖级别 | 星辰超级智能体 | Claude Code | Codex CLI |
|--------|---------|--------------|-------------|-----------|
| **网络搜索** | 核心 | `baidu_search`（内置） | 需替代（见下方） | 需替代（见下方） |
| **文件读取** | 核心 | 内置 | 内置 | 内置 |
| **Word 文档生成** | 非核心 | `docx` skill | 需替代 | 需替代 |
| **当前时间** | 非核心 | 内置 | 内置 | 内置 |

> **核心依赖**：缺少则流程无法正常运行
> **非核心依赖**：缺少仅影响部分功能，主流程可降级运行

---

### Claude Code 适配方案

Claude Code 与星辰超级智能体的 Skill 系统最为接近，适配成本最低。

#### 1. 安装技能

将技能目录放入 Claude Code 的 `.claude/skills/` 目录下：

```bash
# 在项目根目录下
mkdir -p .claude/skills/
cp -r telecom-visit-prep/ .claude/skills/
```

或在 Claude Code 对话中使用 `/add-skill` 命令指向 SKILL.md 文件路径。

#### 2. 网络搜索替代方案

Claude Code 没有内置 `baidu_search`，需通过以下方式替代：

**方案 A：MCP 搜索服务器（推荐）**

安装 MCP 搜索工具，推荐以下任一：

| MCP 服务器 | 安装方式 | 说明 |
|-----------|---------|------|
| @anthropic/mcp-web-search | `claude mcp add web-search -- npx @anthropic/mcp-web-search` | Anthropic 官方，需 API Key |
| @anthropic/mcp-brave-search | `claude mcp add brave-search -- npx @anthropic/mcp-brave-search` | Brave Search，需 API Key |
| custom-search-mcp | 自行开发 | 可对接百度/Google等 |

安装后，需在 `SKILL.md` 第1步中将 `baidu_search` 替换为对应的 MCP 搜索工具调用。

**方案 B：WebFetch + Shell 搜索**

Claude Code 内置 `WebFetch` 工具，可直接抓取网页内容。结合 Shell 命令使用搜索引擎：

```bash
# 示例：通过 curl 搜索（需自行封装或使用搜索引擎API）
curl "https://www.baidu.com/s?wd=企业名称+工商信息"
```

> 注意：直接爬取搜索引擎结果可能触发反爬机制，推荐使用方案 A。

#### 3. Word 文档生成替代方案

**方案 A：python-docx 脚本生成**

Claude Code 可通过 Shell 执行 Python 脚本：

```bash
pip install python-docx
```

在 SKILL.md 中将 docx skill 调用改为：让 Claude Code 生成 Python 脚本，使用 python-docx 库生成 Word 文档。

**方案 B：MCP 文档服务器**

搜索并安装支持 docx 生成的 MCP 服务器（如 `mcp-docx`），实现方式与星辰超级智能体原生的 docx skill 类似。

#### 4. SKILL.md 适配要点

Claude Code 的 Skill 系统与星辰超级智能体兼容，SKILL.md 可直接使用，仅需修改以下内容：

```markdown
# 需要修改的位置：

1. 前置依赖检查章节
   - 删除 docx skill 的安装检查逻辑（`~/.config/teleai-super-agent/skills/docx/SKILL.md`）
   - 改为检查 python-docx 是否已安装：`pip show python-docx`

2. 第1步：多维度企业信息搜索
   - 将 `baidu_search` 替换为 MCP 搜索工具或 WebFetch
   - 搜索策略不变，仅工具调用方式变化

3. 报告输出部分
   - 将 `docx` skill 调用替换为 python-docx 脚本生成
   - 或直接输出 Markdown 格式（Claude Code 对话中即可完整展示）
```

---

### Codex CLI 适配方案

Codex CLI 是 OpenAI 推出的终端 AI 代理，不具备 Skill 系统，需要以"指令注入"方式使用。

#### 1. 使用方式

Codex CLI 没有 Skill 文件机制，需要将 SKILL.md 的内容作为系统提示或首条消息注入：

```bash
# 方式一：作为首条消息发送
codex "$(cat telecom-visit-prep/SKILL.md)

请根据以上工作流程，为我生成以下企业的走访准备报告：[企业名称]"

# 方式二：写入项目指令文件
# 在项目根目录创建 AGENTS.md（Codex CLI 会自动读取）
cat telecom-visit-prep/SKILL.md >> AGENTS.md
```

> 注意：Codex CLI 读取 AGENTS.md 后，技能指令会在该项目的所有会话中生效，建议在独立项目目录中使用。

#### 2. 网络搜索替代方案

Codex CLI 可通过 Shell 执行搜索：

**方案 A：Shell 命令 + 搜索引擎 API**

```bash
# 使用百度搜索 API（需申请 API Key）
curl "https://aip.baidubce.com/rest/2.0/search/v1/websearch?query=企业名称+工商信息&access_token=YOUR_TOKEN"

# 或使用 SerpAPI 等
curl "https://serpapi.com/search?q=企业名称+工商信息&api_key=YOUR_KEY"
```

**方案 B：内置浏览器工具**

如果 Codex CLI 配置了浏览器访问能力，可直接浏览企业信息网站（天眼查、企查查等）获取数据。

#### 3. Word 文档生成替代方案

与 Claude Code 方案一致，使用 python-docx：

```bash
pip install python-docx
# 让 Codex CLI 生成 Python 脚本创建 Word 文档
```

#### 4. 关键适配修改

| SKILL.md 中的内容 | Codex CLI 替代方案 |
|------------------|-------------------|
| `baidu_search` 工具调用 | Shell 命令执行 curl/API 调用 |
| `docx` skill | python-docx 脚本生成 |
| 读取 `references/` 文件 | 将 references/ 内容直接内联到 SKILL.md 或 AGENTS.md 中 |
| 前置依赖检查（docx skill 安装） | 检查 python-docx：`pip show python-docx` |

> **重要提示**：Codex CLI 没有自动读取 references/ 目录的能力，需将 `telecom-products.md`、`speech-scripts.md`、`report-template.md` 的内容直接合并到 AGENTS.md 或 SKILL.md 中，确保 LLM 能获取完整的知识库和模板。

---

### 其他平台通用适配要点

无论迁移到哪个智能体平台，核心适配工作集中以下三处：

#### 1. 网络搜索能力替代

这是最关键的适配点。本技能的核心输入来自企业信息搜索，没有搜索能力则流程无法运行。

适配策略：
- **优先**：使用平台提供的 MCP 或插件式搜索工具
- **次选**：通过 Shell/API 调用搜索引擎接口
- **兜底**：手动提供企业信息，告知用户"请先提供该企业的基本信息，我将基于您提供的信息生成报告"

#### 2. Word 文档生成替代

非核心能力，缺少时报告仍可在对话中完整输出。

适配策略：
- **优先**：使用平台文档生成插件/MCP
- **次选**：生成 Python 脚本调用 python-docx
- **兜底**：输出 Markdown 格式报告，用户自行转换

#### 3. 知识库与模板内联

部分平台（如 Codex CLI）不支持自动读取附属文件，需将 `references/` 下的内容合并到主文件中。

适配策略：
- 评估目标平台是否支持文件引用（`[path](path)` 语法）
- 不支持时，将知识库和模板内容直接嵌入 SKILL.md
- 建议在嵌入时保留分隔标记，便于后续维护更新

---

## 平台兼容性速查表

| 平台 | 技能格式兼容 | 搜索工具 | 文档生成 | 适配难度 | 备注 |
|------|------------|---------|---------|---------|------|
| **星辰超级智能体** | 原生 | baidu_search（内置） | docx skill（内置） | 无 | 原生平台 |
| **Claude Code** | 兼容 | 需装 MCP 搜索 | python-docx / MCP | 低 | Skill 格式接近，适配成本最低 |
| **Codex CLI** | 需改造 | Shell + API | python-docx | 中 | 无 Skill 系统，需内联知识库 |
| **Cursor** | 部分兼容 | 需 MCP | python-docx | 中 | .cursorrules 方式注入指令 |
| **Windsurf** | 部分兼容 | 需 MCP | python-docx | 中 | 类似 Cursor |
| **OpenClaw** | 需改造 | 取决于配置 | 取决于配置 | 中-高 | 需确认工具可用性 |

---

## 常见问题

### Q: 不装搜索工具能用吗？

可以降级使用。将 SKILL.md 第1步的搜索改为"由用户提供企业信息"，技能仍能完成画像构建、商机推荐、话术生成和报告输出。

### Q: 不生成 Word 文件会影响报告质量吗？

不会。报告的核心价值在内容而非格式。对话中的 Markdown 输出已经完整，Word 导出仅为方便存档和打印。

### Q: 能否直接把 SKILL.md 复制到其他平台？

取决于目标平台。Claude Code 的 Skill 格式兼容，改动最小；Codex CLI 等无 Skill 机制的平台需要改造使用方式。关键修改点仅三处（搜索工具、文档生成、文件引用），详见上方各平台适配方案。

### Q: 数据安全方面有注意事项吗？

本技能在本地运行，企业信息搜索通过公开渠道获取。星辰超级智能体的 baidu_search 工具数据不上传；其他平台使用 MCP 或 API 搜索时，请注意所选搜索服务的隐私政策。Word 文档生成均在本地完成，不涉及数据外传。

---

## 技能维护

- **作者**：中国电信常熟分公司人工智能与软研中心
- **版本**：1.3.0
- **最后更新**：2026-05-11
- **许可证**：内部使用，如需分发请联系作者