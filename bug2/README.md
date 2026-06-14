# Bug 2

- Program: TeX
- Discovered: May 2026

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

## Additional information

### Discovery mechanism

As part of the [Texcraft project](https://github.com/jamespfennell/texcraft) I have been reimplementing the hyphenation code in TeX.
Based on my reading of the source code of the post-hyphenation algorithm (aka the reconstitute algorithm),
  I believe I have discovered a different bug in the algorithm that is triggered 
  only when the output contains a hyphenated word.
I found the present bug accidentally when trying to find a minimal example of this other bug.

The other bug will shortly be posted as bug #3 in this repo.
