# latex-moodle

A Claude Code plugin that converts LaTeX quiz files to [Moodle GIFT format](https://docs.moodle.org/en/GIFT_format).

## What it does

The `latex-to-gift` skill reads a `.tex` file containing multiple-choice quiz questions and writes a `.gift` file in the same directory, ready for import into Moodle.

Supported question types:

| Type | Detection |
|---|---|
| True/False | Answer choices are exactly "True" and "False" |
| Single correct | Exactly one `\textbf{(x)}` in the solution |
| Multiple correct | Two or more `\textbf{(x)}` in the solution — partial credit applied automatically |

## Expected LaTeX structure

```latex
\setcounter{number}{4}   % quiz number

\begin{enumerate}[label=\arabic*]
  \item Question text here.
  \begin{enumerate}[label=(\alph*)]
    \item Answer A
    \item Answer B
    \item Answer C
    \item Answer D
  \end{enumerate}
  \begin{solution}
    The correct answer is \textbf{(b)} because ...
  \end{solution}
\end{enumerate}
```

## Installation

### Via marketplace (recommended)

First add this repo as a marketplace, then install the plugin by name:

```bash
/plugin marketplace add Schneggera/latex-moodle-skill
/plugin install latex-moodle
```

### Locally

```bash
git clone https://github.com/Schneggera/latex-moodle-skill
claude plugin install ./latex-moodle-skill/plugins/latex-moodle
```

## Usage

```
/latex-to-gift path/to/quiz.tex
```

The output file is written to the same directory with a `.gift` extension (e.g. `quiz04.tex` → `quiz04.gift`) and starts with a `// Generated from <filename>` comment.

## GIFT output format

**Title line** for every question:

```
::Quiz N, Q<i>::
```

### True/False

```
::Quiz N, Q<i>:: <question text>
{TRUE}
```

`{FALSE}` is used when False is the correct answer.

### Single correct answer

The correct choice is listed first (prefixed `=`), wrong choices follow (prefixed `~`):

```
::Quiz N, Q<i>:: <question text>
{
=<correct answer>
~<wrong answer>
~<wrong answer>
}
```

### Multiple correct answers

Correct answers receive partial credit; wrong answers receive a `%-100%` penalty to discourage selecting all options:

| Correct answers | Credit per correct |
|---|---|
| 2 | `%50%` |
| 3 | `%33.33333%` |
| 4 | `%25%` |

```
::Quiz N, Q<i>:: <question text>
{
~%50%<correct answer 1>
~%50%<correct answer 2>
~%-100%<wrong answer 1>
~%-100%<wrong answer 2>
}
```

## Text transformations applied

- Inline math `$...$` → `\( ... \)`, display math `$$...$$` → `\[ ... \]`
- `\textbf{}`, `\emph{}`, `\text{}`, `\textrm{}` wrappers stripped (content kept)
- Common symbols: `\rightarrow` → `→`, `\leftarrow` → `←`, `\ldots` / `\dots` → `…`
- GIFT control characters (`~`, `=`, `#`, `{`, `}`, `:`) escaped in plain text regions
- Whitespace normalised (leading/trailing stripped, internal runs collapsed)

## License

MIT
