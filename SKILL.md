---
name: overleaf-latex-to-word
description: Convert an Overleaf LaTeX ZIP project into an editable Microsoft Word DOCX. Use when a user uploads a LaTeX/Overleaf project and wants a Word version while preserving manuscript structure, figures, tables, equations, citations, and references as much as possible.
---

# Overleaf LaTeX to Word

## Purpose

Convert a complete Overleaf / LaTeX project into a usable Microsoft Word `.docx` file.

This skill is designed for manuscript conversion, not for simple text extraction. It should inspect the LaTeX project, locate the real main file, handle figures and bibliography resources, preprocess incompatible LaTeX commands when necessary, convert to Word, and validate the generated document.

The expected result is a downloadable `.docx` file that the user can continue editing in Microsoft Word.

---

## When to Use

Use this skill when the user asks to:

- Convert an Overleaf project to Word
- Convert a LaTeX ZIP project to `.docx`
- Export a `.tex` manuscript as Word
- Turn an Overleaf submission package into a Word document
- Fix a failed LaTeX-to-Word conversion
- Preserve formulas, figures, tables, captions, citations, and references during conversion

Do not use this skill when the user only asks how to export a PDF from Overleaf. In that case, answer with normal Overleaf export instructions.

---

## Input

The user usually uploads a `.zip` file exported from Overleaf.

The project may contain:

- Main manuscript files:
  - `main.tex`
  - `paper.tex`
  - `manuscript.tex`
  - `article.tex`
  - `submission.tex`
- Bibliography files:
  - `references.bib`
  - `refs.bib`
  - `bibliography.bib`
- Figure folders:
  - `figures/`
  - `figure/`
  - `fig/`
  - `images/`
  - `image/`
  - `imgs/`
  - `pics/`
  - `pictures/`
- Template files:
  - `*.cls`
  - `*.sty`
- Child files loaded through:
  - `\input{}`
  - `\include{}`
  - `\subfile{}`

---

## Output

Return:

1. A downloadable `.docx` file.
2. A short note explaining what was preserved.
3. A short note about conversion limitations if any.

Example final response:

```text
处理好了，Word 版本在这里下载：

[下载 Word 文件](sandbox:/mnt/data/example_Word版.docx)

我已尽量保留正文结构、图片、表格、公式和参考文献。由于 LaTeX 转 Word 对复杂公式、复杂表格和交叉引用支持有限，建议重点检查公式编号、图表编号和参考文献格式。
```

---

# Workflow

## 1. Create a Clean Workspace

Create a separate working directory. Never modify the original uploaded archive.

Recommended structure:

```text
work_latex_to_word/
├── source/
├── build/
└── output/
```

Example:

```bash
mkdir -p /mnt/data/work_latex_to_word/source
mkdir -p /mnt/data/work_latex_to_word/build
mkdir -p /mnt/data/work_latex_to_word/output
unzip input.zip -d /mnt/data/work_latex_to_word/source
```

Inspect the project:

```bash
find /mnt/data/work_latex_to_word/source -maxdepth 4 -type f
```

Look for:

- `.tex`
- `.bib`
- `.cls`
- `.sty`
- `.png`
- `.jpg`
- `.jpeg`
- `.pdf`
- `.eps`
- `.svg`

---

## 2. Identify the Main LaTeX File

Do not assume the main file is `main.tex`.

Search for files containing:

```latex
\documentclass
\begin{document}
```

Example:

```bash
grep -R "\\documentclass" /mnt/data/work_latex_to_word/source
grep -R "\\begin{document}" /mnt/data/work_latex_to_word/source
```

Choose the main file using this priority:

1. A file containing both `\documentclass` and `\begin{document}`.
2. A file with a common top-level name:
   - `main.tex`
   - `paper.tex`
   - `manuscript.tex`
   - `article.tex`
   - `submission.tex`
3. A file that imports other files through `\input{}` or `\include{}`.

If multiple candidates exist, choose the file that most clearly represents the top-level manuscript.

---

## 3. Detect Bibliography Files

Search for bibliography files:

