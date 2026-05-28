<h1 align="center">🚀 MinerU × Claude Code PDF Skill</h1>
<p align="center"><b>用 MinerU 替换 pypdf/pdfplumber 做 PDF 内容解析，官方 pdf skill 的精准度质变</b></p>
<p align="center">Replace pypdf/pdfplumber with MinerU for PDF content extraction — a quantum leap in accuracy</p>

---

## 目录 / TOC

- [问题 / Problem](#问题--problem)
- [方案 / Solution](#方案--solution)
- [效果对比 / Before & After](#效果对比--before--after)
- [修改步骤 / How to Replicate](#修改步骤--how-to-replicate)
- [最终 SKILL.md / Final SKILL.md](#最终-skillmd--final-skillmd)

---

## 问题 / Problem

Claude Code 官方 `pdf` skill（来自 [anthropics/skills](https://github.com/anthropics/skills)）文本提取依赖 **pypdf + pdfplumber**，遇到复杂 PDF 就跪了：

| 场景 | pypdf/pdfplumber |
|---|---|
| 双栏论文 | 文字按物理位置拼接，阅读顺序全乱 |
| 数学公式 | 乱码或直接丢失 |
| 扫描件 | 完全无法识别 |
| 表格 | 变成散落文本 |
| 图片 | 只能提取二进制，无位置信息 |

---

## 方案 / Solution

**内容解析全部交给 [MinerU](https://github.com/opendatalab/MinerU)**（上海 AI Lab 开源的文档解析引擎），编辑操作（合并/拆分/加水印/加密）继续用 pypdf/qpdf。

修改只有一个原则：

```
解析 PDF → 无脑 MinerU
编辑 PDF → pypdf/qpdf
```

修改后的 SKILL.md 核心变化：

```diff
- ## Quick Start
- reader = PdfReader("document.pdf")
- text = page.extract_text()    # ← pypdf
+ ## Quick Start
+ ### Content Extraction → Always MinerU
+ mineru -p document.pdf -o output/ -b pipeline
+
+ ### PDF Editing → pypdf/qpdf
+ # (unchanged: merge, split, watermark, forms, encrypt)
```

Quick Reference 表变化：

| 任务 | 之前 | 之后 |
|---|---|---|
| Extract text | pdfplumber | **MinerU** |
| Extract tables | pdfplumber | **MinerU** |
| OCR scanned PDFs | pytesseract | **MinerU** |
| Extract images | pdfimages | **MinerU**（含 bbox/caption/page_idx） |
| Merge / Split / Rotate / Watermark | pypdf/qpdf | pypdf/qpdf（不变） |

---

## 效果对比 / Before & After

以一篇双栏学术论文（7 页，含大量公式、表格、图片）实测为例：

| 维度 | pypdf | MinerU |
|---|---|---|
| 文字 | 双栏顺序混乱，碎片化 | ✅ 正确阅读顺序 |
| 公式 | LaTeX 源码散落、格式错乱 | ✅ 结构化 LaTeX 公式完整保留 |
| 表格 | 文本流拼接，行列错位 | ✅ HTML 结构化表格 |
| 图片 | 无法提取 | ✅ 21 张图片 + 精确 bbox 坐标 + 图注文字 |
| 耗时 | <1 秒 | ~3 分钟（首次需下载模型） |
| 输出 | 纯文本字符串 | Markdown + JSON + 图片 |

> 实测过程与本仓库的 diff 修改一一对应，MinerU 输出可直接用 pandoc 转为 Word（公式自动渲染为 Word 原生公式）。

---

## 使用方法 / How to Use

### 1. 安装 MinerU

```bash
pip install "mineru[pipeline]"
```

### 2. 替换官方 SKILL.md

用本仓库的 `pdf-SKILL-modified.md` 覆盖官方 pdf skill 文件：

```
skills/pdf/SKILL.md          # 主文件
cache/.../skills/pdf/SKILL.md # 缓存（如有，同步覆盖）
```

### 3. 重启 Claude Code

---

## 补充 / Notes

- MinerU **完全免费开源**（AGPL-3.0），本地运行，不消耗任何 API 额度
- 第一次运行会下载模型（~2GB，从 ModelScope/HuggingFace），后续秒启动
- 仅 pdf skill 值得这样改——docx/xlsx/pptx 本身是结构化格式，原生库已经够准
- 如果官方 marketplace 更新覆盖了你的修改，重新改一次即可（改动量很小）

---

## 相关项目 / Related

- [MinerU](https://github.com/opendatalab/MinerU) — 一站式开源文档解析工具
- [anthropics/skills](https://github.com/anthropics/skills) — Claude Code 官方 skill 合集
- [pandoc](https://pandoc.org/) — Markdown → docx 转换（LaTeX 公式自动变 Word 原生公式）

---

<p align="center"><sub>MIT License · 欢迎 PR / issue</sub></p>
