# Usage Guide

This guide explains how to use the **Overleaf LaTeX to Word Skill** to convert an Overleaf-exported LaTeX project into an editable Microsoft Word `.docx` file.

## 1. Who This Skill Is For

Use this skill when you have an Overleaf / LaTeX manuscript and need a Word version for submission, review, editing, or collaboration.

It is especially useful for papers that contain:

- Figures
- Tables
- Mathematical formulas
- `.bib` references
- Cross-references such as `\ref{}`, `\eqref{}`, and `\cite{}`
- Multiple files loaded through `\input{}` or `\include{}`

## 2. Export from Overleaf

In Overleaf:

1. Open your LaTeX project.
2. Click **Menu** in the upper-left corner.
3. Choose **Download as ZIP**.
4. Save the ZIP file locally.

Do not only download the PDF if you want an editable Word document. The full ZIP package is better because it contains the `.tex`, figures, bibliography files, and other resources needed for conversion.

## 3. Use with an AI Agent

Upload the ZIP file and use this prompt:

```text
你是一个 LaTeX / Overleaf 文档转换专家。现在我会上传一个从 Overleaf 导出的 LaTeX 项目压缩包，请你帮我将其转换为 Word 版本。

要求：
1. 解压我上传的 Overleaf 项目压缩包；
2. 自动识别主 .tex 文件；
3. 检查 .bib 参考文献、图片路径、\input{} 和 \include{} 子文件；
4. 使用 Pandoc 或其他合适方式转换为 .docx；
5. 如有必要，先清理 Pandoc 不兼容的 LaTeX 命令；
6. 尽量保留标题、作者、摘要、关键词、标题层级、图片、表格、公式、文内引用和参考文献；
7. 转换完成后检查 Word 是否能打开，图片、公式和参考文献是否正常；
8. 最终提供可下载的 Word 文件。

不要只告诉我步骤，请直接帮我完成转换。
```

The agent should then follow [`SKILL.md`](../SKILL.md).

## 4. Local Conversion with Pandoc

If you want to run the conversion locally, install Pandoc first.

Then unzip your Overleaf project and enter the project directory:

```bash
unzip paper.zip -d paper
cd paper
```

Basic conversion:

```bash
pandoc main.tex -s -o output.docx
```

With bibliography:

```bash
pandoc main.tex \
  -s \
  --bibliography=references.bib \
  --citeproc \
  -o output.docx
```

With figure folders:

```bash
pandoc main.tex \
  -s \
  --resource-path=.:figures:figure:fig:images:image:imgs:pics:pictures \
  -o output.docx
```

With both figures and bibliography:

```bash
pandoc main.tex \
  -s \
  --from=latex \
  --to=docx \
  --resource-path=.:figures:figure:fig:images:image:imgs:pics:pictures \
  --bibliography=references.bib \
  --citeproc \
  -o output.docx
```

On Windows, `--resource-path` may use semicolons instead of colons:

```bash
--resource-path=.;figures;figure;fig;images;image;imgs;pics;pictures
```

## 5. Main File Detection

The main file is not always `main.tex`.

A main LaTeX file usually contains:

```latex
\documentclass
\begin{document}
```

You can search for it with:

```bash
grep -R "\\documentclass" .
grep -R "\\begin{document}" .
```

Common main file names include:

- `main.tex`
- `paper.tex`
- `manuscript.tex`
- `article.tex`
- `submission.tex`

## 6. Common Problems and Fixes

| Problem | Cause | Fix |
|---|---|---|
| Images missing | Pandoc cannot find the image directory | Add the correct `--resource-path` |
| References missing | `.bib` file was not passed to Pandoc | Add `--bibliography=... --citeproc` |
| Raw LaTeX commands appear | Unsupported journal template commands | Clean or simplify the LaTeX source before conversion |
| Formulas are broken | Unsupported equation environment | Simplify wrappers but keep formula content |
| Equation numbers disappear | Pandoc limitation | Reinsert equation numbers as plain text when reliable |
| Tables are misaligned | Complex `tabular`, `multirow`, or `multicolumn` | Manually adjust the table in Word |
| Content is missing | `\input{}` or `\include{}` was not resolved | Flatten the LaTeX source before conversion |
| Wrong paper converted | Wrong `.tex` file selected | Re-identify the main file |

## 7. Manual Check After Conversion

After opening the generated Word file, check:

- Title and author information
- Abstract and keywords
- Section headings
- Figure placement and captions
- Table layout and captions
- Equation display quality
- Equation numbering
- In-text citations
- Bibliography / reference list
- Cross-references to figures, tables, and equations
- Any raw LaTeX command residue

## 8. Recommended Final Reply to User

When the conversion is done, the agent should respond like this:

```text
处理好了，Word 版本在这里下载：

[下载 Word 文件](sandbox:/mnt/data/output.docx)

我已尽量保留正文结构、图片、表格、公式和参考文献。由于 LaTeX 转 Word 对复杂公式、复杂表格和交叉引用支持有限，建议重点检查公式编号、图表编号和参考文献格式。
```

## 9. Notes

LaTeX and Word use different document models. A converted Word file may require manual correction even if the conversion succeeds. The most fragile parts are usually:

- Complex equations
- Multi-line aligned formulas
- Long tables
- Tables using `multirow` or `multicolumn`
- Journal-specific templates
- Custom macros
- Cross-references
- Bibliography styles
