---
name: mineru-pdf
description: Use when the user wants high-quality content extraction from complex PDFs — academic papers with formulas, multi-column layouts, scanned PDFs, or any PDF where standard text extraction produces garbled text. Trigger when the user says "MinerU", "解析这篇论文", "提取PDF公式/图片/表格", or describes a PDF with unreadable formula output. For fast plain-text PDFs, the default pdf skill with pypdf is sufficient.
---

# MinerU PDF Content Extraction

## Overview

[MinerU](https://github.com/opendatalab/MinerU) is a high-accuracy document parsing engine (Shanghai AI Lab, open source). Use it for **content extraction** from complex PDFs — formulas, tables, images, multi-column layouts, scanned documents.

For PDF **editing** (merge, split, rotate, watermark, encrypt, forms), use the standard pdf skill (pypdf/qpdf/reportlab).

## Installation

```bash
pip install "mineru[pipeline]"
```

Dependencies include torch, transformers. First run auto-downloads models (~2GB, cached locally). No API keys, no network needed after initial download.

## Quick Start

```bash
mineru -p document.pdf -o output/ -b pipeline
```

Output in `output/<filename>/auto/`:

| File | Content |
|------|---------|
| `*.md` | Structured Markdown with LaTeX formulas |
| `*_content_list.json` | Per-element metadata (type, bbox, page_idx, text) |
| `images/` | All extracted images (JPG) |

## When to Use

Check pypdf output first. If it looks clean, MinerU is overkill:

```bash
python -c "from pypdf import PdfReader; print(PdfReader('doc.pdf').pages[0].extract_text()[:500])"
```

| Signal in pypdf output | Decision |
|------------------------|----------|
| `/Cxx` escape codes (formula garbled) | Use MinerU |
| Math formulas, Greek letters | Use MinerU |
| Multi-column layout (reading order broken) | Use MinerU |
| Need images with position/caption | Use MinerU |
| Scanned PDF (image-based, no text layer) | Use MinerU |
| Clean text, single column, no formulas | Stick with pypdf |

## Advanced Options

```bash
# Specify language for OCR
mineru -p doc.pdf -o output/ -b pipeline -l en

# Disable formula recognition (faster, no MFR model needed)
mineru -p doc.pdf -o output/ -b pipeline -f False

# Disable table recognition
mineru -p doc.pdf -o output/ -b pipeline -t False

# Process specific page range
mineru -p doc.pdf -o output/ -b pipeline -s 0 -e 5
```

## Reading Results

### Markdown

```python
with open("output/auto/doc.md", "r", encoding="utf-8") as f:
    markdown = f.read()
```

### JSON Metadata

```python
import json

with open("output/auto/doc_content_list.json", "r", encoding="utf-8") as f:
    elements = json.load(f)

for el in elements:
    # type: text, image, equation, table, header, footer, page_number
    # bbox: [x0, y0, x1, y1] in pixels
    # page_idx: 0-based page number
    print(f"[{el['type']}] page {el['page_idx']}: {el.get('text', '')[:100]}")
```

### Images

```python
# Images referenced in markdown as:
# ![](images/abc123.jpg)

# JSON gives bbox + page for each image:
for el in elements:
    if el['type'] == 'image':
        print(f"Image at {el['img_path']}, bbox={el['bbox']}, page={el['page_idx']}")
```

## Performance Reference

| PDF Type | Pages | Time |
|----------|-------|------|
| Simple text-only | 10 | ~1 min |
| Academic paper (formulas + figures) | 7 | ~3 min |
| Scanned book chapter | 30 | ~10-15 min |

First run adds model download time (~2GB, one-time).
