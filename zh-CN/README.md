<div align="center">
  <h1>MinerU PDF</h1>
  <p><strong>高精度 PDF 内容解析——为 Claude Code 补充公式、图片、表格提取能力</strong></p>
  <p>A standalone Claude Code skill for high-accuracy PDF content extraction</p>

  [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](../LICENSE)
  [![Platform](https://img.shields.io/badge/Platform-Claude%20Code-blue)](https://code.claude.com)
  [![Stars](https://img.shields.io/github/stars/20kiki/mineru-pdf)](https://github.com/20kiki/mineru-pdf)

  <p><strong>Language:</strong> <a href="../README.md">English</a> | <a href="README.md">简体中文</a></p>
</div>

---

## 📋 目录

- [🔍 问题](#-问题)
- [💡 方案](#-方案)
- [📊 效果对比](#-效果对比)
- [🚀 快速开始](#-快速开始)
- [📖 使用方式](#-使用方式)
- [📌 话题标签](#-话题标签)
- [🤝 贡献](#-贡献)
- [📄 许可](#-许可)

---

## 🔍 问题

Claude Code 默认 pdf skill 依赖 **pypdf + pdfplumber**，日常够用，但面对复杂文档时有明显短板：

| 场景 | 实测表现 |
|:---|:---|
| 数学公式 | pypdf 提取为乱码（`PrðfðXÞ/C0 ...` 不可用） |
| 图片提取 | pdfplumber / pypdf 无法提取，pdfimages 只出二进制无位置信息 |
| 输出格式 | 纯文本，丢失标题层级、阅读顺序、图文关联等文档结构 |
| 扫描件 | 完全依赖 pytesseract，配置繁琐且效果一般 |

---

## 💡 方案

**不替换官方 pdf skill。** `mineru-pdf` 是一个独立 skill，用户按需显式调用。

```text
日常 PDF（纯文字 / 单栏）     → 默认 pdf skill（pypdf，秒级）
复杂 PDF（公式 / 双栏 / 扫描件） → 本 skill（MinerU，高精度）
```

- 编辑操作（合并 / 拆分 / 加水印 / 加密）仍然走官方 pdf skill
- 内容解析遇到公式乱码、图片提取等需求时，调用本 skill
- 独立安装，不跟官方 marketplace 更新冲突，不会被覆盖

底层使用 [MinerU](https://github.com/opendatalab/MinerU)（上海 AI Lab 开源的文档解析引擎），完全本地运行，不消耗任何 API 额度。

---

## 📊 效果对比

以一篇双栏学术论文（PAMI 2004, 7 页，含大量公式和图片）实测：

| 维度 | pypdf | MinerU |
|:---|:---|:---|
| 公式提取 | `PrðfðXÞ/C0 /C22 /C21 /C28 Þ/C20 ...` 乱码 | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau) \leq ...` 可编译 LaTeX |
| 图片提取 | 无 | 21 个图片文件（13 个正文插图 + 8 个图元）+ bbox 坐标 + 图注文字 |
| 耗时 | < 1 秒 | ~ 3 分钟（首次需下载模型） |
| 输出 | 纯文本字符串 | Markdown + JSON + 图片 |

> 表中数据均为本仓库实测结果，可在同论文上复现。

---

## 🚀 快速开始

> **前置要求：** Python 3.8+，Claude Code 已安装

### 第一步：打开终端

- **macOS / Linux：** 打开终端（Terminal）
- **Windows：** 按 `Win + R`，输入 `powershell`，回车

### 第二步：安装 MinerU

```bash
pip install "mineru[pipeline]"
```

> 依赖 torch、transformers 等，首次安装需等待 3-5 分钟。

### 第三步：安装 Skill

**macOS / Linux：**
```bash
mkdir -p ~/.claude/skills/mineru-pdf && curl -o ~/.claude/skills/mineru-pdf/SKILL.md https://raw.githubusercontent.com/20kiki/mineru-pdf/master/SKILL.md
```

**Windows（PowerShell）：**
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\mineru-pdf"; Invoke-WebRequest -Uri "https://raw.githubusercontent.com/20kiki/mineru-pdf/master/SKILL.md" -OutFile "$env:USERPROFILE\.claude\skills\mineru-pdf\SKILL.md"
```

> 文件名必须叫 `SKILL.md`。如果已安装旧版，这个命令会直接覆盖更新。
>
> **如果下载失败**（国内访问 raw.githubusercontent.com 可能被墙），用浏览器打开 [SKILL.md](https://github.com/20kiki/mineru-pdf/blob/master/SKILL.md)，复制内容手动保存到上述路径。或者克隆仓库后复制：`git clone https://github.com/20kiki/mineru-pdf.git && cp mineru-pdf/SKILL.md ~/.claude/skills/mineru-pdf/`

### 第四步：使用

重启 Claude Code，输入：

```text
/mineru-pdf 解析这个 PDF
```

首次使用时会自动下载模型（~2 GB，从 HuggingFace），之后缓存本地不再需要网络。

---

## 📖 使用方式

输入 `/mineru-pdf` 后告诉它 PDF 路径和需求，skill 会自动判断是否需要公式识别、OCR 等。

### 什么时候用它

| 信号 | 建议 |
|:---|:---|
| pypdf 输出的公式是 `/Cxx` 乱码 | `/mineru-pdf` |
| PDF 有数学公式、希腊字母 | `/mineru-pdf` |
| 双栏 / 多栏，需要正确阅读顺序 | `/mineru-pdf` |
| 需要提取图片 + 位置信息 | `/mineru-pdf` |
| 扫描件（图片型 PDF，无文字层） | `/mineru-pdf` |
| 纯文字、单栏、无公式 | 默认 pdf skill 更快 |

---

## 📌 话题标签

[`claude-code`](https://github.com/topics/claude-code) [`pdf-parsing`](https://github.com/topics/pdf-parsing) [`mineru`](https://github.com/topics/mineru) [`skill`](https://github.com/topics/skill) [`document-parser`](https://github.com/topics/document-parser) [`ocr`](https://github.com/topics/ocr) [`latex`](https://github.com/topics/latex)

---

## 🤝 贡献

欢迎提交 Issue 和 PR。修改 `SKILL.md` 后在 Claude Code 中测试至少一个复杂 PDF 再提交。

详见 [CONTRIBUTING.md](../CONTRIBUTING.md)。

---

## 📄 许可

MIT — 详见 [LICENSE](../LICENSE)。

[MinerU](https://github.com/opendatalab/MinerU) 引擎使用 AGPL-3.0 协议。

---

<p align="center"><sub>Made with ❤️ for the Claude Code community</sub></p>
