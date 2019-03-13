---
title: "Patch: Num Workers"
date: 2019-03-07T17:37:15-08:00
---

This patchs adds a command-line option `--workers=NUM` to the
`hugo` and `hugo server` commands to let the user override defaults
for the number of goroutine workers spawned.

See [hugo commit dedeab73f8][] for details.

[hugo commit dedeab73f8]: https://github.com/neurocline/hugo/commit/dedeab73f8d52425702260eba6e83fd303991d36

## add --num-workers, update defaults

Hugo spawns multiple goroutines in specific parts of the program. This
is a good thing, but the parallelism also can cause problems for some
use cases. When debugging, parallel goroutines emit log messages in
an interleaved fashion, making it harder to make sense of operations.
And on some systems, the defaults may lead to reduced performance.

Currently, three places in Hugo spawn multiple goroutines:

- `renderPages` to render markup files to HTML
- `newSiteContentProcessor` to process layouts
- `newCapturer` to iterate directories

This patch does three things.

First, it doesn't use the `GOMAXPROCS` environment variable. This was
almost never set to an actual value, meaning that the parts of Hugo
that used it always did the same thing on all machines. This code
has been removed.

Second, it adds a `--workers=NUM` command-line option. If used, the
specified values is used when creating workers in the three sections
mentioned above. This can be useful for debugging, to get serialized
messages. It can also be useful on high-CPU machines, where creating
many dozens of goroutines is probably counterproductive.

Third, the default values for number of workers is tweaked. The
`renderPages` code now spawns `2 * runtime.NumCPU` workers, with a
minimum of 4. This will actually spawn more workers on higher-CPU
count machines, since due to the use of `GOMAXPROCS` previously, most
Hugo runs used 4 workers here. The `newSiteContentProcessor` code
still spawns `3 * runtime.NumCPU* workers`, but the minimum has been
dropped to 6, which is half of the previous code's default; a minimum
of 12 is far too high for low-core-count systems.

## Open questions

The `newCapturer` function was modifed to add `numWorkers` as a parameter.
This affects the tests that also call `newCapturer`, which now pass 0
to get the default behavior. That's a bit annoying, but I didn't want to have
the `newCapturer` function on the hook to look at config.

Maybe `--workers` should be a multiplier of some sort and not a literal number.
The sites that spawn workers each have different reasons for using parallel
workers. At the very least, having many workers for `newCapturer` is likely
pointless and potentially harmful.

The `newCapturer` code in particular probably needs rework. It spawns equal
numbers of workers for its three tasks, but most of the work only happens
on one of the branches. It should probably turn into a unified
scheme where there's a fixed number of workers and a free worker does whatever
task is needed.

## tests

All tests were run with `go test ./...` from the root of the Hugo directory.

The `gofmt` tool was used on all modified files.
