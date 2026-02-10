---
name: ljg-cards
description: >
  Generates reading cards (1200x1600 PNG, 3:4 ratio) from text input.
  Splits long content into multiple cards automatically.
  Output saved to ~/Downloads/.
  Use when user says "做成卡片", "生成阅读卡", "reading card", "卡片",
  or provides text and asks for visual cards.
metadata:
  author: lijigang
  version: "1.0.0"
allowed-tools: Bash Read Write
---

## Usage

<example>
User: 把这段文字做成卡片：[一段文字]
Assistant: [Calls ljg-cards, generates PNG cards to ~/Downloads/]
</example>

## Instructions

为了执行本项技能，请严格按照以下步骤操作：

### 步骤 1：读取资源

读取以下文件内容到内存（路径相对于本 skill 的 Base directory）：
- `assets/card_template.html`
- 确认截图脚本路径：`assets/capture.js`

### 步骤 2：内容分析与切分

**2.1 预处理**
- 识别标题行（`#`/`##`/`###` 开头，或独立短行）
- 识别引用块（`>` 开头）
- 识别加粗（`**text**`）
- 按空行分割为段落列表

**2.2 计算视觉重量**

模板直接在 1200x1600 全分辨率渲染，正文 26px，行高 1.8。

- 普通段落：字符数 × 1.0
- 标题行（h2 + subtitle）：字符数 × 3.5（字号大 + 上下留白多）
- `.item` 条目组（label + 正文）：字符数 × 1.3
- 引用块：字符数 × 1.2
- 分割线（divider）：固定 40 权重
- 代码块：字符数 × 1.5

**2.3 贪心切分**
- 阈值：每卡约 **500** 字符等价视觉重量
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

### 步骤 3：格式化为 HTML

将内容转为 HTML，遵循以下结构映射：

**基础元素：**
- 普通段落 → `<p>文本</p>`
- 章节标题（##/### 级别） → `<h2>标题</h2>`
- 引用 → `<blockquote><p>引用</p></blockquote>`
- 加粗 → `<strong>文本</strong>`
- 列表 → `<ul><li>...</li></ul>`

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

**首张卡片：**
- 有明确标题时提取为 TITLE_BLOCK

### 步骤 4：渲染模板

对每张卡片，读取 `assets/card_template.html` 并替换模板变量：

| 变量 | 规则 |
|------|------|
| `{{TITLE_BLOCK}}` | 有标题时：`<div class="title-area"><h1>标题</h1></div>`；无标题时：空字符串 |
| `{{BODY_HTML}}` | 步骤 3 生成的 HTML |
| `{{SOURCE}}` | 来源信息（用户提供则填入，否则空） |
| `{{PAGE_INFO}}` | 多卡时 `1 / 3`，单卡时空字符串 |

运行 `date +%Y%m%d_%H%M%S` 获取时间戳，写入：
`/tmp/ljg_card_{timestamp}_{N}.html`

### 步骤 5：截图生成

直接以 1200x1600 全分辨率截图（使用本 skill Base directory 下的脚本）：
```bash
node {BASE_DIR}/assets/capture.js /tmp/ljg_card_{timestamp}_{N}.html ~/Downloads/card_{timestamp}_{N}.png 1200 1600
```

多张卡片可并行截图。

**依赖说明**：需要 `playwright`。首次使用时安装：
```bash
cd {BASE_DIR} && npm install && npx playwright install chromium
```

### 步骤 6：交付

1. 报告卡片数量 + 文件路径
2. 执行 `open ~/Downloads/`
3. 多卡时给出每张摘要（前 30 字）
