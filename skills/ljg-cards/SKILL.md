---
name: ljg-cards
description: Generates reading cards from text input. Default mode renders a single long PNG (1080px wide, height auto). Use -m flag for multi-card mode (1080x1440 per card). Output saved to ~/Downloads/. Use when user says "做成卡片", "生成阅读卡", "reading card", "卡片", or provides text and asks for visual cards.
---

## 约束

本 skill 输出为视觉文件（PNG），不适用 L0 中的 Org-mode 和 Denote 规范。输出为图片渲染，不受 ASCII-only 限制。

## Usage

<example>
User: 把这段文字做成卡片：[一段文字]
Assistant: [Calls ljg-cards, generates a single long PNG card to ~/Downloads/]
</example>

<example>
User: /ljg-cards -m [一段文字]
Assistant: [Calls ljg-cards in multi mode, generates multiple 1080x1440 PNG cards to ~/Downloads/]
</example>

## 参数

| 参数 | 说明 |
|------|------|
| （无） | 默认模式：所有内容渲染为一张宽 1080px 的长图，高度自适应 |
| `-m` | 多卡模式：自动切分为多张 1080x1440 卡片 |

## Instructions

为了执行本项技能，请严格按照以下步骤操作：

**首先判断模式**：检查用户输入中是否包含 `-m` 参数。
- 有 `-m` → 进入**多卡模式**（跳到「多卡模式」章节）
- 无 `-m` → 进入**默认长图模式**（继续下方步骤）

### 步骤 1：读取资源

读取以下文件内容到内存：
- `~/.claude/skills/ljg-cards/assets/card_template_long.html`
- 确认截图脚本路径：`~/.claude/skills/ljg-cards/assets/capture.js`

### 步骤 2：内容预处理

- 识别标题行（`#`/`##`/`###` 开头，或独立短行）
- 识别引用块（`>` 开头）
- 识别加粗（`**text**`）
- **识别金句**：独立成段的短句（通常 < 25 字），承载核心洞察，用 `.highlight` 渲染
- 按空行分割为段落列表
- **不做切分**：所有内容放在一张卡内

### 步骤 3：格式化为 HTML

将内容转为 HTML，遵循以下结构映射：

**基础元素：**
- 普通段落 → `<p>文本</p>`
- 章节标题（##/### 级别） → `<h2>标题</h2>`
- 引用 → `<blockquote><p>引用</p></blockquote>`
- 加粗 → `<strong>文本</strong>`
- 列表 → `<ul><li>...</li></ul>`

**金句（独立成段的核心洞察短句，视觉突出）：**
```html
<p class="highlight">金句文本</p>
```
判断标准：独立成段、< 25 字、承载关键洞察。用 `.highlight` 而非 `<p><strong>`。

**条目组（核心结构，用于有标题+正文的并列条目）：**
```html
<div class="item">
  <p class="label">条目标题</p>
  <p>条目正文</p>
</div>
```

**副标题标签（用于安静的分类标签）：**
```html
<p class="subtitle">标签文字</p>
```

**分割线（章节之间使用）：**
```html
<div class="divider"></div>
```

### 步骤 4：渲染模板

读取 `card_template_long.html` 并替换模板变量：

| 变量 | 规则 |
|------|------|
| `{{TITLE_BLOCK}}` | 有标题时：`<div class="title-area"><h1>标题</h1></div>`；无标题时：空字符串 |
| `{{BODY_HTML}}` | 步骤 3 生成的全部 HTML |
| `{{SOURCE}}` | 来源/作者信息（用户提供则填入，否则 `李继刚`） |

注意：长图模板无 `{{HEADER_BLOCK}}`、无 `{{PAGE_INFO}}`。logo 图片路径已硬编码在模板中，无需替换。

**文件命名**：从内容中提取标题或核心思想作为文件名前缀 `{name}`：
- 优先用文章标题（h1）
- 无标题则取首段核心关键词（2-6 个中文字或英文单词）
- 文件名规则：中文直接用，英文用连字符连接，去除标点和特殊字符，长度不超过 20 字符

