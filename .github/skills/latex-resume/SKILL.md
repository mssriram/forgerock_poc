---
name: latex-resume
description: "Edit and format LaTeX resume documents. Use when: editing a resume in LaTeX, formatting sections, fixing LaTeX syntax errors, improving visual layout, adding or restructuring resume content, converting plain text to LaTeX resume format, ensuring clean and compilable LaTeX output."
argument-hint: "Describe the edit or section to update (e.g. 'add new job entry', 'fix formatting in skills section')"
---

# LaTeX Resume Editor

This skill is specifically for resume editing and formatting, not general LaTeX documents.

## When to Use
- Editing or adding content to a LaTeX resume
- Converting plain text resume content into LaTeX
- Fixing LaTeX compilation errors or malformed syntax
- Restructuring or reformatting resume sections
- Improving spacing, alignment, and visual consistency
- Ensuring the document compiles cleanly with no warnings

## Guiding Principles
- **Resume-specific scope**: focus only on resume content and structure
- **Content freeze by default**: do not change, add, remove, or reword resume content unless the user explicitly asks for content updates
- **Calibri-first typography**: use Calibri as the primary font (prefer XeLaTeX/LuaLaTeX with `fontspec`)
- **No paragraph indentation**: do not use tab-style starts or paragraph indents; keep content flush-left
- **One-page target**: keep resume to one page whenever reasonably possible; allow page 2 only when content is clearly too large
- **Minimal packages**: only use packages that are necessary; prefer standard ones (`geometry`, `enumitem`, `hyperref`, `fontenc`, `inputenc`, `titlesec`, `xcolor`)
- **No hardcoded spacing hacks**: use proper LaTeX spacing commands (`\vspace`, `\medskip`, `\smallskip`) consistently rather than ad-hoc `\\[Xpt]` overrides
- **Semantic structure**: use named commands/environments for repeated patterns (e.g., a custom `\resumeentry` command) to keep the source DRY
- **ATS-friendly output**: avoid tables or complex floats for the main content; use clean lists and section headers
- **Consistent style**: all section headers, bullet points, dates, and company names must follow the same formatting pattern throughout the document

## Procedure

### 0. Set Root-File Guardrails (LaTeX Workshop)
- Add magic comments at the top of every main resume file:
  - `% !TEX root = <main-file>.tex`
  - `% !TEX program = xelatex` (preferred for Calibri)
- Ensure the root `.tex` file is the active editor when triggering VS Code build commands
- If a workspace has one primary resume file, configure LaTeX Workshop root search to include it

### 0.5 Set Resume Typography and Paragraph Behavior
- Prefer XeLaTeX or LuaLaTeX and set Calibri via `fontspec`:
  - `\usepackage{fontspec}`
  - `\setmainfont{Calibri}`
- Ensure paragraph indentation is disabled:
  - `\setlength{\parindent}{0pt}`
- Do not introduce tabs or manual paragraph indents at line starts

### 1. Understand the Edit
- Read the full `.tex` file or the relevant section before making changes
- Identify the pattern used for existing entries (e.g., how job entries, bullet points, and dates are structured)
- Classify request type before editing:
  - **Cosmetic-only** (default): formatting, spacing, alignment, typography, compile/root fixes
  - **Content update** (explicit opt-in): any wording, metrics, bullet text, section additions/removals

### 2. Apply the Change
- Match the exact indentation and formatting style of surrounding content
- For new entries (job, project, education), replicate the existing entry structure exactly
- For content from `Professional Summary.txt` or similar plain-text sources, convert to LaTeX following the document's existing patterns
- Escape special characters: `&`, `%`, `$`, `#`, `_`, `{`, `}`, `~`, `^`, `\`

### 3. Formatting Rules
- **Dates**: right-aligned on the same line as the company/role using `\hfill` or a two-column layout — never on a separate line
- **Bullet points**: use `\begin{itemize}` with `\item`; keep each bullet to 1–2 lines; start with a strong action verb
- **Experience bullet spacing**: in `Professional Experience`, use `itemsep=3pt` to keep bullets clearly separated
- **Section headers**: use the document's existing `\section` or custom header command; do not mix styles
- **Heading divider spacing**: add visible spacing below the horizontal rule after each section heading (e.g., `\titlespacing{\section}{0pt}{6pt}{5pt}`)
- **Margins**: default to `1in` all sides unless the document already defines a tighter layout
- **Font**: Calibri is required unless unavailable in the user environment
- **Font size**: default `10pt`; do not mix font sizes within sections
- **Paragraph style**: no first-line indent and no tab-started paragraphs
- **Hyperlinks**: wrap URLs and emails with `\href{}{}` from `hyperref`; set `colorlinks=true` with neutral colors

### 3.5 One-Page Compression Strategy
- First pass: tighten list spacing (`noitemsep`, small `topsep`) and remove redundant bullets
- Second pass: shorten bullet wording while preserving outcomes and metrics
- Third pass: reduce vertical whitespace between sections/entries conservatively
- Only allow a second page if the content remains materially too dense after the above steps

### 4. Validate Before Finalizing
- Check all `\begin{...}` have matching `\end{...}`
- Verify no unescaped special characters remain in text content
- Confirm no `\usepackage` duplicates in the preamble
- Ensure the preamble has `\usepackage[T1]{fontenc}` and `\usepackage[utf8]{inputenc}` (or `\usepackage{fontspec}` for LuaLaTeX/XeLaTeX)

## Common LaTeX Resume Patterns

### Job Entry Template
```latex
\textbf{Job Title} \hfill \textit{Month Year -- Month Year} \\
\textit{Company Name} \hfill \textit{Location}
\begin{itemize}[leftmargin=*, noitemsep, topsep=2pt]
  \item Accomplished [X] by doing [Y], resulting in [Z].
\end{itemize}
```

### Skills Section Template
```latex
\section*{Technical Skills}
\begin{tabular}{@{} l l}
  \textbf{Languages:} & Java, Python, SQL \\
  \textbf{Frameworks:} & Spring Boot, FastAPI \\
\end{tabular}
```

### Section Header (titlesec example)
```latex
\titleformat{\section}{\large\bfseries}{}{0em}{}[\titlerule]
```

## Quality Checklist
- [ ] Content matches the intended edit — no accidental deletions
- [ ] Special characters are properly escaped
- [ ] All environments are properly closed
- [ ] Root magic comments exist at the top of the main `.tex` file
- [ ] Calibri is configured (`fontspec` + `\setmainfont{Calibri}`) when using XeLaTeX/LuaLaTeX
- [ ] Paragraph indentation is disabled (`\setlength{\parindent}{0pt}`)
- [ ] Section headers have consistent spacing below the horizontal divider
- [ ] Resume is one page, unless content is clearly too extensive
- [ ] Dates and locations are consistently formatted
- [ ] Bullet points begin with action verbs
- [ ] Professional Experience bullet lists use `itemsep=3pt`
- [ ] No duplicate `\usepackage` declarations