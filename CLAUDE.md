# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LaTeX resume for 徐其栋 (Xu Qidong), targeting embedded systems / IoT engineering positions. Compiled with **XeLaTeX**.

## Build

```bash
# Compile the resume (run from the version directory, e.g. 15_20260628_003208/)
xelatex -interaction=nonstopmode 徐其栋求职简历.tex
```

Requires: XeLaTeX, `ctex`, `fontawesome5`, `tikz`, and CJK font `SimHei` installed system-wide.

## Repository Structure

- **Versioned snapshots** (`11_…` through `15_…` directories): Each contains a frozen copy of the `.tex` source and its compiled `.pdf`. The **highest-numbered directory is the current working version**.
- **Root `.tex` files** (`徐其栋求职简历_*_v1.tex` … `v10.tex`): Earlier iteration history before the directory-based versioning scheme.
- **Supporting docs**: Project write-ups (`机器狗项目详解.md`, `新项目_…实现手册.md`), extracted text from Word docs (`.txt` files), and an `AI 简历/` directory.

## Key Conventions

- The resume is a **single self-contained `.tex` file** — all styling, colors, and layout are inline (no external `.cls` or `.sty`).
- Language is Chinese (Simplified); all content uses `\setCJKmainfont{SimHei}`.
- Icons via `fontawesome5` (`\faPhone`, `\faEnvelope`, `\faGraduationCap`, etc.).

## Edit & Version Workflow

**每次修改简历后，必须执行以下步骤：**

1. 在根目录下创建新快照文件夹，命名格式：`{版本号}_{YYYYMMDD}_{HHMMSS}`（版本号递增，如当前最新为 `15_…`，则新文件夹为 `16_…`）
2. 将修改后的 `.tex` 文件复制到新文件夹中
3. 在新文件夹内编译：`xelatex -interaction=nonstopmode 徐其栋求职简历.tex`
4. 编译完成后新文件夹内应包含：`.tex`、`.pdf`、`.aux`、`.log`、`.out`
5. 将新文件夹中的 `.tex` 和 `.pdf` 复制到根目录，覆盖根目录的旧版本

绝不直接修改旧版本文件夹中的内容。
