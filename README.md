# ljg-cards

A Claude Code skill that converts text into beautifully styled reading cards (1200×1600 PNG, 3:4 ratio).

```
┌─────────────────────────┐
│  ┌───────────────────┐  │
│  │     Title (green)  │  │
│  ├───────────────────┤  │
│  │                   │  │
│  │  Content area     │  │
│  │  · items (yellow) │  │
│  │  · quotes (dim)   │  │
│  │  · sections (cyan)│  │
│  │                   │  │
│  ├───────────────────┤  │
│  │ source    page    │  │
│  └───────────────────┘  │
│        1200 × 1600      │
└─────────────────────────┘
```

## Features

- **Dark theme** — Catppuccin-inspired palette (`#272B33` card on `#1E2128` background)
- **CJK-first typography** — PingFang SC, 26px body, 1.8 line-height
- **Smart splitting** — greedy algorithm with visual weight calculation, auto-splits long content into multiple cards
- **Semantic HTML** — headings, items, quotes, dividers mapped to styled components

## Install

### As Claude Code plugin

```bash
claude plugin add lijigang/ljg-cards
```

### Manual

```bash
git clone https://github.com/lijigang/ljg-cards.git ~/.claude/plugins/ljg-cards
cd ~/.claude/plugins/ljg-cards
npm install
npx playwright install chromium
```

## Usage

In Claude Code:

```
/ljg-cards
把这段文字做成卡片：...
```

Trigger phrases: `做成卡片`, `生成阅读卡`, `reading card`, `卡片`

## Output

PNG files saved to `~/Downloads/card_{timestamp}_{N}.png`

## Dependencies

- Node.js
- Playwright (Chromium)

## License

MIT
