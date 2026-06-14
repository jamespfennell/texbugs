# Bug 2

## Input files

### dmr10.tfm: Computer Modern 10pt but with a custom lig/kern program

The bug requires a custom lig/kern program.
The example here has a very simple lig/kern program with only one rule: the character pair `y.` is replaced by `y,`.
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
because TeX don't load a `.tfm` file if it thinks it has already loaded it based on the file name.

### input.tex

A straightforward TeX file, nothing complicated here.

## Reproduction

Compile the TeX document:
```
tex document.tex
```

At this point the bug can be seen by inspecting the DVI file bytes.

However it's easier to see it in a PDF.
To do this, replace the `dmr10` font ID in the DVI file with `cmr10`.
This is necessary because there are no font glyphs for the made-up font `dmr10`.

```
perl -pe 's/dmr10/cmr10/g' document.dvi > documentv2.dvi
```

Then convert the file to PDF:

```
dvipdf documentv2.dvi
```
