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

以双栏学术论文 [Statistical Region Merging (PAMI 2004)](https://ieeexplore.ieee.org/document/1262169) 为例：

| 维度 | pypdf | MinerU |
|---|---|---|
| 文字 | 双栏顺序混乱 | ✅ 正确阅读顺序 |
| 公式 | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau)` 散落 | ✅ LaTeX 公式完整保留 |
| 表格 | 文本流拼接 | ✅ HTML 结构化表格 |
| 图片 | 无法提取 | ✅ 21 张图片 + bbox 坐标 + 图注 |
| 耗时 | <1 秒 | ~3 分钟 |
| 输出 | 纯文本 | Markdown + JSON + 图片 |

**MinerU 输出后可以用 pandoc 一键转 docx，公式自动变 Word 原生公式。**

---

## 修改步骤 / How to Replicate

### 1. 安装 MinerU

```bash
pip install "mineru[pipeline]"
```

### 2. 找到 pdf skill 文件

如果你已安装官方文档四件套，skill 文件在：

```
C:\Users\<用户名>\.claude\plugins\marketplaces\anthropic-agent-skills\skills\pdf\SKILL.md
```

如果是通过 marketplace 安装的，缓存也需要同步：

```
C:\Users\<用户名>\.claude\plugins\cache\anthropic-agent-skills\document-skills\<hash>\skills\pdf\SKILL.md
```

### 3. 修改 SKILL.md

修改三个位置：

**a) Overview 里加入 MinerU 优先声明（替换原有提取逻辑）**

```markdown
### Content Extraction → Always MinerU

**All PDF content extraction (text, tables, formulas, images) goes through MinerU.**

```bash
mineru -p document.pdf -o output/ -b pipeline

# Output:
#   auto/*.md                — structured Markdown (LaTeX formulas, HTML tables)
#   auto/*_content_list.json — per-element metadata (type, bbox, caption, page_idx)
#   auto/images/             — extracted images
```

All editing operations (merge, split, watermark, forms, encrypt) use the pypdf/qpdf sections below.
```

**b) 把 pdfplumber Text/Table Extraction 部分换成 MinerU**

```markdown
### Text/Table/Image Extraction → MinerU (Preferred)

**Use MinerU for all content extraction.** It handles complex layouts (multi-column), 
formulas (→ LaTeX), tables (→ HTML), scanned PDFs (→ OCR), and extracts images with 
position metadata (bbox, page_idx, caption).
```

**c) Quick Reference 表第一行改为 MinerU**

```markdown
| Extract text/tables/formulas/images | **MinerU** | `mineru -p doc.pdf -o output/` |
```

去掉 pdfplumber 提取、pytesseract OCR 等行，保留 pypdf/qpdf 编辑相关行。

### 4. 同步缓存 + 重启 Claude Code

```bash
cp skills/pdf/SKILL.md cache/.../skills/pdf/SKILL.md
# 重启 Claude Code
```

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
