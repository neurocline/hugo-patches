---
title: "Patch: Static Files Perf"
date: 2019-03-07T16:52:42-08:00
---

This patch measures and outputs the time copying the `static` directory takes.

See [hugo commit d2be6f785f][] for details.

[hugo commit d2be6f785f]: https://github.com/neurocline/hugo/commit/d2be6f785f2531b49ee6a1474e2ad9d7489e4514

## Extend --stepAnalysis to show static file copy time

The `--stepAnalysis` feature is useful, but it doesn't time the copying of
static files, and this can be quite substantial in large sites.

The reason it wasn't being timed is that, normally, Hugo runs two goroutines,
one to generate the site, and the other to copy static files. The `nitro`
package is a very simple global timer, so we can't stamp events from multiple
goroutines. To get around this issue, the patch add a manual time delta
measurement for the `copyStaticFunc` function, writes that value to a global,
and then outputs it at the end.

The result looks like this; it simply shows the elapsed time for static
file copy at the end of `--stepAnalysis` output.

```
C:\projects\github\neurocline\hugo-notebook>hugo --stepAnalysis
Building sites … initialize:
        31.9143ms (32.9114ms)       6.33 MB     106104 Allocs
load data:
        0s (33.9092ms)      0.02 MB     181 Allocs
load i18n:
        0s (33.9092ms)      0.00 MB     0 Allocs
read and convert pages from source:
        42.8857ms (77.7926ms)      19.04 MB     151770 Allocs
build Site meta:
        0s (78.7891ms)      0.01 MB     138 Allocs
prepare pages:
        0s (79.7884ms)      0.23 MB     2132 Allocs
render and write aliases:
        0s (80.7836ms)      0.00 MB     12 Allocs
render and write pages:
        20.9444ms (101.728ms)       8.91 MB     59931 Allocs
render and write Sitemap:
        2.9885ms (105.7171ms)       0.25 MB     4458 Allocs
render and write robots.txt:
        0s (106.7154ms)     0.00 MB     17 Allocs
render and write 404:
        0s (106.7154ms)     0.00 MB     23 Allocs
render and write pages:
        5.9845ms (113.6965ms)       1.36 MB     18414 Allocs
Static files in 663 ms
```

Note here that site generation was long since done by the time static file
copying finished. This is useful information for people trying to optimize the
build time of large sites.

And, static file copy needs some optimization, it's slower than it could
be (this was about 80MB of files located on a fast SSD).

## tests

All tests were run with `go test ./...` from the root of the Hugo directory.

The `gofmt` tool was used on all modified files.