```bash
find /mnt/data/work_latex_to_word/source -name "*.bib"
```

Inspect the main `.tex` file for:

```latex
\bibliography{...}
\addbibresource{...}
```

If bibliography files exist, pass them to Pandoc:

```bash
--bibliography=references.bib --citeproc
```

For multiple files:

```bash
--bibliography=refs1.bib --bibliography=refs2.bib --citeproc
```

If no `.bib` file exists but the source has a `thebibliography` environment, preserve that section as plain manuscript content.

---

## 4. Detect Image and Resource Paths

Search for figures and embedded resources:

```bash
find /mnt/data/work_latex_to_word/source -type f \( \
  -name "*.png" -o \
  -name "*.jpg" -o \
  -name "*.jpeg" -o \
  -name "*.pdf" -o \
  -name "*.eps" -o \
  -name "*.svg" \
\)
```

Use common paths in Pandoc:

```bash
--resource-path=.:figures:figure:fig:images:image:imgs:pics:pictures
```

On Windows, use semicolons:

```bash
--resource-path=.;figures;figure;fig;images;image;imgs;pics;pictures
```

If a paper stores figures in nested folders, include those folders too.

---

## 5. Resolve Included Files

Check whether the main file uses:

```latex
\input{section}
\include{section}
\subfile{section}
```

Pandoc may handle simple includes, but complex projects often convert better after flattening.

Recommended approach:

1. Try Pandoc directly first.
2. If content is missing, create a flattened copy of the source.
3. Replace simple `\input{}` and `\include{}` calls with the content of the referenced files.
4. Keep the original project unchanged.

---

## 6. Preprocess LaTeX for Word Conversion

Create a cleaned copy of the main LaTeX file in the build directory.

Do not delete meaningful manuscript content.

### 6.1 Remove Layout-Only Commands

These usually do not help Word conversion:

```latex
\vspace{}
\hspace{}
\newpage
\clearpage
\pagebreak
\thispagestyle{}
\pagestyle{}
\setlength{}
\renewcommand{\baselinestretch}{}
```

### 6.2 Simplify Journal Template Commands

Remove or simplify template-only metadata and running-head commands, such as:

```latex
\markboth{}{}
\runninghead{}
\correspondingauthor{}
\journal{}
\volume{}
\issue{}
\history{}
\doi{}
```

Preserve actual title, authors, abstract, keywords, section text, captions, tables, equations, and references.

### 6.3 Expand Simple Custom Macros

For simple commands:

```latex
\newcommand{\method}{CDP-MCG}
```

Replace uses of `\method` with:

```text
CDP-MCG
```

For complex macros, preserve the visible text when possible rather than leaving raw LaTeX commands in the Word file.

### 6.4 Convert Cross-References to Readable Text

Pandoc may not resolve all cross-references.

Useful fallbacks:

```latex
\ref{fig:framework}   -> [fig:framework]
\eqref{eq:loss}       -> ([eq:loss])
\autoref{tab:result}  -> [tab:result]
\cite{key}            -> [key]
\citep{key}           -> [key]
\citet{key}           -> [key]
```

If exact numbering can be inferred, preserve visible numbering:

```text
Fig. 1
Table 2
Eq. (3)
```

If exact numbering cannot be inferred, keep the label in brackets rather than deleting it.

### 6.5 Preserve Equation Blocks

Keep equation content from common environments:

```latex
\begin{equation}
...
\end{equation}
```

```latex
\begin{align}
...
\end{align}
```

```latex
\begin{cases}
...
\end{cases}
```

Do not remove formula bodies. If a wrapper is unsupported, simplify the wrapper and preserve the formula.

### 6.6 Preserve Equation Numbers

If equations contain `\tag{}` or source-visible numbering, preserve numbers such as:

```text
(1)
(2)
(3)
```

If Pandoc drops equation numbering, add the number back as nearby plain text when it can be reliably identified.

---

## 7. Convert with Pandoc

Use Pandoc as the primary conversion tool.

Basic command:

```bash
pandoc main.tex -s -o output.docx
```

Recommended command with resource paths:

