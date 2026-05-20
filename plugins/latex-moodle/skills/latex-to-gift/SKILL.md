---
name: latex-to-gift
description: Convert Latex file to Gift format
---
# Latex to gift skill

Convert the LaTeX quiz file at `$ARGUMENTS` to Moodle GIFT format and write the result to the same directory with a `.gift` extension.

## Step 1 — Read the file

Read the file at `$ARGUMENTS`. If the path is missing or the file does not exist, tell the user and stop.

## Step 2 — Extract structure

From the file extract:

1. **Quiz number** `N`: from `\setcounter{number}{N}`.
2. **Questions**: each top-level `\item` inside the outer `\begin{enumerate}` (the one with `label=\arabic*`). Number them 1, 2, 3, … in order.
3. **Answer choices**: the nested `\item` entries inside each inner `\begin{enumerate}` (the one with `label=(\alph*)`). Map them in order to letters: first item = (a), second = (b), etc.
4. **Correct answers**: from `\begin{solution}...\end{solution}`. The correct letters are those wrapped in `\textbf{(x)}` inside the solution text.

## Step 3 — Detect question type per question

- **True/False**: the answer choices are exactly two items whose text is "True" and "False" (case-insensitive, in any order).
- **Single correct**: the solution contains exactly one `\textbf{(x)}` letter.
- **Multiple correct**: the solution contains two or more `\textbf{(x)}` letters.

## Step 4 — Apply text transformations

Apply these transformations to **all question text and answer text** before writing GIFT output:

1. **Math delimiters**: Replace inline math `$...$` with `\( ... \)` and display math `$$...$$` with `\[ ... \]`. Keep the LaTeX commands inside unchanged.
2. **LaTeX formatting wrappers**: Strip the wrapper but keep the content — `\textbf{x}` → `x`, `\emph{x}` → `x`, `\text{x}` → `x`, `\textrm{x}` → `x`.
3. **Common LaTeX symbols**: Replace with Unicode — `\rightarrow` → `→`, `\leftarrow` → `←`, `\ldots` → `…`, `\dots` → `…`.
4. **Escape GIFT control characters**: In plain text regions (outside `\(...\)` and `\[...\]`), prefix each occurrence of `~`, `=`, `#`, `{`, `}`, `:` with a backslash so they are not misread as GIFT syntax.
5. **Trim whitespace**: Remove leading/trailing whitespace and collapse internal runs of whitespace (including newlines from multi-line LaTeX) into a single space.

## Step 5 — Build GIFT syntax per question

Use the question number `i` (1-based) and quiz number `N` from Step 2.

**Title line** (same for all types):
```
::Quiz N, Q<i>::
```

### True/False

```
::Quiz N, Q<i>:: <question text>
{TRUE}
```

Use `{FALSE}` if False is the correct answer.

### Single correct answer

```
::Quiz N, Q<i>:: <question text>
{
=<correct answer text>
~<wrong answer text>
~<wrong answer text>
~<wrong answer text>
}
```

List the correct answer first (prefixed `=`), then all wrong answers (prefixed `~`), preserving their original order among the wrong ones.

### Multiple correct answers

Let K = number of correct answers. Each correct answer gets `%P%` partial credit where:
- K = 2 → P = 50
- K = 3 → P = 33.33333
- K = 4 → P = 25

Wrong answers get `%-100%` to prevent guessing by selecting all.

```
::Quiz N, Q<i>:: <question text>
{
~%50%<correct answer 1>
~%50%<correct answer 2>
~%-100%<wrong answer 1>
~%-100%<wrong answer 2>
}
```

List correct answers first (in their original order), then wrong answers (in their original order).

## Step 6 — Write the output file

- Output path: same directory as the input file, same basename, `.gift` extension (e.g. `iase26_quiz04.tex` → `iase26_quiz04.gift`).
- Start the file with: `// Generated from <input filename>`
- Separate each question block with exactly one blank line.
- Write the file using the Write tool.

After writing, confirm to the user: the output path and how many questions were converted, noting the type of each (single, multi, true/false).
