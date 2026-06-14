# Bug 2

## Input files

```
(LIGTABLE
   (LABEL O 54)
   (/LIG/> O 174 C C)
   (STOP)
   (LABEL C y)
   (/LIG> O 56 O 54)
   (STOP)
   )
```

## Reproduce the bug


```
tex document.tex
perl -pe 's/dmr10/cmr10/g' document.dvi > documentv2.dvi
dvipdf documentv2.dvi
```
