# Bug 2

- Program: TeX
- Discovered: May 2026
- Status: to be reported

This is a bug in TeX at the intersection between hyphenation and lig/kern programming.
When the bug is hit, the output document contains duplicated characters.
In the minimal reproduction below, a paragraph `The journey.`
  should be typeset as `The journey,` based on the lig/kern program (the period is replaced by a comma).
However if TeX's hyphenation algorithm runs, the paragraph is incorrectly typeset as `The journey,,` (two commas).
Note that it is not necessary for TeX to actually hyphenate any words in the paragraph for the bug to appear.
If hyphenation does not occur, TeX's output is correct.

The bug is in sections 910 and 911 of the TeX source code (part 41: post hyphenation).

Understanding the bug requires significant background material, so I start with a reproduction first.
Then I describe the bug, and then I propose some fixes.

## Minimal reproduction

### Input files

#### dmr10.tfm: Computer Modern 10pt but with a custom lig/kern program

The bug requires a custom lig/kern program.
The example here uses a very simple lig/kern program with only one rule: the character pair `y.` is replaced by `y,`.
In property list format:

```
(LIGTABLE
   (LABEL C y)
   (/LIG> O 56 O 54)
   (STOP)
   )
```

Starting with the property list file for Computer Modern 10pt `cmr10.plst`,
  replace the lig/kern program to get `dmr10.plst`.
To generate `dmr10.tfm` run:

```
pltotf dmr10.plst
```

It is necessary to use a different filename (`dmr10.tfm` rather than `cmr10.tfm`)
because TeX doesn't load a `.tfm` file if it thinks it has already loaded it based on the file name.

#### input.tex

A straightforward TeX file.
The file contains two identical paragraphs `The journey.`.
The second paragraph is preceded by `\pretolerance=-1` which forces TeX's hyphenation
  algorithm to run as part of typesetting the paragraph.

Based on the lig/kern program, the expected output is two paragraphs `The journey,` (the period is replaced by a comma).
However the second paragraph is instead `The journey,,`: the comma is duplicated.

### Steps to reproduce:

Compile the TeX document:
```
tex document.tex
```

At this point the bug can be seen by inspecting the DVI file bytes.
Analyzing the output shows that the second paragraph is incorrectly `The journey,,`:

```
dvitype document.dvi
```

It's interesting to see the bug in a PDF too.
To do this, replace the `dmr10` font ID in the DVI file with `cmr10`.
This is necessary because there are no font glyphs for the made-up font `dmr10`.

```
perl -pe 's/dmr10/cmr10/g' document.dvi > documentv2.dvi
```

Then convert the file to PDF:

```
dvipdf documentv2.dvi
```

## Description of the bug

### Background

In order to describe the bug I first briefly recall how lig/kern programs and hyphenation work in TeX.

**Lig/kern programs** are a text preprocessing facility in TeX that transform the input sequence of characters.
A simple example is in Computer Modern, where the `ffi` sequence in the word `difficult` gets transformed into a single ligature `ﬃ`
  (character `0x0E` in Computer Modern, or Unicode `U+FB03`).
Lig/kern programs can add, remove, swap and otherwise change characters in the input.

**Hyphenation** is used in TeX's line breaking algorithm.
If TeX cannot find a "good" set of line breaks for a paragraph,
  it runs its hyphenation algorithm.
This algorithm runs _after_ the lig/kern program.
The algorithm inserts appropriate discretionary hyphens into some words in the paragraph.
For example, TeX inserts a discretionary hyphen [after the third character in word flying](https://hyphenate.dev/flying).
The line breaking algorithm can then choose to break lines at these additional points, as opposed to limiting itself to breaks between words.
If a line is broken at such a discretionary hyphen, a hyphen character is inserted into the output.

Now there is a tension between lig/kern programs and hyphenation, and the word `difficult` illustrates why.
Difficult has [a few possible hyphenation points](https://hyphenate.dev/difficult): `d-if-fi-cult`.
The second hyphenation point is in the middle of the `ffi` ligature.
Thus in order to hyphenate the word correctly, the ligature operation needs to be "undone" in some sense.
Because of this hyphenation is a three step process:

1. Pre-hyphenation (part 40 in TeX): the ligatures are broken up and the original sequence of characters is recovered.
2. Hyphenation (part 42 in TeX): hyphenation points are determined.
3. Post-hyphenation (part 41 in TeX): the ligatures are _reconstituted_ while discretionary hyphens are inserted. 

In step 3 the lig/kern program runs again.
In general this is a much more complex lig/kern invocation than the first time
  because the program has to factor in optional hyphen characters that may be inserted based on the breakpoints.
However for the purposes of this bug we can ignore this complication: entirely
  the bug relates to how post-hyphenation handles the original word only.

### Boundary character handling

We've seen how lig/kern programs and hyphenation are linked.
But there is a difference in which sequence of characters these operations run over.
A lig/kern program runs over each word in the paragraph _including punctuation_.
Thus, in the minimal example of this bug, the lig/kern program operates on `My` and `journey.`.

Hyphenation, on the other hand, only operates on alphabetic characters.
(Precisely: characters for which `\lccode` is non-zero.)
Thus hyphenation operates on `My` and `journey` and ignores the trailing `.`.
The lig/kern program that runs in post-hyphenation operates on the same input as hyphenation: it only operates on `My` and `journey`.

TeX is aware that there is a correctness issue here:
  when the lig/kern program runs in post-hyphenation it may need to factor in the character after the word and apply a lig/kern rule.
This is the regime the bug is in: our lig/kern program has a rule for `y.`.
To solve this TeX sets the right boundary character of the word to be the first character after the word, in this case `.`.



## Proposed solutions

## Additional information

### Discovery mechanism

As part of the [Texcraft project](https://github.com/jamespfennell/texcraft) I have been reimplementing the hyphenation code in TeX.
Based on my reading of the source code of the post-hyphenation algorithm (aka the reconstitute algorithm),
  I believe I have discovered a different bug in the algorithm that is triggered 
  only when the output contains a hyphenated word.
I found the present bug accidentally when trying to find a minimal example of this other bug.

The other bug will shortly be posted as bug #3 in this repo.
