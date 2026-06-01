---
name: mineru-pdf
description: >
  使用 MinerU（上海 AI Lab）进行高精度 PDF 内容提取。
  当用户需要从复杂 PDF 中提取文本、公式、表格或图片时使用此 skill —
  尤其是学术论文、多栏排版、扫描文档，或任何 pypdf 产生 /Cxx 乱码的 PDF。
  触发词："MinerU"、"解析这篇论文"、"提取 PDF 公式"、"这个 PDF 乱码了"、
  "扫描件 OCR"、"双栏 PDF"，或任何涉及 PDF 公式 / 图片提取的请求。
  即使用户没有明确说 "MinerU"，当他们抱怨 PDF 文本乱码或需要高质量提取时，
  主动建议使用此 skill。对于简单的单栏纯文字 PDF，默认 pdf skill 更快。
---

# MinerU PDF 内容提取

## 概述

[MinerU](https://github.com/opendatalab/MinerU) 是一个文档解析引擎，能处理 pypdf/pdfplumber 无法胜任的场景：数学公式、多栏排版的阅读顺序、带位置元数据的图片提取、扫描件 OCR。完全本地运行（无需 API Key，模型下载后无需网络）。

本 skill 仅用于**内容提取**。PDF 编辑操作（合并、拆分、旋转、水印、加密、表单）请使用标准 pdf skill。

## 工作流程

### 1. 检查安装

```bash
pip show mineru
```

如未安装：
```bash
pip install "mineru[pipeline]"
```

首次运行会自动下载模型（~2 GB）。如果 HuggingFace 下载慢或被墙，切换到 ModelScope（国内镜像）：
```bash
export MINERU_MODEL_SOURCE=modelscope   # macOS / Linux
$env:MINERU_MODEL_SOURCE="modelscope"   # Windows PowerShell
```
设为 ModelScope 后从 `modelscope.cn` 下载，国内速度快很多。这个环境变量只需在首次运行前设置一次。

### 2. 运行 MinerU

```bash
mineru -p "文件路径" -o "输出目录/" -b pipeline
```

关键选项（按需向用户说明）：

| 参数 | 使用场景 |
|:---|:---|
| `-l en` | 非中文文档（默认 `ch`） |
| `-f False` | 无公式时跳过公式识别，节省时间 |
| `-t False` | 无表格时跳过表格识别 |
| `-s N -e M` | 只处理指定页码范围 |

### 3. 报告结果

提取完成后，向用户总结：

- 处理了多少页
- 输出目录和关键文件
- 提取到的图片、公式、表格数量
- 标注任何异常（空页、缺失内容等）

## 输出结构

```
输出目录/<文件名>/auto/
├── <文件名>.md                  # 结构化 Markdown
├── <文件名>_content_list.json   # 元素元数据
├── <文件名>_model.json          # 版面检测结果
├── <文件名>_middle.json         # 中间处理数据
└── images/                      # 提取的图片（JPG）
```

Markdown 保留了文档结构（标题层级、阅读顺序、图片引用）。JSON 提供每个元素的 `type`、`bbox`、`page_idx` 和 `text`，方便下游处理。

## 性能参考

首次运行从 HuggingFace 下载模型（~2 GB），永久缓存。后续运行跳过此步骤。

| PDF 类型 | 约需时间 |
|:---|:---|
| 10 页纯文字 | 1 分钟 |
| 7 页学术论文（公式 + 图片） | 3 分钟 |
| 30 页扫描章节 | 10-15 分钟 |

纯 CPU 环境也可运行，但较慢。GPU 可显著加速版面检测和公式识别。