```bash
pandoc cleaned_main.tex \
  -s \
  --from=latex \
  --to=docx \
  --resource-path=.:figures:figure:fig:images:image:imgs:pics:pictures \
  -o output.docx
```

Command with bibliography:

```bash
pandoc cleaned_main.tex \
  -s \
  --from=latex \
  --to=docx \
  --resource-path=.:figures:figure:fig:images:image:imgs:pics:pictures \
  --bibliography=references.bib \
  --citeproc \
  -o output.docx
```

Only include `--bibliography` options when `.bib` files exist.

---

## 8. Fallback Conversion Strategy

If Pandoc fails badly:

1. Try to compile the LaTeX project to PDF.
2. Convert the compiled PDF to Word using a PDF-to-Word workflow.
3. Use this only as a fallback.

Important limitation:

- PDF-to-Word often preserves visual layout better.
- PDF-to-Word usually produces less editable formulas, references, and structure.

Clearly tell the user if this fallback was used.

---

## 9. Validate the DOCX

Before returning the file, check:

- The `.docx` opens successfully.
- The manuscript is not empty or truncated.
- Title, authors, abstract, and keywords are present.
- Section headings are preserved.
- Figures appear.
- Figure captions appear.
- Tables are present and readable.
- Equations are present.
- Equation numbering is not obviously lost.
- References or bibliography are present.
- There are no obvious raw LaTeX commands in visible text.
- There is no serious encoding corruption.

If possible, render several pages of the DOCX and inspect:

- First page
- A page with equations
- A page with figures
- A page with tables
- A references page

---

## 10. Repair Common Problems

| Problem | Likely Cause | Fix |
|---|---|---|
| Images missing | Wrong resource path | Add correct `--resource-path` |
| References missing | `.bib` not included | Add `--bibliography` and `--citeproc` |
| Citations show raw keys | Missing `.bib` or unsupported citation command | Keep readable keys or process bibliography |
| Formulas broken | Unsupported LaTeX environment | Simplify wrapper, preserve formula body |
| Equation numbers missing | Pandoc limitation | Reinsert as plain text where reliable |
| Tables broken | Complex `tabular`, `multirow`, `multicolumn` | Simplify table or preserve readable text |
| Raw template commands visible | Journal class unsupported | Remove or simplify template commands |
| Content missing | Includes not resolved | Flatten source file |
| DOCX opens but looks empty | Wrong main file | Re-identify main `.tex` file |

---

## 11. File Naming

Use a clean output filename.

Recommended:

```text
<project_name>_Word版.docx
<project_name>_converted.docx
<project_name>_docx.docx
```

For Chinese users, prefer:

```text
论文名称_Word版.docx
```

---

## 12. Final Response

Keep the response practical.

Include:

1. Download link.
2. What was preserved.
3. What may require checking.

Example:

```text
处理好了，Word 版本在这里下载：

[下载 Word 文件](sandbox:/mnt/data/example_Word版.docx)

我已尽量保留正文结构、图片、表格、公式和参考文献。由于 LaTeX 转 Word 对复杂公式、复杂表格和交叉引用支持有限，建议重点检查公式编号、图表编号和参考文献格式。
```

---

# Rules

## Do

- Work on a copy of the source.
- Identify the real main `.tex` file before conversion.
- Include bibliography files when available.
- Include image and resource paths.
- Preserve scientific manuscript structure.
- Preserve equations as much as possible.
- Validate the `.docx` before returning it.
- Be transparent about limitations.

## Do Not

- Do not only explain the method when the user provides a file.
- Do not delete meaningful manuscript content.
- Do not assume `main.tex` is correct without checking.
- Do not ignore figures or `.bib` files.
- Do not claim 100% fidelity unless verified.
- Do not hide conversion limitations.
- Do not overwrite the original LaTeX project.

---

# Quality Bar

A successful conversion should meet these standards:

- The DOCX opens normally.
- The manuscript is complete from title to references.
- Most figures are visible.
- Most tables are readable.
- Most equations are preserved.
- Bibliography or references are present.
- The user can continue editing the document in Word.
- Known limitations are clearly stated.
