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
<!--Documentation https://docs.moodle.org/502/en/GIFT_format-->

Use the question number `i` (1-based) and quiz number `N` from Step 2.

**Title line** (same for all types):
```
::Quiz N, Q<i>::
```

**GIFT format** allows someone to use a text editor to write multiple-choice, true-false, short answer, matching missing word and numerical questions in a simple format that can be imported. The GIFT format is also an export file format available in Question bank. The format has been developed within the Moodle Community but other software may support it to a greater or lesser degree.

- When creating a large numbers of questions, GIFT can provide a quick way of bulk loading questions either into a question category, or into a Lesson.
- Sometimes it is easier proofing questions in a question category by viewing them in a GIFT file.

### General instructions

At least one blank line must be left between each question.

In the simple form, the question comes first, then the answers are set in between brackets, with an equal sign (=) indicating the correct answer(s) and tilde (~) the wrong answers. A hash (#) will insert a response. Questions can be weighted by placing percentage signs (%..%) around the weight. Comments are preceded by double slashes (//) and are not imported.

Here are some useful [GIFT examples](http://moodle.org/pluginfile.php/134/mod_forum/attachment/236161/GIFT-examples.zip) than can be imported or used as rough template. Many of the examples below used the questions in the file as a starting point.

### UTF-8 encoding

Any GIFT file **must** be correctly encoded in UTF-8. You can use Microsoft's text editor Notepad which comes with Windows to save your file in UTF-8.

**Note**: ANSI format will (only) work for languages without any special characters (like ä, ö, ü, æ, å, ø, œ or ß). And don't use "Unicode" as format as this is actually UTF-16 and won't work. See "Converting files to UTF-8" for further information.

### Format symbols

Here are some common GIFT symbols and their use.

| Symbols | Use |
|---|---|
| `// text` | Comment until end of line (optional) |
| `::title::` | Question title (optional) |
| `text` | Question text (becomes title if no title specified) |
| `[...format...]` | The format of the following bit of text. Options are `[html]`, `[moodle]`, `[plain]` and `[markdown]`. The default is `[moodle]` for the question text, other parts of the question default to the format used for the question text. |
| `{` | Start answer(s) -- without any answers, text is a description of following questions |
| `{T}` or `{F}` | True or False answer; also `{TRUE}` and `{FALSE}` |
| `{ ... =right ... }` | Correct answer for multiple choice, (multiple answer? -- see page comments) or fill-in-the-blank |
| `{ ... ~wrong ... }` | Incorrect answer for multiple choice or multiple answer |
| `{ ... =item -> match ... }` | Answer for matching questions |
| `#feedback text` | Answer feedback for preceding multiple, fill-in-the-blank, or numeric answers |
| `####general feedback` | General feedback |
| `{#` | Start numeric answer(s) |
| `answer:tolerance` | Numeric answer accepted within +/- tolerance range |
| `low..high` | Lower and upper range values of accepted numeric answer |
| `=%n%answer:tolerance` | n percent credit for one of multiple numeric ranges within tolerance from answer |
| `}` | End answer(s) |
| `\character` | Backslash escapes the special meaning of ~, =, #, {, }, and : |
| `\n` | Places a newline in question text -- blank lines delimit questions |

Here are some quick examples:

```gift
// true/false
::Q1:: 1+1=2 {T}

// multiple choice with specified feedback for right and wrong answers
::Q2:: What's between orange and green in the spectrum?
{ =yellow # right; good! ~red # wrong, it's yellow ~blue # wrong, it's yellow }

// fill-in-the-blank
::Q3:: Two plus {=two =2} equals four.

// matching
::Q4:: Which animal eats which food? { =cat -> cat food =dog -> dog food }

// math range question
::Q5:: What is a number from 1 to 5? {#3:2}

// math range specified with interval end points
::Q6:: What is a number from 1 to 5? {#1..5}
// translated on import to the same as Q5, but unavailable from Moodle question interface

// multiple numeric answers with partial credit and feedback
::Q7:: When was Ulysses S. Grant born? {
         =1822:0      # Correct! Full credit.
         =%50%1822:2  # He was born in 1822. Half credit for being close.
}

// essay
::Q8:: How are you? {}
```

### Format symbols explained

The multiple choice format below has a comment line `//` for the question, when Moodle exports it the question unique id number will appear here.

The first set of `::` precedes the question title.

The second `::` precedes the actual question. The first `{` indicates the start of the answers. The correct answer is preceded by an `=` sign and wrong answers by a `~`. Teacher responses have a `#` in front of them. The question ends with a `}` and then a blank line. NOTE it is `{ }` not `( )` parenthesis! Usually these are obtained with help of the [AltGr] key.

```gift
//Comment line
::Question title
:: Question {
=A correct answer
~Wrong answer1
#A response to wrong answer1
~Wrong answer2
#A response to wrong answer2
~Wrong answer3
#A response to wrong answer3
~Wrong answer4
#A response to wrong answer4
}
```

The shortest format for a multiple choice question is:

```gift
Question{= A Correct Answer ~Wrong answer1 ~Wrong answer2 ~Wrong answer3 ~Wrong answer4 }
```

*Tip:* If you don't specify a question title the WHOLE question will be used as the title at the time of import into Moodle. There are pros and cons to allowing this to happen.

- Cons: This can add a lot of unnecessary words. This can include characters which might confuse the export GIFT process.
- Pros: On the other hand, if the start of each question is different, it can make finding a single question easier in a category list of questions. It will save you typing. Having the same title for every question is a very bad idea.

### Question format examples

There are several ways to use a text editor to write a GIFT format. We will try to show the simple version for example and in some formats we will introduce some more complex features that can be imported into many Moodle Question formats.

#### Multiple choice

For multiple choice questions, wrong answers are prefixed with a tilde (~) and the correct answer is prefixed with an equal sign (=).

Here is a simple acceptable GIFT multiple choice format:

```gift
Who's buried in Grant's tomb?{=Grant ~no one ~Napoleon ~Churchill ~Mother Teresa }
```

Here is a longer format that uses most of the GIFT elements:

```gift
// question: 1 name: Grants tomb
::Grants tomb::Who is buried in Grant's tomb in New York City? {
=Grant
~No one
#Was true for 12 years, but Grant's remains were buried in the tomb in 1897
~Napoleon
#He was buried in France
~Churchill
#He was buried in England
~Mother Teresa
#She was buried in India
}
```

#### Multiple choice with multiple right answers

That is, using checkboxes, not radio buttons:

```gift
What two people are entombed in Grant's tomb? {
   ~%-100%No one
   ~%50%Grant
   ~%50%Grant's wife
   ~%-100%Grant's father
}
```

#### True-false

In this question-type the answer indicates whether the statement is true or false. The answer should be written as `{TRUE}` or `{FALSE}`, or abbreviated to `{T}` or `{F}`.

```gift
// question: 0 name: TrueStatement using {T} style
::TrueStatement about Grant::Grant was buried in a tomb in New York City.{T}

// question: 0 name: FalseStatement using {FALSE} style
::FalseStatement about sun::The sun rises in the West.{FALSE}
```

#### Short answer

Answers in Short Answer question-type are all prefixed by an equal sign (=), indicating that they are all correct answers. The answers must not contain a tilde.

Here are two examples using the simple method showing possible right answers for credit.

```gift
Who's buried in Grant's tomb?{=Grant =Ulysses S. Grant =Ulysses Grant}

Two plus two equals {=four =4}
```

If there is only one correct Short Answer, it may be written without the equal sign prefix, as long as it cannot be confused as True-False.

#### Matching

Matching pairs begin with an equal sign (=) and are separated by this symbol "->". There must be at least three matching pairs.

```gift
Match the following countries with their corresponding capitals. {
   =Canada -> Ottawa
   =Italy  -> Rome
   =Japan  -> Tokyo
   =India  -> New Delhi
   }
```

Matching questions do not support feedback or percentage answer weights.

#### Missing word

The Missing Word format automatically inserts a fill-in-the-blank line (like this _____) in the middle of the sentence. To use the Missing Word format, place the answers where you want the line to appear in the sentence.

```gift
Moodle costs {~lots of money =nothing ~a small amount} to download from moodle.org.
```

If the answers come before the closing punctuation mark, a fill-in-the-blank line will be inserted for the "missing word" format. All question types can be written in the Missing Word format.

There must be a blank line (double carriage return) separating questions. For clarity, the answers can be written on separate lines and even indented. Some examples:

```gift
Mahatma Gandhi's birthday is an Indian holiday on  {
~15th
~3rd
=2nd
} of October.

Since {
  ~495 AD
  =1066 AD
  ~1215 AD
  ~ 43 AD
}
the town of Hastings England has been "famous with visitors".
```

#### Numerical questions

The answer section for Numerical questions must start with a number sign (#). Numerical answers can include an error margin, which is written following the correct answer, separated by a colon. So for example, if the correct answer is anything between 1.5 and 2.5, then it would be written as follows `{#2:0.5}`. This indicates that 2 with an error margin of 0.5 is correct (i.e., the span from 1.5 to 2.5). If no error margin is specified, it will be assumed to be zero.

Here is a simple numerical format question. It will accept a range of 5 years.

```gift
When was Ulysses S. Grant born?{#1822:5}
```

It is a good idea to check the margins of the range, 3.141 is not counted as correct and 3.142 is considered in the range.

```gift
What is the value of pi (to 3 decimal places)? {#3.14159:0.0005}.
```

Optionally, numerical answers can be written as a span in the following format `{#MinimumValue..MaximumValue}`.

```gift
What is the value of pi (to 3 decimal places)? {#3.141..3.142}.
```

Moodle's browser interface does not support multiple numerical answers, but Moodle's code can and so does GIFT. This can be used to specify numerical multiple spans, and can be particularly useful when combined with percentage weight grades. If multiple answers are used, they must be separated by an equal sign, like short answer questions.

```gift
When was Ulysses S. Grant born? {#
   =1822:0
   =%50%1822:2
}
```

Note that since Moodle's browser GUI didn't support multiple answers for Numerical questions, there's no way in older Moodle versions to see them or edit them through Moodle. The only way to change a numerical answer beyond the first, is to delete the question and re-import it (or use something like phpMyAdmin). But better would be to upgrade your Moodle to at least 1.9.

#### Essay

An essay question is simply a question with an empty answer field. Nothing is permitted between the curly braces at all.

```gift
Write a short biography of Dag Hammarskjold. {}
```

#### Description

A description "question" has no answer part at all

```gift
You can use your pencil and paper for these next math questions.
```

### Options

In addition to these basic question types, this filter offers the following options: line comments, question name, feedback and percentage answer weight.

#### Line comments

Comments that will not be imported into Moodle can be included in the text file. This can be used to provide headers or more information about questions. All lines that start with a double backslash (not counting tabs or spaces) will be ignored by the filter.

```gift
// Subheading: Numerical questions below
What's 2 plus 2? {#4}
```

Comments will be exported from Moodle and will include the unique question id. The above question after it was imported and then exported from Moodle:

```gift
// question: 914  name: What's 2 plus 2?
::What's 2 plus 2?::What's 2 plus 2?{#
    =4:0#
}
```

The question ID number and any question tags are exported and can be imported using the format:

```gift
//  [id:123] [tag:basic] [tag:set 1]
```

#### Question name

A question name can be specified by placing it first and enclosing it within double colons (::...::).

```gift
::Kanji Origins::Japanese characters originally
came from what country? {=China}

::Thanksgiving Date::The American holiday of Thanksgiving is
celebrated on the {~second ~third =fourth} Thursday of November.
```

If no question name is specified, the entire question will be used as the name by default.

#### Feedback

Feedback can be included for each answer by following the answer with a number sign (# also known as a hash mark) and the feedback.

```gift
What's the answer to this multiple-choice question? {
  ~wrong answer#feedback comment on the wrong answer
  ~another wrong answer#feedback comment on this wrong answer
  =right answer#Very good!
}

//From The Hitchhiker's Guide to the Galaxy
Deep Thought said " {
  =forty two#Correct according to The Hitchhiker's Guide to the Galaxy!
  =42#Correct, as told to Loonquawl and Phouchg
  =forty-two#Correct!
}  is the Ultimate Answer to the Ultimate Question of Life, The Universe, and Everything."

   42 is the Absolute Answer to everything.{
FALSE#42is the Ultimate Answer.#You gave the right answer.}
```

For Multiple Choice questions, feedback is displayed only for the answer the student selected. For short answer, feedback is shown only when students input the corresponding correct answer. For true-false questions, there can be one or two feedback strings. The first is shown if the student gives the wrong answer. The second if the student gives the right answer.

#### Percentage answer weights

Percentage answer weights are available for both Multiple Choice and Short Answer questions. Percentage answer weights can be included by following the tilde (for Multiple Choice) or equal sign (for Short Answer) with the desired percent enclosed within percent signs (e.g., %50%). This option can be combined with feedback comments.

```gift
Difficult question.{~wrong answer ~%50%half credit answer =full credit answer}

::Jesus' hometown::Jesus Christ was from {
   ~Jerusalem#This was an important city, but the wrong answer.
   ~%25%Bethlehem#He was born here, but not raised here.
   ~%50%Galilee#You need to be more specific.
   =Nazareth#Yes! That's right!
}.

::Jesus' hometown:: Jesus Christ was from {
   =Nazareth#Yes! That's right!
   =%75%Nazereth#Right, but misspelled.
   =%25%Bethlehem#He was born here, but not raised here.
}
```

Note that the last two examples are essentially the same question, first as multiple choice and then as short answer.

Note that it is possible to specify percentage answer weights that are NOT available through the browser interface. The Match Grades drop-down on the import page determines how these are handled. You can either request that an error be reported or that the answer weight be adjusted to the nearest valid answer weight.

Note specifically, that Moodle uses 5 decimal places to do its math, so if you wish to divide percentages in thirds, use %33.33333 instead of %33 or %33.33.

Specifying text formatting for the question
The question text (only) may have an optional text format specified. Currently the available formats are `moodle` (Moodle Auto-Format), `html` (HTML format), `plain` (Plain text format) and `markdown` (Markdown format). The format is specified in square brackets immediately before the question text. See "Formatting text" for further information.

```gift
[markdown]The *American holiday of Thanksgiving* is celebrated on the {
   ~second
   ~third
   =fourth
} Thursday of November.
```

#### Multiple answers

The Multiple Answers option is used for multiple choice questions when two or more answers must be selected in order to obtain full credit. The multiple answers option is enabled by assigning partial answer weight to multiple answers, while allowing no single answer to receive full credit.

```gift
What two people are entombed in Grant's tomb? {
   ~No one
   ~%50%Grant
   ~%50%Grant's wife
   ~Grant's father
}
```

Note that there is no equal sign (=) in any answer and the answers should total no more than 100%, otherwise Moodle will return an error. To avoid the problem of students automatically getting 100% by simply checking all of the answers, it is best to include negative answer weights for wrong answers.

```gift
What two people are entombed in Grant's tomb? {
   ~%-50%No one
   ~%50%Grant
   ~%50%Grant's wife
   ~%-50%Grant's father
}
```

#### Special characters ~ = # { }

These symbols `~ = # { } :` control the operation of this filter and cannot be used as normal text within questions. Since these symbols have a special role in determining the operation of this filter, they are called "control characters." But sometimes you may want to use one of these characters, for example to show a mathematical formula in a question. The way to get around this problem is "escaping" the control characters. This means simply putting a backslash (\) before a control character so the filter will know that you want to use it as a literal character instead of as a control character. For example:

```gift
Which answer equals 5? {
   ~ \= 2 + 2
   = \= 2 + 3
   ~ \= 2 + 4
}

::GIFT Control Characters::
Which of the following is NOT a control character for the GIFT import format? {
  ~ \~     # \~ is a control character.
  ~ \=     # \= is a control character.
  ~ \#     # \# is a control character.
  ~ \{     # \{ is a control character.
  ~ \}     # \} is a control character.
  = \      # Correct! \ (backslash) is not a control character. BUT,
             it is used to escape the control characters.
}
```

When the question is processed, the backslash is removed and is not saved in Moodle.

#### HTML in answers

The GIFT format will interpret HTML correctly if you add `[html]` in front of the question. See this forum thread for further discussion.

#### Specifying categories

It is possible to change the category into which the questions are added within the GIFT file. You can change the category as many times as you wish within the file. All questions after the modifier up to the next modifier or the end of the file will be added to the specified category. Up to the first category modifier the category specified on the import screen will be used. Note that for this to work the "from file:" box must be ticked on the import screen.

To include a category modifier include a line like this (with a blank line before and after):

```gift
$CATEGORY: tom/dick/harry
```

or simply

```gift
$CATEGORY: mycategory
```

The first example specifies a path of nested categories. In this case the questions will go into harry. The categories are created if they do not exist.

To find out how your categories are organized, you might try exporting some questions including category data first and check the exported GIFT formatted file. The lowest level of system context might give you something like `$CATEGORY: $system$/...`.

If you need to escape the / character, you need to put two slashes instead of one.

#### Making questions case sensitive

Short Answer questions can be made case sensitive by changing "0" to "1" in the following line **of your moodle/question/format/gift/format.php file**:

```php
$question->usecase = 0; // Ignore case
```

### Hints and tips

- Use the `::title::` at the beginning of every question to organize your questions when Moodle presents a list or exports them as another GIFT file. When the title is left blank, Moodle will put the beginning of the question as the title. Some teachers want to see something like "001 LIT101 Poe ref Purloin Letter" or "The Purloin Letter was written by (AmLit pg 254)" in the title.
- You can specify markup if you need to format the question by setting `[html]`, `[moodle]`, `[plain]` or `[markdown]` just before the question text. See more about this in the reference pdf below.
- In the Lesson module, in a question page, correct answers jump by default to Next page and incorrect answers jump to This page (i.e. student has to "try again"). When importing from a GIFT format file, this is exactly the mechanism which is used.
- If you want a student to be taken directly from one question to the next irrespective of their answer being correct or incorrect: in the Lesson Settings, set Maximum number of attempts to 1.
  - Please note, however, that a message "correct / incorrect" will still be displayed to the student upon answering each question. If you do not want this (default) feedback message to be displayed then enter your own feedback message (i.e. "continue", "---", etc.)
  - In case you want no visible message displayed then enter a non-breaking space as feedback. Moodle will not put its automatic response because it sees the blank space. To do this, put a `#` after the answer and write `[[Image:Nbsp.png]]` (without spaces between these characters).
- Need to use a special GIFT character in your question or answer? Put a `\` in front of the GIFT character.
  - For example, if you want to use curly braces, `{` or `}`, or equal sign, `=`, or `#` or `~` in a GIFT file (in a math question including TeX expressions) you must "escape" them by preceding them with a `\` directly in front of each `{` or `}` or `=`. It is possible to use a replace program/macro/editor filter to do this conversion before importing to Moodle.
- Want to change T/F type questions to multiple choice? Consider exporting the T/F questions as a GIFT file, then using a text editor to replace the `(T)` with `(=True ~False)`. Perhaps change the title slightly so you will recognize the new questions.
- Alphabetic case-sensitive comparison is disabled by default. If you need case-sensitive comparison for short answer questions (an unusual need), precede them with:

```php
$question->usecase = 1;
```

surrounded by blank lines.


## Step 6 — Write the output file

- Output path: same directory as the input file, same basename, `.gift` extension (e.g. `iase26_quiz04.tex` → `iase26_quiz04.gift`).
- Start the file with: `// Generated from <input filename>`
- Separate each question block with exactly one blank line.
- Write the file using the Write tool.

After writing, confirm to the user: the output path and how many questions were converted, noting the type of each (single, multi, true/false).
