# LaTeX Beginner Cheat Sheet & Style Guide

A practical guide for authors and beginners writing academic papers in LaTeX.

---

## 1. Document Anatomy

```latex
% !TEX root = main.tex
\documentclass[12pt,a4paper]{aktu}   % 1. Document class defines layout

% --- PREAMBLE (Packages & Definitions) ---
\usepackage{graphicx}                % For images
\usepackage{amsmath,amssymb}         % For math formulas
\graphicspath{{assets/}}             % Default image directory

\begin{document}                     % 2. Start of document

% --- FRONT MATTER ---
\pagenumbering{roman}
\input{frontmatter/cover}
\input{frontmatter/contents}

% --- MAIN BODY ---
\pagenumbering{arabic}
\clearpage
\input{chapters/01_introduction}

% --- BACK MATTER ---
\clearpage
\input{backmatter/references}

\end{document}                       % 3. End of document
```

---

## 2. How to Add a New Chapter

1. Create a new `.tex` file in `chapters/`, e.g. `chapters/09_evaluation.tex`.
2. Add the root magic comment at the top:
   ```latex
   % !TEX root = ../main.tex
   \chapter{Experimental Evaluation}
   \label{chap:evaluation}

   This chapter discusses our experimental findings...
   ```
3. Open `synopsis/main.tex` and insert the input line:
   ```latex
   \clearpage
   \input{chapters/09_evaluation}
   ```
4. Save and compile with <kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>B</kbd>.

---

## 3. Inserting Figures

Always use the standard floating `figure` environment with `\centering`, `\caption`, and `\label`:

```latex
\begin{figure}[htbp]
  \centering
  \includegraphics[width=0.85\linewidth]{architecture_diagram.png}
  \caption{Overview of the MEV-aware liquidation pipeline.}
  \label{fig:mev_pipeline}
\end{figure}
```

> [!TIP]
> Place `\label` **after** `\caption`. If `\label` is placed before `\caption`, `\ref{fig:...}` will refer to the section number instead of the figure number.

---

## 4. Creating Tables

```latex
\begin{table}[htbp]
  \centering
  \caption{Performance comparison of liquidation strategies.}
  \label{tab:performance_comparison}
  \begin{tabular}{l|c|c|r}
    \hline
    \textbf{Strategy} & \textbf{Latency (ms)} & \textbf{Gas Used} & \textbf{Success Rate} \\
    \hline
    Standard Arbitrage & 142 & 210,000 & 82.4\% \\
    MEV-Protected Flash & 38  & 185,000 & 96.1\% \\
    \hline
  \end{tabular}
\end{table}
```

---

## 5. Mathematical Equations

* **Inline math:** Use `$E = mc^2$` or `\( E = mc^2 \)`.
* **Numbered display equation:**
  ```latex
  \begin{equation}
    \mathcal{L}_{\text{total}} = \alpha \mathcal{L}_{\text{MEV}} + (1 - \alpha) \mathcal{L}_{\text{slippage}}
    \label{eq:loss_function}
  \end{equation}
  ```
* Reference the equation in prose using:
  ```latex
  As defined in Equation~\eqref{eq:loss_function}, the loss function combines...
  ```

---

## 6. Citations & Cross-Referencing

* **Citing a paper:**
  ```latex
  Recent advances in DeFi liquidations~\cite{daian2020flashboys} have demonstrated...
  ```
* **Cross-referencing sections, figures, or chapters:**
  ```latex
  See Section~\ref{sec:motivation} for context, and Figure~\ref{fig:mev_pipeline} for the architecture.
  ```
  *(The `~` creates a non-breaking space so numbers are never wrapped to the next line alone).*

---

## 7. Crucial Typography & Escaping Rules

In LaTeX, these characters have special syntactic meaning and **must** be escaped with a backslash if you want them to print literally:

| Character | Meaning in LaTeX | How to write literal text |
| :---: | :--- | :--- |
| `%` | Comment | `\%` |
| `_` | Subscript | `\_` |
| `&` | Table column separator | `\&` |
| `#` | Macro parameter | `\#` |
| `$` | Math mode toggle | `\$` |
| `{` `}` | Argument grouping | `\{` `\}` |
| `\` | Command prefix | `\textbackslash{}` |
| `~` | Non-breaking space | `\textasciitilde{}` |

* **Quotation marks:** Never use straight double quotes `""`. Use backticks for opening and single quotes for closing:
  * `` ``quoted text'' `` $\rightarrow$ “quoted text”
