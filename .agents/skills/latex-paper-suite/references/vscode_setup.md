# VS Code + LaTeX Workshop Setup Guide

This guide details the exact environment configuration, toolchains, and editor settings required for seamless LaTeX editing and compilation in this workspace on Windows without external dependencies like Perl.

---

## 1. The Windows / MiKTeX Toolchain Reality

On Windows, **MiKTeX** includes `pdflatex.exe`, `xelatex.exe`, and `latexmk.exe`. However:
* `latexmk.exe` is a Perl script wrapper. By default, MiKTeX does **not** bundle a Perl interpreter.
* Attempting to run `latexmk` without a separate ActivePerl or Strawberry Perl installation crashes with:
  ```text
  MiKTeX could not find the script engine 'perl' which is required to execute 'latexmk'.
  ```
* **The Solution:** Configure VS Code's **LaTeX Workshop** extension to use **`pdflatex` directly**, eliminating any dependency on Perl.

---

## 2. Workspace Configuration (`.vscode/settings.json`)

The following settings must be present in `.vscode/settings.json` at the repository root (`paper/.vscode/settings.json`):

```json
{
  "latex-workshop.latex.tools": [
    {
      "name": "pdflatex",
      "command": "pdflatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "%DOC%"
      ],
      "env": {}
    }
  ],
  "latex-workshop.latex.recipes": [
    {
      "name": "pdflatex ➞ pdflatex",
      "tools": [
        "pdflatex",
        "pdflatex"
      ]
    },
    {
      "name": "pdflatex",
      "tools": [
        "pdflatex"
      ]
    }
  ],
  "latex-workshop.latex.outDir": "%DIR%",
  "latex-workshop.view.pdf.viewer": "tab",
  "latex-workshop.latex.autoBuild.run": "onSave",
  "latex-workshop.latex.rootFile.relativePath": "synopsis/main.tex",
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true,
    "**/Thumbs.db": true,
    "**/build": true,
    "**/*.aux": true,
    "**/*.toc": true,
    "**/*.lof": true,
    "**/*.lot": true,
    "**/*.fls": true,
    "**/*.fdb_latexmk": true,
    "**/*.log": true,
    "**/*.synctex.gz": true,
    "**/*.out": true
  }
}
```

### Critical Setting Explanations:
1. `"latex-workshop.latex.outDir": "%DIR%"`:
   * Keeps the generated `main.pdf` right inside the document's directory (e.g., `synopsis/main.pdf`).
   * **Never** redirect `outDir` to a subfolder like `build/` unless explicitly requested, because external PDF viewers and VS Code tabs will lose synchronization and display stale versions.
2. `"files.exclude"`:
   * Hides all intermediate compiler files (`.aux`, `.toc`, `.log`, `.fls`, etc.) from the VS Code explorer sidebar so the user only sees clean source files.
3. `"latex-workshop.latex.rootFile.relativePath"`:
   * Tells LaTeX Workshop where the master document is located so compiling from subfiles does not trigger root-file errors.

---

## 3. Keyboard Shortcuts & Workflow

| Action | Shortcut (Windows) | Description |
| :--- | :--- | :--- |
| **Build Document** | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>B</kbd> | Runs the configured `pdflatex ➞ pdflatex` recipe. |
| **View PDF in Tab** | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>V</kbd> | Opens the PDF viewer in a side-by-side tab. |
| **SyncTeX: PDF ➔ Code** | <kbd>Ctrl</kbd> + Left Click in PDF | Jumps editor cursor directly to the corresponding `.tex` line. |
| **SyncTeX: Code ➔ PDF** | <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>J</kbd> | Jumps PDF viewer to the location of current code cursor. |
| **VS Code Build Task** | <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>B</kbd> | Runs the terminal-based build task defined in `tasks.json`. |

---

## 4. Subfile Root Magic Comments

Every subfile located in subfolders (`frontmatter/`, `chapters/`, `backmatter/`) **must** start with:

```latex
% !TEX root = ../main.tex
```

This ensures that whenever the user presses Build or saves while looking at any subfile, LaTeX Workshop builds `main.tex` and never treats the subfile as standalone.
