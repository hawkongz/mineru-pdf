<h1 align="center">MinerU PDF Extract Skill</h1>
<p align="center"><b>独立 skill——为 Claude Code 补充复杂 PDF 的高精度内容解析能力</b></p>
<p align="center">A standalone skill for high-accuracy PDF content extraction: formulas, images, tables, scanned docs</p>

---

## 目录 / TOC

- [问题 / Problem](#问题--problem)
- [方案 / Solution](#方案--solution)
- [效果对比 / Before & After](#效果对比--before--after)
- [安装教程 / Setup](#安装教程--setup)
- [使用方式 / Usage](#使用方式--usage)

---

## 问题 / Problem

Claude Code 默认 pdf skill 依赖 **pypdf + pdfplumber**，日常够用，但复杂文档有明显短板：

| 问题 | 实测表现 |
|---|---|
| 数学公式 | pypdf 提取为乱码（`PrðfðXÞ/C0 ...` 不可用） |
| 图片 | pdfplumber/pypdf 无法提取，pdfimages 只出二进制无位置信息 |
| 输出格式 | 纯文本，丢失文档结构（标题层级、阅读顺序、图文关联） |
| 扫描件 | 完全依赖 pytesseract，配置繁琐且效果一般 |

---

## 方案 / Solution

**不替换官方 pdf skill。** 本仓库是一个**独立 skill**，用户显式调用时激活。

```
日常 PDF（纯文字、单栏） → 默认 pdf skill（pypdf，秒级）
复杂 PDF（公式/双栏/扫描件）→ 这个 skill（MinerU，高精度）
```

- 编辑操作（合并/拆分/加水印/加密）仍然走官方 pdf skill
- 内容解析遇到公式乱码、图片提取等需求时，调用本 skill
- 不跟官方 marketplace 更新冲突，不会被覆盖

---

## 效果对比 / Before & After

以一篇双栏学术论文（PAMI 2004, 7 页，含大量公式和图片）实测：

| 维度 | pypdf | MinerU |
|---|---|---|
| 公式提取 | `PrðfðXÞ/C0 /C22 /C21 /C28 Þ/C20 ...` 乱码 | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau) \leq ...` 可编译 LaTeX |
| 图片提取 | 无 | 21 个图片文件（13 个正文插图 + 8 个图元）+ bbox 坐标 + 图注文字 |
| 耗时 | <1 秒 | ~3 分钟（首次需下载模型） |
| 输出 | 纯文本字符串 | Markdown + JSON + 图片 |

> 表中数据均为本仓库实测结果，可在同论文上复现。

---

## 安装教程 / Setup

### 第一步：安装 MinerU

```bash
pip install "mineru[pipeline]"
```

> 依赖 torch、transformers 等，等待 3-5 分钟。

### 第二步：安装 Skill

将 `SKILL.md` 放到 Claude Code 的 skills 目录即可。

### 第三步：重启 Claude Code

重启后，输入 `/mineru-pdf` 即可激活本 skill。

首次使用时 MinerU 自动下载模型（~2GB，一次缓存）。

---

## 使用方式 / Usage

### 使用

```bash
/mineru-pdf
```

然后告诉它你的 PDF 路径和需求即可。Skill 会自动判断是否需要公式识别、OCR 等。

### 什么时候用它

| 信号 | 建议 |
|---|---|
| pypdf 输出的公式是 `/Cxx` 乱码 | `/mineru-pdf` |
| PDF 有数学公式、希腊字母 | `/mineru-pdf` |
| 双栏/多栏，阅读顺序需要正确 | `/mineru-pdf` |
| 需要提取图片 + 位置信息 | `/mineru-pdf` |
| 扫描件（图片型 PDF，无文字层） | `/mineru-pdf` |
| 纯文字、单栏、无公式 | 默认 pdf skill 更快 |

---

## 补充 / Notes

- MinerU **完全免费开源**（AGPL-3.0），本地运行，不消耗任何 API 额度
- 本 skill **不影响**官方 pdf skill，两者可以共存
- docx/xlsx/pptx 不需要 MinerU，原生库已经够准

---

## 相关项目 / Related

- [MinerU](https://github.com/opendatalab/MinerU) — 上海 AI Lab 开源文档解析引擎
- [anthropics/skills](https://github.com/anthropics/skills) — Claude Code 官方 skill 合集

---

<p align="center"><sub>MIT License · 欢迎 PR / issue</sub></p>
