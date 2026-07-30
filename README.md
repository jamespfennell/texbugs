# TeX bugs

This is a repository for me to record bugs I find in TeX and related software.
Contemporary bugs in TeX tend to be very subtle and technical because the software is old and because it is feature frozen.
All of the "simple" bugs have already been found.
I use this repository to build minimal reproductions and polish the bug reports.

I find bugs in TeX as a side effect of my work on [Texcraft](https://github.com/jamespfennell/texcraft).

| Bug # | Discoved   | Software | Synopsis                                              | Status
|-------|------------|----------|-------------------------------------------------------|-
| 1     | March 2024 | tftopl   | Some valid `.tfm` files are rejected.                 | Confirmed
| 2     | May 2026   | TeX      | After hyphenation some ligature nodes are duplicated. | To be reported
