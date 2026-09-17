---
name: latex-paper-suite
description: >-
  Comprehensive guide and standard operating procedure for writing, structuring,
  debugging, and compiling academic LaTeX projects in this repository (synopsis,
  review paper, research paper). Activate when the user asks to compile LaTeX,
  add chapters/sections, format tables/figures/equations, fix compilation errors,
  manage intermediate files, or scaffold new research papers.
---

# LaTeX Academic Paper Suite (Skill Runbook)

This skill equips the agent to manage, write, format, debug, and compile academic LaTeX documents within this repository in strict accordance with professional academic publishing standards and Windows/VS Code toolchains.

---

## 1. Project Architecture & Mental Model

```
paper/
├── .gitignore                     # Global LaTeX build artifact ignore rules
├── .vscode/                       # VS Code build recipes & settings (no Perl required)
│   ├── settings.json
│   └── tasks.json
├── shared/                        # Shared resources across all papers
│   ├── bibliography/              # Centralized references.bib
│   └── figures/                   # Common system diagrams & logos
├── synopsis/                      # Master university synopsis / thesis
│   ├── main.tex                   # Root document (preamble & chapter inputs)
│   ├── aktu.cls                   # University document class
│   ├── name.sty                   # Chapter title styling package
│   ├── assets/                    # Project-specific images (e.g. JSS.png)
│   ├── frontmatter/               # Cover, vision/mission, PO/PSO, cert, abstract, TOC
│   ├── chapters/                  # Chapters 01 through 08
│   ├── backmatter/                # References & appendices
│   └── archive/                   # Deprecated drafts
├── review-paper/                  # (Planned) Survey / Literature Review Paper
└── research-paper/                # (Planned) Primary Conference / Journal Paper
```

For deep architecture principles, see the [Monorepo Architecture Guide](./references/architecture.md).

---

## 2. Mandatory Rules for the AI Agent

When modifying or adding LaTeX code in this repository, the agent **MUST** follow these 7 rules:

### Rule 1: Always Use `\input`, Never `\include` for Subfiles
`\include{file}` forces LaTeX to generate an independent `.aux` file on disk for every subfile, resulting in dozens of loose files. `\input{file}` embeds content cleanly, consolidating all metadata into a single `main.aux`. Separate chapters with `\clearpage`.

### Rule 2: Always Prepend Root Magic Comments to Subfiles
Every `.tex` file inside `frontmatter/`, `chapters/`, and `backmatter/` **must** begin with:
```latex
% !TEX root = ../main.tex
```
This guarantees that VS Code's LaTeX Workshop always compiles `main.tex` and never misidentifies a subfile as standalone.

### Rule 3: Always Use `\providecommand` for Global Custom Macros
Never use `\newcommand` for macros that may be called across multiple subfiles (e.g., `\normal`, `\size`, `\bigsize`, `\bigbigsize`). Use `\providecommand` so LaTeX safely no-ops instead of halting with `Command already defined`.

### Rule 4: Never Insert `\\` After Block Environments
Line break `\\` requires an active paragraph line. Placing `\\` after `\end{center}`, `\end{flushleft}`, `\end{table}`, or `\end{enumerate}` causes fatal `! LaTeX Error: There's no line here to end.` Use `\vspace{5mm}` or standard paragraph breaks instead.

### Rule 5: Keep Output In-Place (`outDir: %DIR%`)
Do not redirect compiler output to a separate `build/` folder. Doing so detaches `main.pdf` from the root directory, causing the user to see a stale PDF. Keep `main.pdf` in the document directory, and use VS Code's `files.exclude` to hide intermediate files.

### Rule 6: Escape Special Characters in Plain Text
Characters `%`, `_`, `&`, `#`, and `$` are reserved syntax in LaTeX. They must be escaped (`\%`, `\_`, `\&`, `\#`, `\$`) whenever appearing as literal text in titles, headings, or prose.

### Rule 7: Always Perform a Two-Pass Compilation Verification
Before concluding any task, the agent must run `pdflatex -interaction=nonstopmode main.tex` twice and confirm exit code `0` so cross-references and table of contents entries are fully resolved.

---

## 3. Action Runbooks

### Runbook A: Adding a New Chapter or Section
1. Create a new `.tex` file in `chapters/` following the naming convention `0X_topic.tex`.
2. Add the root comment and sectioning content:
   ```latex
   % !TEX root = ../main.tex
   \chapter{Title of the Chapter}
   \label{chap:topic}

   \section{Introduction}
   Chapter content here...
   ```
3. Open `synopsis/main.tex` and register the chapter:
   ```latex
   \clearpage
   \input{chapters/0X_topic}
   ```
4. Run `pdflatex -interaction=nonstopmode main.tex` twice to verify.

---

### Runbook B: Inserting Figures & Tables
* **Figures:** Place images in `assets/`. Wrap in floating `figure` environment with `\centering`, `\caption`, and `\label` (always place `\label` *after* `\caption`).
  ```latex
  \begin{figure}[htbp]
    \centering
    \includegraphics[width=0.85\linewidth]{assets/my_figure.png}
    \caption{Description of the architecture.}
    \label{fig:my_figure}
  \end{figure}
  ```
* **Tables:** Use `\begin{table}[htbp]` with `\centering`, `\caption`, and a `tabular` environment.
* Reference in prose with `Figure~\ref{fig:my_figure}` or `Table~\ref{tab:results}`.

---

### Runbook C: Adding Citations & References
1. Add the BibTeX entry to `shared/bibliography/references.bib` or `backmatter/references.tex`.
2. In text, cite with `\cite{citation_key}`.
3. Use non-breaking spaces before citations: `Algorithm proposed by Smith et al.~\cite{smith2024}`.

---

### Runbook D: Troubleshooting Compilation Errors
When a user reports a build error:
1. View `main.log` or run `pdflatex -interaction=nonstopmode main.tex` from the document directory.
2. Check against the [LaTeX Troubleshooting Catalog](./references/troubleshooting.md):
   * `There's no line here to end` $\rightarrow$ Remove stray `\\`.
   * `Runaway argument` $\rightarrow$ Look for unclosed `{` or nested section commands.
   * `Command already defined` $\rightarrow$ Change `\newcommand` to `\providecommand`.
   * `Undefined control sequence` $\rightarrow$ Check package imports or use standard tabular tables.
3. Fix the syntax, recompile, and ensure exit code `0`.

---

### Runbook E: Scaffolding a New Paper (Review Paper / Research Paper)
When the user wants to start a review or research paper:
1. Create a dedicated folder: `paper/review-paper/` or `paper/research-paper/`.
2. Copy the relevant template (`IEEEtran.cls`, `acmart.cls`, or standard `article`).
3. Create `main.tex` with root structure and modular `sections/` directory.
4. Link to the shared bibliography: `\bibliography{../shared/bibliography/references}`.
5. Create `.vscode/tasks.json` entry for one-click compilation.

---

## 4. Reference Documentation Index

For complete step-by-step guides, consult the reference files:
* [VS Code & LaTeX Workshop Setup Guide](./references/vscode_setup.md): Complete setup without Perl dependencies, SyncTeX shortcuts, and editor preferences.
* [Troubleshooting & Error Catalog](./references/troubleshooting.md): Complete catalog of specific LaTeX errors, causes, and verified solutions.
* [Monorepo Architecture & Multi-Paper Best Practices](./references/architecture.md): Guidelines for structuring multi-document academic suites.
* [LaTeX Beginner Cheat Sheet & Style Guide](./references/latex_cheatsheet.md): Essential reference for chapters, tables, math, and escaping rules.
