---
title: "Patch: LF endings for go.mod"
date: 2019-03-08T08:53:27-08:00
---

This patch makes the `go.mod` and `go.sum` files be checked out with LF
line endings on Windows.

See [hugo commit 4a6a6b78f2][].

[hugo commit 4a6a6b78f2]: https://github.com/neurocline/hugo/commit/4a6a6b78f2e9306127fd6685288cb6ad08777bcd

## Force LF line endings for go.mod/go.sum

Go tooling writes files with LF line endings. For peace of mind on Windows,
make sure that any Go files are checked out with LF line endings regardless
of the default settings for Git on that machine.

To that end, add `go.mod` and `go.sum` to the list of files that Hugo forces
LF line endings for. The updated `.gitattributes` now looks like this:

```
# Text files have auto line endings
* text=auto

# Go source files always have LF line endings
*.go text eol=lf
go.mod text eol=lf
go.sum text eol=lf

# SVG files should not be modified
*.svg -text
```
