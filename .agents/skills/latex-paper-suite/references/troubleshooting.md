# LaTeX Troubleshooting & Error Catalog

This catalog documents the exact errors frequently encountered in this workspace, why they occur in LaTeX, and the verified remedies.

---

## 1. `! LaTeX Error: There's no line here to end.`

### Cause
The line-break command `\\` was invoked when LaTeX was **not** in a paragraph that has an active line to terminate.
Common triggers:
1. Placing `\\` immediately after a block environment like `\end{center}`, `\end{flushleft}`, `\end{table}`, or `\end{enumerate}`.
2. Placing `\\` at the very beginning of an environment or page.
3. Writing consecutive line breaks like `\\ \\` without any content in between.

### Solution
* **Never** use `\\` to add vertical spacing between blocks. Use standard LaTeX spacing commands:
  * `\vspace{5mm}` or `\vspace{\baselineskip}`
  * Or simply a blank line to start a new paragraph, combined with `\par\noindent`.
* Inside tables or headings, make sure each row contains text before ending with `\\`.

---

## 2. `Runaway argument? ... File ended while scanning use of \@xdblarg.`

### Cause
An opening brace `{` was never closed, or a command was improperly nested inside a sectioning macro.
Example from project:
```latex
% BROKEN:
\chapter{\uppercase\textbf{\section{Motivation}
```
Here, `\section{...}` was placed inside `\chapter{...}`, and curly braces remained unclosed before EOF.

### Solution
Sectioning commands (`\chapter`, `\section`, `\subsection`) must be separate declarations:
```latex
% FIXED:
\chapter{Motivation}
```

---

## 3. `! LaTeX Error: Command \bigsize already defined.`

### Cause
`\newcommand` was called for the same macro name in multiple included files (e.g., `cover.tex`, `VM.tex`, `POPSO.tex`, `CO-PO_PSO.tex`). `\newcommand` throws a fatal error if the macro already exists in LaTeX's symbol table.

### Solution
1. **Define globally once** in the preamble of `main.tex`:
   ```latex
   \providecommand{\normal}{\fontsize{12pt}{16pt}\selectfont}
   \providecommand{\size}{\fontsize{14pt}{18pt}\selectfont}
   \providecommand{\bigsize}{\fontsize{16pt}{20pt}\selectfont}
   \providecommand{\bigbigsize}{\fontsize{20pt}{24pt}\selectfont}
   ```
2. In subfiles, always use `\providecommand` instead of `\newcommand`. `\providecommand` safely no-ops if the macro has already been declared.

---

## 4. `! LaTeX Error: \begin{abstract} ended by \end{abstractpage}`

### Cause
In custom class files (e.g. `aktu.cls`), `\def\abstractpage` was defined to open an `abstract` environment:
```latex
\def\abstractpage{ ... \begin{abstract}}
```
However, the closing macro `\def\endabstractpage` was commented out or omitted. When `\end{abstractpage}` was reached, LaTeX found an open `abstract` environment without a corresponding `\end{abstract}`.

### Solution
Ensure matching environment terminators are defined in the class file:
```latex
\def\endabstractpage{\end{abstract}\newpage}
```

---

## 5. `! Undefined control sequence. <argument> \undefinedpagestyle`

### Cause
Calling `\thispagestyle{}` with an empty argument. LaTeX attempts to look up `\ps@{}` which evaluates to `\undefinedpagestyle`.

### Solution
Pass a valid page style name:
```latex
\thispagestyle{empty}  % or \thispagestyle{plain}
```

---

## 6. `! Undefined control sequence: \cftdot` or `\nomenclature`

### Cause
Commands from optional packages (`tocloft`, `nomencl`) were called in the document body, but the packages were not loaded in the preamble, or their auxiliary indexes (`makeindex`) were never run.

### Solution
For simple lists of symbols or abbreviations, avoid heavy dependencies and use standard LaTeX tables:
```latex
\begin{center}
\section*{\size\textbf{LIST OF SYMBOLS / ABBREVIATIONS}}
\begin{tabular}{p{3cm} p{10cm}}
\textbf{Symbol} & \textbf{Description} \\[1ex]
CNN & Convolutional Neural Network \\
CT  & Computed Tomography \\
GAN & Generative Adversarial Network \\
\end{tabular}
\end{center}
```

---

## 7. Edits to Subfiles Don't Appear in `main.pdf`

### Cause
1. **Compilation was not triggered:** Saving a subfile does not always trigger a build unless LaTeX Workshop detects the root file.
2. **Directory mismatch:** If `outDir` in `settings.json` was set to a subfolder like `build/`, `main.pdf` in the root folder remains an un-updated copy while `build/main.pdf` receives the updates.

### Solution
1. Verify that `settings.json` sets `"latex-workshop.latex.outDir": "%DIR%"`.
2. Ensure every subfile begins with `% !TEX root = ../main.tex`.
3. Press <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>B</kbd> to compile explicitly.