写入：`/tmp/ljg_card_long_{name}.html`

### 步骤 5：截图生成

使用 fullpage 模式截图，高度由内容自动撑开：
```bash
node ~/.claude/skills/ljg-cards/assets/capture.js /tmp/ljg_card_long_{name}.html ~/Downloads/{name}.png 1080 800 fullpage
```

**依赖说明**：`playwright` 已安装在 `~/.claude/skills/ljg-cards/node_modules/`。如报错：
```bash
cd ~/.claude/skills/ljg-cards && npm install playwright && npx playwright install chromium
```

### 步骤 6：交付

1. 报告文件路径
2. 执行 `open ~/Downloads/`

---

## 多卡模式（`-m` 参数）

当用户传入 `-m` 参数时，执行以下流程（替代上方默认长图流程）：

### M-步骤 1：读取资源

读取以下文件内容到内存：
- `~/.claude/skills/ljg-cards/assets/card_template.html`
- 确认截图脚本路径：`~/.claude/skills/ljg-cards/assets/capture.js`

### M-步骤 2：内容分析与切分

**2.1 预处理**

与默认模式步骤 2 相同：
- 识别标题行、引用块、加粗、金句
- 按空行分割为段落列表

**2.2 计算视觉重量**

模板直接在 1080x1440 全分辨率渲染，正文 36px，行高 1.7。手机端缩放比 ≈ 0.36，实际显示约 13pt。

- 普通段落：字符数 × 1.4
- 标题行（h1 首卡 84px）：字符数 × 6.0
- 金句（`.highlight` 40px + 左边框 + 上下留白）：字符数 × 3.0
- `.item` 条目组（label + 正文）：字符数 × 1.8
- 引用块：字符数 × 1.7
- 分割线（divider）：固定 60 权重
- 代码块：字符数 × 2.2
- Running title（续页头部）：固定 70 权重

**2.3 贪心切分**
- 阈值：每卡约 **380** 字符等价视觉重量
- 逐段累加，超过阈值时在当前段之前切分
- **切分规则**：
  - 绝不在句子中间切
  - 优先在段落/条目/章节边界切
  - 标题不落单（必须跟至少一个内容元素在同一卡）
  - 超长单段在句号处强制切
  - 一个章节（h2 + 3 items）通常刚好一卡

**2.4 特殊情况**
- 只有一张卡：不显示页码
- 多张卡：显示 `1 / N` 格式页码

### M-步骤 3：格式化为 HTML

与默认模式步骤 3 完全相同的结构映射规则。

### M-步骤 4：渲染模板

对每张卡片，读取 `card_template.html` 并替换模板变量：

| 变量 | 规则 |
|------|------|
| `{{HEADER_BLOCK}}` | 续页卡：`<div class="header"><span class="running-title">文章标题</span></div>`；首卡或单卡：空字符串 |
| `{{TITLE_BLOCK}}` | 首卡有标题时：`<div class="title-area"><h1>标题</h1></div>`；续页卡或无标题时：空字符串 |
| `{{BODY_HTML}}` | M-步骤 3 生成的 HTML |
| `{{SOURCE}}` | 来源/作者信息（用户提供则填入，否则 `李继刚`） |
| `{{PAGE_INFO}}` | 多卡时 `1 / 3`，单卡时空字符串 |

注意：logo 图片路径已硬编码在模板中，无需替换。

**文件命名**：同默认模式规则，从内容提取标题或核心思想作为 `{name}`。

写入：`/tmp/ljg_card_{name}_{N}.html`

### M-步骤 5：截图生成

直接以 1080x1440 全分辨率截图：
```bash
node ~/.claude/skills/ljg-cards/assets/capture.js /tmp/ljg_card_{name}_{N}.html ~/Downloads/{name}_{N}.png 1080 1440
```

多张卡片可并行截图。

**依赖说明**：同默认模式。

### M-步骤 6：交付

1. 报告卡片数量 + 文件路径
2. 执行 `open ~/Downloads/`
3. 多卡时给出每张摘要（前 30 字）
