# Overleaf LaTeX to Word Skill

A reusable agent skill for converting Overleaf / LaTeX ZIP projects into editable Microsoft Word `.docx` files.

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
├── examples/
│   └── user-prompt.md
└── .gitignore
```

## Recommended Use

When the user uploads an Overleaf ZIP file and asks for a Word version, load `SKILL.md` and follow the workflow.

Typical user request:

```text
这是我的 Overleaf 导出的 LaTeX 项目 zip，请帮我转换成 Word 版本。
```

## Output

The expected output is a downloadable `.docx` file plus a short note explaining conversion quality and any parts that should be manually checked.

## Main Limitation

LaTeX-to-Word conversion is rarely 100% perfect. Complex equations, tables, custom macros, and cross-references may need manual inspection after conversion.
