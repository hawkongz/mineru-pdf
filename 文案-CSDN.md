# Claude Code 复杂 PDF 解析神器 MinerU：公式不再乱码，图片精准提取，三步上手

## 背景：Claude Code 读 PDF 的硬伤

如果你常用 Claude Code 分析 PDF，大概率遇到过这些情况：

- 论文里的数学公式提取出来是一串 `/C22 /C21 /C28` 乱码
- 双栏排版的 PDF 读出来段落顺序全乱
- 图表完全提取不出来
- 扫描件 PDF 直接罢工

原因很简单：Claude Code 默认 pdf skill 底层是 **pypdf + pdfplumber**。这两个库处理纯文本文档完全没问题，但面对公式、多栏排版、图片型 PDF，它们的文本层提取能力就捉襟见肘了。

## MinerU PDF 是什么

**MinerU PDF** 是一个 Claude Code 独立 skill，为 Claude Code 补充高精度 PDF 内容解析能力。底层使用上海人工智能实验室开源的 [MinerU](https://github.com/opendatalab/MinerU) 文档解析引擎。

它**不替换官方 pdf skill**——编辑操作（合并、拆分、加水印、加密）仍然走官方 skill。只有内容解析遇到公式、图片、复杂排版时，才显式调用 `/mineru-pdf`。两者互补，互不冲突。

## 工作原理

MinerU 引擎采用 pipeline 架构处理 PDF：

1. **布局分析** — 识别页面中的文本块、图片、表格、公式区域，并确定正确的阅读顺序
2. **公式识别** — 对数学公式区域进行 OCR + LaTeX 渲染，输出可编译的 LaTeX 代码
3. **图片提取** — 提取所有图片并保存为独立文件，同时记录 bbox 坐标和关联图注
4. **结构化输出** — 最终输出 Markdown（带标题层级和图文关联）+ JSON（结构化数据）+ 图片文件

整个过程完全在本地运行，模型下载后不需要联网，不消耗任何 API 额度。

## 实测对比：同一篇论文，两种引擎

测试对象：一篇双栏学术论文（PAMI 2004，7 页，含大量公式和插图）

| 维度 | pypdf（默认 pdf skill） | MinerU（本 skill） |
|---|---|---|
| **公式提取** | `PrðfðXÞ/C0 /C22 /C21 /C28 Þ/C20 ...` 乱码，完全不可用 | `\mathrm{Pr}(f(\mathbf{X})-\mu\geq\tau) \leq ...` 可编译 LaTeX |
| **图片提取** | 无法提取图片 | 21 个图片文件（13 个正文插图 + 8 个图元）+ bbox 坐标 + 图注文字 |
| **输出格式** | 纯文本字符串 | Markdown + JSON + 图片文件 |
| **耗时** | < 1 秒 | ~ 3 分钟（首次需下载模型，之后更快） |

> 以上数据均为本仓库在真实论文上的实测结果，可复现。

## 安装与使用（三步）

**前置要求：** Python 3.8+，Claude Code 已安装

### 第一步：安装 MinerU 引擎

```bash
pip install "mineru[pipeline]"
```

依赖 torch、transformers 等，首次安装约 3-5 分钟。

国内用户如果 HuggingFace 下载慢，可以切到 ModelScope 镜像：

```bash
# macOS / Linux
export MINERU_MODEL_SOURCE=modelscope

# Windows PowerShell
$env:MINERU_MODEL_SOURCE="modelscope"
```

### 第二步：安装 Skill 到 Claude Code

**macOS / Linux：**

```bash
mkdir -p ~/.claude/skills/mineru-pdf && curl -o ~/.claude/skills/mineru-pdf/SKILL.md https://raw.githubusercontent.com/hawkongz/mineru-pdf/master/SKILL.md
```

**Windows（PowerShell）：**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\mineru-pdf"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/hawkongz/mineru-pdf/master/SKILL.md" -OutFile "$env:USERPROFILE\.claude\skills\mineru-pdf\SKILL.md"
```

### 第三步：开始使用

重启 Claude Code，输入：

```text
/mineru-pdf 解析这个 PDF
```

首次使用会自动下载模型（约 2GB），之后缓存本地，不再需要网络。

## 什么时候用它

| 信号 | 建议 |
|---|---|
| pypdf 输出的公式是 `/Cxx` 乱码 | 用 `/mineru-pdf` |
| PDF 含数学公式、希腊字母 | 用 `/mineru-pdf` |
| 双栏 / 多栏，需要正确阅读顺序 | 用 `/mineru-pdf` |
| 需要提取图片 + 位置信息 | 用 `/mineru-pdf` |
| 扫描件（图片型 PDF，无文字层） | 用 `/mineru-pdf` |
| 纯文字、单栏、无公式 | 默认 pdf skill 更快 |

简单说：**日常 PDF 用默认，复杂 PDF 切 MinerU。**

## 费用与协议

- **MinerU PDF skill：** MIT 协议，完全免费
- **MinerU 引擎：** AGPL-3.0 协议（使用时注意合规）
- **运行方式：** 完全本地，无 API Key，无调用次数限制

## 项目地址

GitHub：[github.com/hawkongz/mineru-pdf](https://github.com/hawkongz/mineru-pdf)

如果这个项目帮你解决了 PDF 解析的痛点，欢迎 Star 支持。
