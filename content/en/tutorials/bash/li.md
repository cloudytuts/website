---
title: ls
---

## NAME
ls -- list directory contents

## SYNOPSIS
```shell
ls [-ABCFGHLOPRSTUW@abcdefghiklmnopqrstuwx1%] [file ...]
```

## DESCRIPTION
For each operand that names a file of a type other than directory, ls displays its name as well as any requested,
associated information.  For each operand that names a file of type directory, ls displays the names of files con-
tained within that directory, as well as any requested, associated information.

If no operands are given, the contents of the current directory are displayed.  If more than one operand is given,
non-directory operands are displayed first; directory and non-directory operands are sorted separately and in lexico-
graphical order.

The following options are available: