---
name: mineru-pdf
description: >
  High-accuracy PDF content extraction using MinerU (Shanghai AI Lab).
  Use this whenever the user needs to extract text, formulas, tables, or images
  from a complex PDF — especially academic papers, multi-column layouts,
  scanned documents, or any PDF where pypdf produces garbled /Cxx formula output.
  Trigger on: "MinerU", "解析这篇论文", "提取PDF公式", "这个PDF乱码了",
  "扫描件OCR", "双栏PDF", or any request involving PDF formula/image extraction.
  Even if the user doesn't say "MinerU" explicitly, suggest it when they
  complain about garbled PDF text or need high-quality extraction.
  For simple single-column text-only PDFs, the default pdf skill is faster.
---

# MinerU PDF Extraction

## Overview

[MinerU](https://github.com/opendatalab/MinerU) is a document parsing engine that handles what pypdf/pdfplumber cannot: math formulas, reading order in multi-column layouts, image extraction with position metadata, and scanned PDFs. It runs entirely locally (no API keys, no network after model download).

This skill is for **content extraction only**. For PDF editing (merge, split, rotate, watermark, encrypt, forms), defer to the standard pdf skill.

## Workflow

### 1. Check installation

```bash
pip show mineru
```

If not installed:
```bash
pip install "mineru[pipeline]"
```

### 2. Quick probe (optional)

Run pypdf on the first page to confirm MinerU is actually needed:

```bash
python -c "from pypdf import PdfReader; print(PdfReader('FILE').pages[0].extract_text()[:500])"
```

If the output is clean (no `/Cxx` codes, normal text), tell the user pypdf might be enough. If it's garbled, proceed with MinerU.

### 3. Run MinerU

```bash
mineru -p "FILE" -o "OUTPUT_DIR/" -b pipeline
```

Key options (explain to user when relevant):

| Flag | When to use |
|------|-------------|
| `-l en` | Non-Chinese documents (default is `ch`) |
| `-f False` | Skip formula recognition to save time when no formulas |
| `-t False` | Skip table recognition when no tables |
| `-s N -e M` | Process only a page range |

### 4. Report results

After extraction completes, summarize for the user:

- How many pages processed
- Output directory and key files
- Count of images, equations, tables extracted
- Flag anything unusual (e.g., empty pages, missing content)

## Output Structure

```
OUTPUT_DIR/<filename>/auto/
├── <filename>.md              # Structured Markdown
├── <filename>_content_list.json  # Element metadata
├── <filename>_model.json      # Layout detection results
├── <filename>_middle.json     # Intermediate processing data
└── images/                    # Extracted images (JPG)
```

The Markdown preserves document structure (headings, reading order, image references). The JSON provides per-element `type`, `bbox`, `page_idx`, and `text` — useful for downstream processing.

## Performance

First run downloads models (~2GB) from HuggingFace, cached permanently. Subsequent runs skip this step.

| PDF Type | ~Time |
|----------|-------|
| 10-page text-only | 1 min |
| 7-page academic paper (formulas + figures) | 3 min |
| 30-page scanned chapter | 10-15 min |

CPU-only environments work but are slower. A GPU speeds up layout detection and formula recognition significantly.
