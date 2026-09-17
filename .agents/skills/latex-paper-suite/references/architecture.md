# Multi-Document LaTeX Architecture & Monorepo Best Practices

This document outlines the standard architecture for managing academic paper suites in a single workspace.

---

## 1. Monorepo Directory Layout

When preparing academic research, multiple documents are usually needed:
1. **Academic Synopsis** (University submission format, e.g. AKTU thesis)
2. **Literature Review Paper** (Survey format for journals/conferences)
3. **Research Paper** (Primary conference/journal submission, e.g. IEEEtran, Springer LNCS, ACM)
4. **Defense Slides** (Beamer presentation)

To prevent template collision and cross-talk, each document resides in its own self-contained directory with a shared resource layer:

```
paper/
├── .gitignore                     # Global rule: ignores all build junk across all folders
├── .vscode/                       # Workspace-wide VS Code editor & build config
│   ├── settings.json
│   └── tasks.json
│
├── shared/                        # Shared assets and data across papers
│   ├── bibliography/
│   │   └── references.bib         # Master BibTeX database for ALL papers
│   └── figures/                   # Common diagrams, architecture charts, logos
│
├── synopsis/                      # Document 1: University Synopsis
│   ├── main.tex                   # Master file
│   ├── aktu.cls                   # University class
│   ├── name.sty                   # Styling package
│   ├── assets/                    # Project-specific images
│   ├── frontmatter/               # Cover, declaration, abstract, TOC
│   ├── chapters/                  # Chapters 01 through 08
│   └── backmatter/                # References & appendices
│
├── review-paper/                  # Document 2: Survey / Review Paper
│   ├── main.tex
│   ├── IEEEtran.cls (or template)
│   ├── sections/
│   └── figures/
│
└── research-paper/                # Document 3: Primary Research Paper
    ├── main.tex
    ├── sections/
    └── figures/
```

---

## 2. `\input` vs `\include`: Why `\input` is Essential

In standard LaTeX:
* **`\include{filename}`**:
  1. Issues an automatic `\clearpage`.
  2. Opens an independent auxiliary stream and generates a separate `filename.aux` file on disk for every single file.
  3. In a 15-section thesis, `\include` litters the directory with 15 separate `.aux` files.
* **`\input{filename}`**:
  1. Directly reads and typesets the content as if it were typed inline.
  2. Consolidates all labels, citations, and counters into the **single** `main.aux` file.
  3. Combined with explicit `\clearpage` between chapters, you get identical visual layout without directory clutter.

**Rule:** Always use `\input` for subfiles.

---

## 3. Centralized Bibliography (`references.bib`)

Instead of maintaining separate bibliographies across the synopsis, review paper, and research paper:
1. Store all citation entries in `shared/bibliography/references.bib`.
2. In each document's `main.tex`, reference the shared file:
   ```latex
   \bibliographystyle{IEEEtran}     % or unsrt, plain
   \bibliography{../shared/bibliography/references}
   ```
3. When adding a new paper citation, add it once to `references.bib`—it becomes available immediately to all projects.

---

## 4. Image & Asset Management

* Always place figures in an `assets/` or `figures/` subfolder.
* Declare `\graphicspath{{assets/}}` in the preamble.
* In the document body, reference the image by filename only:
  ```latex
  \includegraphics[width=0.8\textwidth]{system_architecture.png}
  ```
  LaTeX will search the `assets/` directory automatically.

---

## 5. Git Tracking Rules

* **Tracked:**
  * `.tex` source files
  * `.bib` bibliography databases
  * `.cls` document classes and `.sty` style packages
  * Image assets (`.png`, `.jpg`, `.pdf`, `.svg`)
  * `.vscode/settings.json` and `tasks.json`
* **Untracked (Ignored in `.gitignore`):**
  * `.aux`, `.log`, `.toc`, `.lof`, `.lot`
  * `.fls`, `.fdb_latexmk`
  * `.synctex.gz`
  * `build/`, `out/`
