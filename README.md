<h1 align="center">🚀 MinerU × Claude Code PDF Skill</h1>
<p align="center"><b>用 MinerU 替换 pypdf/pdfplumber 做 PDF 内容解析，官方 pdf skill 的精准度质变</b></p>
<p align="center">Replace pypdf/pdfplumber with MinerU for PDF content extraction — a quantum leap in accuracy</p>

---

## 目录 / TOC

- [问题 / Problem](#问题--problem)
- [方案 / Solution](#方案--solution)
- [效果对比 / Before & After](#效果对比--before--after)
- [小白教程 / Step-by-Step](#小白教程--step-by-step)

---

## 问题 / Problem

Claude Code 官方 `pdf` skill（来自 [anthropics/skills](https://github.com/anthropics/skills)）文本提取依赖 **pypdf + pdfplumber**，在复杂文档上有明显短板：

| 问题 | 实测表现 |
|---|---|
| 数学公式 | pypdf 提取为乱码（`PrðfðXÞ/C0 ...` 不可用） |
| 图片 | pdfplumber/pypdf 无法提取，pdfimages 只出二进制无位置信息 |
| 输出格式 | 纯文本，丢失文档结构（标题层级、阅读顺序、图文关联） |
| 扫描件 | 完全依赖 pytesseract，配置繁琐且效果一般 |

---

## 方案 / Solution

**内容解析全部交给 [MinerU](https://github.com/opendatalab/MinerU)**（上海 AI Lab 开源的文档解析引擎），编辑操作（合并/拆分/加水印/加密）继续用 pypdf/qpdf。

```
解析 PDF → 无脑 MinerU
编辑 PDF → pypdf/qpdf
```

就这么简单。完整改动见 [pdf-SKILL-modified.md](pdf-SKILL-modified.md)，diff 见 [SKILL-diff.patch](SKILL-diff.patch)。

---

## 效果对比 / Before & After

以一篇双栏学术论文（7 页，含大量公式、表格、图片）实测：

| 维度 | pypdf | MinerU |
|---|---|---|
| 公式提取 | `PrðfðXÞ/C0 /C22 /C21 /C28 Þ/C20 ...` 乱码 | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau) \leq ...` 可编译 LaTeX |
| 图片提取 | 无 | ✅ 21 张图片 + bbox 坐标 + 图注文字 |
| 耗时 | <1 秒 | ~3 分钟（首次需下载模型） |
| 输出 | 纯文本字符串 | Markdown + JSON + 图片 |

> 表格中的 pypdf 公式输出为实际提取结果，MinerU 公式输出也是真实解析内容。

---

## 小白教程 / Step-by-Step

### 第一步：安装 MinerU

```bash
pip install "mineru[pipeline]"
```

> 依赖较多（torch、transformers 等），耐心等待 3-5 分钟。

### 第二步：下载模型（首次必须）

MinerU 首次运行会自动从 ModelScope/HuggingFace 下载模型文件（~2GB），**下载一次后永久缓存**，后续毫秒级启动。

找任意一个 PDF 跑一次即可触发下载：

```bash
# 随便找一个 PDF，跑完后模型就下载好了
mineru -p 任意文件.pdf -o ./test_output/ -b pipeline
```

> 这一步耗时取决于网速，通常 5-15 分钟。看到进度条跑完就说明模型下载完成。

### 第三步：替换 SKILL.md

用本仓库的 `pdf-SKILL-modified.md` 替换官方 pdf skill 文件：

**主文件：**
```
<Claude Code 安装目录>\plugins\marketplaces\anthropic-agent-skills\skills\pdf\SKILL.md
```

**缓存文件（如有，同步替换）：**
```
<Claude Code 安装目录>\plugins\cache\anthropic-agent-skills\document-skills\<hash>\skills\pdf\SKILL.md
```

> 文件名必须叫 `SKILL.md`（Claude Code 强制要求），外层目录可以改。

### 第四步：重启 Claude Code

重启后 AI 处理任何 PDF 解析请求就会自动走 MinerU。

---

## 补充 / Notes

- MinerU **完全免费开源**（AGPL-3.0），本地运行，不消耗任何 API 额度
- 仅 pdf skill 值得这样改——docx/xlsx/pptx 本身是结构化格式，原生库已经够准
- 如果官方 marketplace 更新覆盖了你的修改，重新替换 SKILL.md 即可（一行命令）

---

## 相关项目 / Related

- [MinerU](https://github.com/opendatalab/MinerU) — 一站式开源文档解析工具
- [anthropics/skills](https://github.com/anthropics/skills) — Claude Code 官方 skill 合集
- [pandoc](https://pandoc.org/) — Markdown → docx 转换（LaTeX 公式自动变 Word 原生公式）

---

<p align="center"><sub>MIT License · 欢迎 PR / issue</sub></p>
