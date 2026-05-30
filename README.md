# Overleaf LaTeX to Word Skill

A reusable agent skill for converting Overleaf / LaTeX ZIP projects into editable Microsoft Word `.docx` files.

This repository stores a standard `SKILL.md` that can be used by an AI agent when a user uploads an Overleaf-exported LaTeX project and asks for a Word version.

## What It Does

This skill guides an AI agent through a reliable LaTeX-to-Word conversion workflow:

- Unpack an Overleaf ZIP project
- Identify the real main `.tex` file
- Detect `.bib` bibliography files
- Detect image and resource folders
- Resolve `\input{}` / `\include{}` child files
- Preprocess unsupported LaTeX commands
- Convert with Pandoc
- Validate the generated DOCX
- Repair common conversion issues

## Repository Structure

```text
.
├── SKILL.md
├── README.md
├── docs/
│   └── usage.md
├── examples/
│   └── user-prompt.md
└── .gitignore
```

## Quick Start

### 1. Export your Overleaf project

In Overleaf:

1. Open your project.
2. Click **Menu**.
3. Choose **Download as ZIP**.
4. Save the exported `.zip` file locally.

### 2. Give the ZIP file to an AI agent

Use the prompt in [`examples/user-prompt.md`](examples/user-prompt.md), or simply say:

```text
这是我的 Overleaf 导出的 LaTeX 项目 zip，请帮我转换成 Word 版本。
```

### 3. Let the agent follow `SKILL.md`

The agent should:

1. Extract the ZIP package.
2. Find the main `.tex` file.
3. Detect figures and bibliography files.
4. Convert the project to `.docx`.
5. Check whether the Word file opens correctly.
6. Return a downloadable Word file.

## Recommended Use

Use this skill when the user asks to:

- Convert an Overleaf project to Word
- Convert a LaTeX ZIP project to `.docx`
- Export a `.tex` manuscript as Word
- Preserve formulas, figures, tables, citations, and references during conversion

Do not use this skill for simple PDF export instructions. It is meant for actual LaTeX-to-Word conversion.

## Detailed Usage Guide

See the full guide here:

[docs/usage.md](docs/usage.md)

The guide includes:

- Required tools
- How to export from Overleaf
- How to use the skill with ChatGPT / Codex-style agents
- Local Pandoc command examples
- Common conversion problems and fixes
- Manual checking checklist after conversion

## Expected Output

The expected output is a downloadable `.docx` file plus a short note explaining conversion quality and any parts that should be manually checked.

Example:

```text
处理好了，Word 版本在这里下载：

[下载 Word 文件](sandbox:/mnt/data/example_Word版.docx)

我已尽量保留正文结构、图片、表格、公式和参考文献。由于 LaTeX 转 Word 对复杂公式、复杂表格和交叉引用支持有限，建议重点检查公式编号、图表编号和参考文献格式。
```

## Main Limitation

LaTeX-to-Word conversion is rarely 100% perfect. Complex equations, tables, custom macros, journal templates, and cross-references may need manual inspection after conversion.
