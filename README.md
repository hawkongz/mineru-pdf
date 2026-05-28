<div align="center">
  <h1>MinerU PDF</h1>
  <p><strong>High-accuracy PDF content extraction for Claude Code — formulas, images, tables</strong></p>
  <p>为 Claude Code 补充复杂 PDF 的高精度内容解析能力</p>

  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
  [![Platform](https://img.shields.io/badge/Platform-Claude%20Code-blue)](https://code.claude.com)
  [![Stars](https://img.shields.io/github/stars/20kiki/mineru-enhance-pdf-skill)](https://github.com/20kiki/mineru-enhance-pdf-skill)

  <p><strong>Language:</strong> <a href="README.md">English</a> | <a href="zh-CN/README.md">简体中文</a></p>
</div>

---

## 📋 Table of Contents

- [🔍 The Problem](#-the-problem)
- [💡 Solution](#-solution)
- [📊 Before & After](#-before--after)
- [🚀 Quick Start](#-quick-start)
- [📖 Usage](#-usage)
- [📌 Topics](#-topics)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)

---

## 🔍 The Problem

Claude Code's default pdf skill relies on **pypdf + pdfplumber**. Fine for simple documents, but falls short on complex ones:

| Scenario | What actually happens |
|:---|:---|
| Math formulas | pypdf outputs garbage like `PrðfðXÞ/C0 ...` — completely unusable |
| Image extraction | pdfplumber / pypdf cannot extract images; pdfimages gives raw binaries with no position info |
| Output format | Plain text, losing heading hierarchy, reading order, and image-caption associations |
| Scanned PDFs | Entirely dependent on pytesseract with tedious setup and mediocre results |

---

## 💡 Solution

**Does not replace the official pdf skill.** `mineru-pdf` is a standalone skill — use it explicitly when you need high-quality extraction.

```
Simple PDFs (plain text / single column) → default pdf skill (pypdf, sub-second)
Complex PDFs (formulas / multi-column / scanned) → this skill (MinerU, high accuracy)
```

- Editing operations (merge, split, watermark, encrypt) stay with the official pdf skill
- Content extraction with formulas, images, or complex layouts → this skill
- Installed independently — won't conflict with or be overwritten by marketplace updates

Powered by [MinerU](https://github.com/opendatalab/MinerU) (Shanghai AI Lab), running entirely locally — no API keys, no usage limits.

---

## 📊 Before & After

Tested on a dual-column academic paper (PAMI 2004, 7 pages, formulas and figures throughout):

| Dimension | pypdf | MinerU |
|:---|:---|:---|
| Formula extraction | `PrðfðXÞ/C0 /C22 /C21 /C28 Þ/C20 ...` garbled | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau) \leq ...` compilable LaTeX |
| Image extraction | None | 21 image files (13 figures + 8 elements) + bbox coordinates + captions |
| Time | < 1 sec | ~ 3 min (first run downloads models) |
| Output | Raw text string | Markdown + JSON + Images |

> All data is from actual tests on the same paper — reproducible.

---

## 🚀 Quick Start

> **Prerequisites:** Python 3.8+, Claude Code installed

### Step 1 — Install MinerU

```bash
pip install "mineru[pipeline]"
```

> Pulls in torch, transformers, etc. First install takes 3-5 minutes.

### Step 2 — Install the Skill

Place `SKILL.md` in Claude Code's skills directory:

- **macOS / Linux:** `~/.claude/skills/mineru-pdf/SKILL.md`
- **Windows:** `C:\Users\<username>\.claude\skills\mineru-pdf\SKILL.md`

> Create the `skills` directory if it doesn't exist. The file must be named `SKILL.md`.

### Step 3 — Use It

Restart Claude Code, then:

```
/mineru-pdf extract this PDF
```

On first use, MinerU auto-downloads models (~2 GB from HuggingFace), cached permanently thereafter.

---

## 📖 Usage

Type `/mineru-pdf` followed by your PDF path and needs. The skill auto-detects whether formula recognition, OCR, etc. are needed.

### When to use it

| Signal | Action |
|:---|:---|
| pypdf output has `/Cxx` escape codes | `/mineru-pdf` |
| PDF contains math formulas, Greek letters | `/mineru-pdf` |
| Multi-column layout — reading order matters | `/mineru-pdf` |
| Need images with position metadata | `/mineru-pdf` |
| Scanned PDF (image-based, no text layer) | `/mineru-pdf` |
| Plain text, single column, no formulas | Default pdf skill is faster |

---

## 📌 Topics

[`claude-code`](https://github.com/topics/claude-code) [`pdf-parsing`](https://github.com/topics/pdf-parsing) [`mineru`](https://github.com/topics/mineru) [`skill`](https://github.com/topics/skill) [`document-parser`](https://github.com/topics/document-parser) [`ocr`](https://github.com/topics/ocr) [`latex`](https://github.com/topics/latex)

---

## 🤝 Contributing

Issues and PRs welcome. Test changes to `SKILL.md` with at least one complex PDF in Claude Code before submitting.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📄 License

MIT — see [LICENSE](LICENSE).

The [MinerU](https://github.com/opendatalab/MinerU) engine is licensed under AGPL-3.0.

---

<p align="center"><sub>Made with ❤️ for the Claude Code community</sub></p>
